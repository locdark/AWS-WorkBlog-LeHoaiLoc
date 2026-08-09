---
title: "Deploy FoodieRecipe to production"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

This section combines the prepared components into a complete deployment: build both Docker images, retrieve AWS configuration, run Next.js/NestJS on EC2, configure Nginx, DNS, HTTPS, and verify production.

#### 1. Deployment architecture

```text
Browser
   │ HTTPS — myapps.io.vn
   ▼
Nginx on EC2
   ├── /      → Next.js container :3000
   └── /api/  → NestJS container  :3001
                         ├── RDS PostgreSQL
                         ├── S3 image bucket
                         ├── Rekognition
                         ├── Bedrock
                         └── CloudFront signed URL
```

Ports `3000` and `3001` bind only to `127.0.0.1`; Nginx is the only Internet-facing entry point. RDS remains private and accepts only the EC2 Security Group.

#### 2. Prepare the server

Confirm that EC2 is running with an Elastic IP and IAM role:

![FoodieRecipe EC2 ready for deployment](/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

```bash
aws sts get-caller-identity
docker --version
nginx -v
git --version

sudo systemctl enable --now docker
sudo systemctl enable --now nginx
```

Clone source and create one release tag for both images:

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe
git pull --ff-only

RELEASE_TAG="$(date +%Y%m%d-%H%M%S)"
echo "$RELEASE_TAG"
```

Do not build from an unverified branch. In CI/CD, use the commit SHA instead of a timestamp so every image is traceable to source.

#### 3. Prepare AWS configuration

Separate values by sensitivity:

| Store | Values |
| ----- | ------ |
| Secrets Manager `prod/foodie-recipe/db` | `DATABASE_URL` |
| Secrets Manager `prod/foodie-recipe/app` | `JWT_SECRET`, `CLOUDFRONT_PRIVATE_KEY_BASE64` |
| Parameter Store `/foodierecipe/prod/*` | Bucket, model ID, CloudFront domain/key pair/TTL |

The EC2 role needs `secretsmanager:GetSecretValue`, `ssm:GetParameter`, and `kms:Decrypt` when customer-managed KMS keys are used. Never create AWS access keys on the server.

Pre-build checks:

```bash
REGION=ap-southeast-1

aws secretsmanager describe-secret \
  --region "$REGION" \
  --secret-id prod/foodie-recipe/db

aws ssm get-parameters-by-path \
  --region "$REGION" \
  --path /foodierecipe/prod/ \
  --recursive \
  --query 'Parameters[*].Name'
```

#### 4. Build the NestJS image

The `api` Dockerfile generates Prisma Client and builds NestJS in the builder stage. The runtime image deploys migrations before starting the API:

```dockerfile
CMD ["sh", "-c", "./node_modules/.bin/prisma migrate deploy && exec node dist/main.js"]
```

Build the image:

```bash
docker build \
  --tag "foodierecipe-api:${RELEASE_TAG}" \
  ./api
```

A live database is unnecessary during the build because Prisma only generates the client. RDS connectivity is required at container startup for migrations.

#### 5. Build the Next.js image

Next.js uses `output: "standalone"`. `NEXT_PUBLIC_*` values are embedded at build time, so the Dockerfile builder must accept the API URL and CloudFront domain:

```dockerfile
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}

ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}

RUN pnpm build
```

Retrieve the CloudFront domain from Parameter Store and build:

```bash
CLOUDFRONT_DOMAIN=$(aws ssm get-parameter \
  --region "$REGION" \
  --name /foodierecipe/prod/CLOUDFRONT_DOMAIN \
  --query Parameter.Value \
  --output text)

docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://myapps.io.vn/api \
  --build-arg NEXT_PUBLIC_CLOUDFRONT_DOMAIN="$CLOUDFRONT_DOMAIN" \
  --tag "foodierecipe-web:${RELEASE_TAG}" \
  ./web
```

#### 6. Run NestJS with AWS-managed configuration

Create `deploy-api.sh`. It keeps secrets in the current process and does not create `.env.production` on disk:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

REGION="ap-southeast-1"
PARAMETER_PATH="/foodierecipe/prod"
RELEASE_TAG="${1:?Usage: deploy-api.sh <release-tag>}"

get_parameter() {
  aws ssm get-parameter \
    --region "$REGION" \
    --name "$PARAMETER_PATH/$1" \
    --query Parameter.Value \
    --output text
}

DB_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id prod/foodie-recipe/db \
  --query SecretString --output text)

APP_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id prod/foodie-recipe/app \
  --query SecretString --output text)

DATABASE_URL=$(jq -er '.DATABASE_URL' <<< "$DB_SECRET")
JWT_SECRET=$(jq -er '.JWT_SECRET' <<< "$APP_SECRET")
CLOUDFRONT_PRIVATE_KEY_BASE64=$(
  jq -er '.CLOUDFRONT_PRIVATE_KEY_BASE64' <<< "$APP_SECRET"
)

docker rm -f foodierecipe-api 2>/dev/null || true
docker run -d \
  --name foodierecipe-api \
  --restart unless-stopped \
  -p 127.0.0.1:3001:3001 \
  -e "DATABASE_URL=$DATABASE_URL" \
  -e PORT=3001 \
  -e NODE_ENV=production \
  -e COOKIE_SECURE=true \
  -e FRONTEND_URL=https://myapps.io.vn \
  -e "JWT_SECRET=$JWT_SECRET" \
  -e JWT_EXPIRES_IN=7d \
  -e AWS_REGION="$REGION" \
  -e "AWS_BUCKET_NAME=$(get_parameter AWS_BUCKET_NAME)" \
  -e "BEDROCK_MODEL_ID=$(get_parameter BEDROCK_MODEL_ID)" \
  -e "CLOUDFRONT_DOMAIN=$(get_parameter CLOUDFRONT_DOMAIN)" \
  -e "CLOUDFRONT_KEY_PAIR_ID=$(get_parameter CLOUDFRONT_KEY_PAIR_ID)" \
  -e "CLOUDFRONT_PRIVATE_KEY_BASE64=$CLOUDFRONT_PRIVATE_KEY_BASE64" \
  -e "CLOUDFRONT_URL_EXPIRES_IN=$(get_parameter CLOUDFRONT_URL_EXPIRES_IN)" \
  "foodierecipe-api:${RELEASE_TAG}"

unset DB_SECRET APP_SECRET DATABASE_URL JWT_SECRET
unset CLOUDFRONT_PRIVATE_KEY_BASE64
```

```bash
chmod 700 deploy-api.sh
./deploy-api.sh "$RELEASE_TAG"
docker logs --tail 100 foodierecipe-api
curl http://127.0.0.1:3001/api
```

Logs must show `prisma migrate deploy` completing before NestJS listens on port `3001`.
`FRONTEND_URL` also restricts CORS to the production website, while `COOKIE_SECURE=true` requires browsers to send cookies only over HTTPS.

#### 7. Run the Next.js container

```bash
docker rm -f foodierecipe-web 2>/dev/null || true
docker run -d \
  --name foodierecipe-web \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  "foodierecipe-web:${RELEASE_TAG}"

docker logs --tail 100 foodierecipe-web
curl -I http://127.0.0.1:3000
docker ps
```

Continue to Nginx only after both containers are `Up` and their local health requests succeed.

#### 8. Configure the Nginx reverse proxy

Create `/etc/nginx/conf.d/foodierecipe.conf`:

```nginx
server {
    listen 80;
    server_name myapps.io.vn;

    client_max_body_size 10m;

    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

`client_max_body_size` must exceed the NestJS image limit. `proxy_read_timeout` must accommodate Bedrock without being unbounded.

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I http://<EC2_ELASTIC_IP>
```

#### 9. Configure DNS and HTTPS

Create an `A` record for `myapps.io.vn` pointing to the Elastic IP. With Route 53:

```json
{
  "Comment": "FoodieRecipe production domain",
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "myapps.io.vn.",
      "Type": "A",
      "TTL": 300,
      "ResourceRecords": [{ "Value": "<EC2_ELASTIC_IP>" }]
    }
  }]
}
```

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id <HOSTED_ZONE_ID> \
  --change-batch file://dns-change.json

dig +short myapps.io.vn
sudo certbot --nginx -d myapps.io.vn
sudo certbot renew --dry-run
curl -I https://myapps.io.vn
```

If another provider manages DNS, create the equivalent `A` record. Run Certbot only after the domain resolves to EC2.

#### 10. Verify CloudFront and the complete system

CloudFront is configured in detail in section 5.4.4. After deployment, verify the distribution and image access:

```bash
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query 'Distribution.{Status:Status,Domain:DomainName}'

curl -I '<SIGNED_CLOUDFRONT_IMAGE_URL>'
curl -I 'https://my-foodie-ai-images.s3.<region>.amazonaws.com/ai-images/<key>.jpg'
```

The signed URL must return `200`; direct S3 access must return `403`.

![FoodieRecipe running on the production domain](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

Post-deployment smoke tests:

1. Register/sign in and sign out.
2. Search, create, and view recipes.
3. Like/unlike and create/delete comments.
4. Upload an image through AI Fridge.
5. Confirm Rekognition labels, the Bedrock recipe, and RDS records.
6. Confirm that the image loads through a signed URL.

#### 11. Restart, rollback, and image retention

Verify recovery:

```bash
sudo systemctl restart docker
docker ps
curl -f https://myapps.io.vn/api
```

Retain at least one previous stable tag. If the new release fails:

```bash
FAILED_TAG="$RELEASE_TAG"
PREVIOUS_TAG="<previous-stable-tag>"

docker rm -f foodierecipe-api foodierecipe-web
./deploy-api.sh "$PREVIOUS_TAG"

docker run -d \
  --name foodierecipe-web \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  "foodierecipe-web:${PREVIOUS_TAG}"

curl -f https://myapps.io.vn/api
```

Migrations must remain backward-compatible with the previous release. Breaking schema changes require a separate database backup and rollback procedure.

#### 12. Deployment completion criteria

- `https://myapps.io.vn` has valid HTTPS and serves Next.js.
- `/api` routes through Nginx to NestJS; ports `3000/3001` are not public.
- Migrations complete and the API reaches private RDS.
- The EC2 role can call S3, Rekognition, Bedrock, Secrets Manager, and Parameter Store.
- CloudFront signed URLs work and direct S3 access is denied.
- Both containers restart automatically and can roll back by release tag.
