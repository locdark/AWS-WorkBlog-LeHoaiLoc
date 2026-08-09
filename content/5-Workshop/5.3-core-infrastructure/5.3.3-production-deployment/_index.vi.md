---
title: "Deploy FoodieRecipe lên production"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

Phần này ghép các thành phần đã chuẩn bị thành một quy trình deploy hoàn chỉnh: build hai Docker image, lấy cấu hình từ AWS, chạy Next.js/NestJS trên EC2, cấu hình Nginx, DNS, HTTPS và kiểm tra production.

#### 1. Kiến trúc deploy

```text
Browser
   │ HTTPS — myapps.io.vn
   ▼
Nginx trên EC2
   ├── /      → Next.js container :3000
   └── /api/  → NestJS container  :3001
                         ├── RDS PostgreSQL
                         ├── S3 image bucket
                         ├── Rekognition
                         ├── Bedrock
                         └── CloudFront signed URL
```

Port `3000` và `3001` chỉ bind `127.0.0.1`; Nginx là entry point duy nhất từ Internet. RDS nằm private và chỉ nhận kết nối từ Security Group của EC2.

#### 2. Chuẩn bị máy chủ

Xác nhận EC2 đang chạy, đã gắn Elastic IP và IAM Role:

![EC2 FoodieRecipe sẵn sàng deploy](/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

```bash
aws sts get-caller-identity
docker --version
nginx -v
git --version

sudo systemctl enable --now docker
sudo systemctl enable --now nginx
```

Clone source và tạo release tag dùng chung cho hai image:

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe
git pull --ff-only

RELEASE_TAG="$(date +%Y%m%d-%H%M%S)"
echo "$RELEASE_TAG"
```

Không build trực tiếp từ branch chưa kiểm tra. Trong CI/CD, thay release timestamp bằng commit SHA để truy vết chính xác source của image.

#### 3. Chuẩn bị cấu hình AWS

Lưu dữ liệu theo hai nhóm:

| Nơi lưu | Giá trị |
| ------- | ------- |
| Secrets Manager `prod/foodie-recipe/db` | `DATABASE_URL` |
| Secrets Manager `prod/foodie-recipe/app` | `JWT_SECRET`, `CLOUDFRONT_PRIVATE_KEY_BASE64` |
| Parameter Store `/foodierecipe/prod/*` | Bucket, model ID, CloudFront domain/key pair/TTL |

IAM Role của EC2 cần `secretsmanager:GetSecretValue`, `ssm:GetParameter` và `kms:Decrypt` nếu dùng customer managed KMS key. Không tạo AWS access key trên máy chủ.

Kiểm tra trước khi build:

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

#### 4. Build NestJS image

Dockerfile `api` tạo Prisma Client và build NestJS trong builder stage. Runtime image tự chạy migration trước khi khởi động API:

```dockerfile
CMD ["sh", "-c", "./node_modules/.bin/prisma migrate deploy && exec node dist/main.js"]
```

Build image:

```bash
docker build \
  --tag "foodierecipe-api:${RELEASE_TAG}" \
  ./api
```

Không cần database thật trong build stage vì Prisma chỉ generate client; kết nối RDS chỉ diễn ra lúc container runtime và migration.

#### 5. Build Next.js image

Next.js dùng `output: "standalone"`. Các biến `NEXT_PUBLIC_*` được đóng gói tại build time, vì vậy Dockerfile builder cần nhận đầy đủ API URL và CloudFront domain:

```dockerfile
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}

ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}

RUN pnpm build
```

Lấy CloudFront domain từ Parameter Store và build:

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

#### 6. Chạy NestJS bằng cấu hình lấy từ AWS

Tạo script `deploy-api.sh`. Script chỉ giữ secret trong process hiện tại, không tạo `.env.production` trên ổ đĩa:

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

Log phải cho thấy `prisma migrate deploy` hoàn tất trước khi NestJS lắng nghe cổng `3001`.
`FRONTEND_URL` đồng thời giới hạn CORS về đúng website production; `COOKIE_SECURE=true` yêu cầu trình duyệt chỉ gửi cookie qua HTTPS.

#### 7. Chạy Next.js container

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

Chỉ chuyển sang bước Nginx khi cả hai container ở trạng thái `Up` và hai health request thành công.

#### 8. Cấu hình Nginx reverse proxy

Tạo `/etc/nginx/conf.d/foodierecipe.conf`:

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

`client_max_body_size` phải lớn hơn giới hạn ảnh tại NestJS; `proxy_read_timeout` cần đủ cho Bedrock nhưng không nên đặt vô hạn.

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I http://<EC2_ELASTIC_IP>
```

#### 9. Cấu hình DNS và HTTPS

Tạo bản ghi `A` của `myapps.io.vn` trỏ đến Elastic IP. Nếu dùng Route 53:

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

Nếu DNS do nhà cung cấp khác quản lý, tạo bản ghi `A` tương đương. Chỉ chạy Certbot sau khi domain đã phân giải đúng về EC2.

#### 10. Kiểm tra CloudFront và toàn bộ hệ thống

CloudFront chi tiết được cấu hình ở phần 5.4.4. Sau deploy, kiểm tra distribution và quyền ảnh:

```bash
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query 'Distribution.{Status:Status,Domain:DomainName}'

curl -I '<SIGNED_CLOUDFRONT_IMAGE_URL>'
curl -I 'https://my-foodie-ai-images.s3.<region>.amazonaws.com/ai-images/<key>.jpg'
```

Signed URL phải trả `200`; URL S3 trực tiếp phải trả `403`.

![FoodieRecipe hoạt động trên domain production](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

Smoke test sau deploy:

1. Đăng ký/đăng nhập và logout.
2. Tìm kiếm, tạo và xem công thức.
3. Thích/bỏ thích và tạo/xóa bình luận.
4. Upload một ảnh tại Tủ lạnh AI.
5. Xác nhận Rekognition labels, Bedrock recipe và record trong RDS.
6. Kiểm tra ảnh được đọc qua signed URL.

#### 11. Restart, rollback và giữ image

Kiểm tra tự phục hồi:

```bash
sudo systemctl restart docker
docker ps
curl -f https://myapps.io.vn/api
```

Giữ ít nhất một image tag ổn định trước đó. Khi release mới lỗi:

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

Migration phải tương thích ngược với release trước; thay đổi phá vỡ schema cần quy trình backup và rollback database riêng.

#### 12. Tiêu chí deploy thành công

- `https://myapps.io.vn` có HTTPS hợp lệ và trả giao diện Next.js.
- `/api` đi qua Nginx tới NestJS; port `3000/3001` không public.
- Migration hoàn tất và API kết nối được RDS private.
- EC2 Role gọi được S3, Rekognition, Bedrock, Secrets Manager và Parameter Store.
- Signed CloudFront URL hoạt động, S3 direct URL bị chặn.
- Hai container tự khởi động lại và có thể rollback bằng release tag.
