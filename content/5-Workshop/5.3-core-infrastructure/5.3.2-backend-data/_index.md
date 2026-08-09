---
title: "Prepare NestJS, EC2, and Amazon RDS"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### 1. VPC network architecture

Open **Amazon VPC → Your VPCs → Resource map** to inspect the VPC, subnets, route table, and Internet Gateway.

![Resource map of the VPC used for FoodieRecipe](/AWS-WorkBlog-LeHoaiLoc/images/5-Workshop/5.3-Core-infrastructure/5.3.2-backend-data/vpc-resource-map.png)

- EC2/Nginx resides in a public subnet and uses an Elastic IP.
- RDS uses a DB subnet group spanning at least two Availability Zones.
- RDS has **Public access: No**.
- `FoodieRecipeRdsSg` accepts PostgreSQL `5432` only from `FoodieRecipeEc2Sg`.
- Ports `3000` and `3001` bind only to loopback; users enter through Nginx on `80/443`.

> **Note:** The resource map shows relationships only. Inspect route tables and Security Groups to confirm public/private boundaries.

#### 2. Create Amazon RDS for PostgreSQL

1. Create a DB subnet group from the selected subnets.
2. Select PostgreSQL, identifier `database-foodie-recipe`, and a workshop-appropriate size.
3. Enable encryption, automated backups, and **Public access: No**.
4. Attach `FoodieRecipeRdsSg`.
5. Store the connection string in `prod/foodie-recipe/db`.

![FoodieRecipe RDS PostgreSQL in the Available state](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.2-Week2/rds-database-created.png)

Never expose `5432` to `0.0.0.0/0` or commit the database password.

#### 3. Run Prisma migrations

The current schema includes `users`, `recipes`, `recipe_ingredients`, `recipe_steps`, `recipe_images`, `recipe_likes`, `comments`, and `ai_generation_history`. AI history supports `PENDING`, `PROCESSING`, `SUCCESS`, and `FAILED`.

```bash
cd api
pnpm prisma generate
pnpm prisma migrate deploy
pnpm prisma studio
```

![Tables after Prisma migration](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/prisma-migration-result.png)

`RecipeImage` stores the URL, image type, and display order. AI status belongs to `AIGenerationHistory`, not the image table.

#### 4. Prepare EC2

1. Create a Linux EC2 instance in the public subnet and attach an Elastic IP.
2. Attach the Backend IAM role and `FoodieRecipeEc2Sg`.
3. Allow SSH `22` only from the administrator IP and expose `80/443` to users.

![FoodieRecipe EC2 instance running](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

![IAM role and Security Groups attached to EC2](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.7-Week7/ec2-iam-role.png)

```bash
aws sts get-caller-identity
sudo dnf update -y
sudo dnf install -y git docker nginx jq
sudo systemctl enable --now docker nginx
sudo usermod -aG docker ec2-user
```

Sign out and reconnect for Docker group membership to take effect.

#### 5. Build and run NestJS

The `api` Dockerfile uses Node.js 22, pnpm, Prisma Client, and a NestJS build. Its entrypoint runs `prisma migrate deploy` before `node dist/main.js`.

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe/api
docker build -t foodierecipe-api:production .
```

Do not create `.env.production`. The deployment script retrieves secrets with temporary EC2 role credentials and injects them into the container:

```bash
REGION=ap-southeast-1

DB_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id prod/foodie-recipe/db \
  --query SecretString --output text)

APP_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id prod/foodie-recipe/app \
  --query SecretString --output text)

DATABASE_URL=$(jq -r '.DATABASE_URL' <<< "$DB_SECRET")
JWT_SECRET=$(jq -r '.JWT_SECRET' <<< "$APP_SECRET")

docker run -d \
  --name foodierecipe-api \
  --restart unless-stopped \
  -p 127.0.0.1:3001:3001 \
  -e "DATABASE_URL=$DATABASE_URL" \
  -e PORT=3001 \
  -e NODE_ENV=production \
  -e COOKIE_SECURE=true \
  -e "JWT_SECRET=$JWT_SECRET" \
  -e AWS_REGION="$REGION" \
  -e AWS_BUCKET_NAME=my-foodie-ai-images \
  -e BEDROCK_MODEL_ID='<model-id>' \
  foodierecipe-api:production

unset DB_SECRET APP_SECRET DATABASE_URL JWT_SECRET
```

![PostgreSQL container running in the local environment](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/docker-postgres-running.png)

#### 6. Configure Nginx

```nginx
server {
    listen 80;
    server_name myapps.io.vn;

    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo nginx -t
sudo systemctl reload nginx
curl http://127.0.0.1:3001/api
curl -I https://myapps.io.vn
```

![Health endpoint returning Hello World](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/api-hello-world.png)

#### 7. Verify the API and database

Swagger verifies the API contract:

![FoodieRecipe Swagger API](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/swagger-overview.png)

![Successful sign-in through Swagger](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/swagger-login-success.png)

Confirm that migrations complete, registration/sign-in works, recipe creation writes to RDS, and the container restarts after an EC2 reboot.
