---
title: "Chuẩn bị NestJS, EC2 và Amazon RDS"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### 1. Kiến trúc mạng VPC

Mở **Amazon VPC → Your VPCs → Resource map** để kiểm tra VPC, subnet, route table và Internet Gateway.

![Resource map của VPC triển khai FoodieRecipe](/AWS-WorkBlog-LeHoaiLoc/images/5-Workshop/5.3-Core-infrastructure/5.3.2-backend-data/vpc-resource-map.png)

- EC2/Nginx nằm trong public subnet và dùng Elastic IP.
- RDS dùng DB subnet group trải trên ít nhất hai Availability Zone.
- RDS đặt **Public access: No**.
- `FoodieRecipeRdsSg` chỉ nhận PostgreSQL `5432` từ `FoodieRecipeEc2Sg`.
- Port `3000` và `3001` chỉ bind loopback; người dùng truy cập qua Nginx `80/443`.

> **Lưu ý:** Resource map chỉ thể hiện quan hệ tài nguyên. Hãy kiểm tra route table và Security Group để xác nhận ranh giới public/private.

#### 2. Tạo Amazon RDS for PostgreSQL

1. Tạo DB subnet group từ các subnet đã chọn.
2. Chọn PostgreSQL, DB identifier `database-foodie-recipe` và cấu hình phù hợp workshop.
3. Bật encryption, automated backup và **Public access: No**.
4. Gắn `FoodieRecipeRdsSg`.
5. Lưu connection string trong secret `prod/foodie-recipe/db`.

![RDS PostgreSQL FoodieRecipe ở trạng thái Available](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.2-Week2/rds-database-created.png)

Không mở `5432` cho `0.0.0.0/0` và không đưa password vào Git.

#### 3. Chạy migration Prisma

Schema hiện có các bảng chính: `users`, `recipes`, `recipe_ingredients`, `recipe_steps`, `recipe_images`, `recipe_likes`, `comments` và `ai_generation_history`. Trạng thái lịch sử AI gồm `PENDING`, `PROCESSING`, `SUCCESS`, `FAILED`.

```bash
cd api
pnpm prisma generate
pnpm prisma migrate deploy
pnpm prisma studio
```

![Các bảng sau khi Prisma migration](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/prisma-migration-result.png)

`RecipeImage` lưu URL, loại ảnh và thứ tự hiển thị; trạng thái AI thuộc `AIGenerationHistory`, không thuộc bảng ảnh.

#### 4. Chuẩn bị EC2

1. Tạo Linux EC2 trong public subnet và gắn Elastic IP.
2. Gắn IAM Role của Backend cùng `FoodieRecipeEc2Sg`.
3. Chỉ mở SSH `22` từ IP quản trị; mở `80/443` cho người dùng.

![EC2 FoodieRecipe đang chạy](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

![IAM Role và Security Group gắn với EC2](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.7-Week7/ec2-iam-role.png)

```bash
aws sts get-caller-identity
sudo dnf update -y
sudo dnf install -y git docker nginx jq
sudo systemctl enable --now docker nginx
sudo usermod -aG docker ec2-user
```

Đăng xuất rồi SSH lại để quyền Docker có hiệu lực.

#### 5. Build và chạy NestJS

Dockerfile của `api` dùng Node.js 22, pnpm, Prisma Client và NestJS build. Entrypoint chạy `prisma migrate deploy` trước `node dist/main.js`.

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe/api
docker build -t foodierecipe-api:production .
```

Không tạo `.env.production`. Script deploy lấy secret bằng temporary credential của EC2 Role và truyền vào container:

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

![PostgreSQL container hoạt động trong môi trường local](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/docker-postgres-running.png)

#### 6. Cấu hình Nginx

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

![Health endpoint trả về Hello World](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/api-hello-world.png)

#### 7. Kiểm tra API và database

Swagger giúp kiểm tra contract của API:

![Swagger FoodieRecipe API](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/swagger-overview.png)

![Đăng nhập thành công qua Swagger](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.3-Week3/swagger-login-success.png)

Xác nhận migration hoàn tất, đăng ký/đăng nhập hoạt động, API tạo công thức ghi được RDS và container tự khởi động lại sau reboot EC2.
