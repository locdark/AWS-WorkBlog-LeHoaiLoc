---
title: "Chuẩn bị môi trường"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### 1. Tài khoản và Region

1. Đăng nhập AWS bằng IAM user/role dành cho workshop, không dùng root user.
2. Chọn một Region hỗ trợ Amazon Rekognition và model Bedrock dự kiến sử dụng.
3. Ghi lại **Account ID** và **Region** để đặt tên tài nguyên.
4. Tạo AWS Budget Alert trước khi bắt đầu.

> **Cảnh báo:** Amazon Bedrock model availability và quyền truy cập phụ thuộc Region/tài khoản. Hãy xác nhận model đã được cấp quyền trước khi thực hiện phần 5.4.3.

#### 2. Công cụ cục bộ

Cài đặt và kiểm tra:

```bash
node --version
npm --version
aws --version
docker --version
git --version
```

Khuyến nghị sử dụng Node.js LTS, AWS CLI v2 và Docker Desktop/Engine phiên bản ổn định.

#### 3. Cấu hình AWS CLI

```bash
aws configure
aws sts get-caller-identity
aws configure get region
```

Không commit file credential, `.env` hoặc pre-signed URL vào Git.

#### 4. Chuẩn bị mã nguồn

Cấu trúc tham khảo:

```text
foodierecipe/
├── web/            # Next.js web application
├── api/            # NestJS API
├── docs/
└── .gitignore
```

Giữ `api/.env.example` chỉ gồm tên key để hướng dẫn chạy local; không điền giá trị thật:

```dotenv
# Database
DATABASE_URL=

# App
PORT=
NODE_ENV=
# Set false for HTTP deployments; set true when the API is served over HTTPS.
COOKIE_SECURE=

# JWT / auth (optional placeholders)
JWT_SECRET=
JWT_EXPIRES_IN=

# S3 bucket
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=

# CloudFront private delivery (leave all values empty to use S3 presigned URLs locally)
CLOUDFRONT_DOMAIN=
CLOUDFRONT_KEY_PAIR_ID=
# Base64 of the PEM private key. Never commit the real key.
CLOUDFRONT_PRIVATE_KEY_BASE64=
CLOUDFRONT_URL_EXPIRES_IN=

# Email verification (Resend)
RESEND_API_KEY=
EMAIL_FROM=

BEDROCK_MODEL_ID=
```

> **Cảnh báo:** Các giá trị trong `.env.example` chỉ là placeholder. Không commit `.env` thật. Production không tạo `.env.production`: EC2 dùng IAM Role cho AWS credential, Secrets Manager cho giá trị nhạy cảm và Parameter Store cho cấu hình không nhạy cảm. Đặt `COOKIE_SECURE=true` khi API được phục vụ qua HTTPS.

- `DATABASE_URL` sử dụng PostgreSQL local ở cổng `5433`.
- `CLOUDFRONT_*` để trống khi chạy local; `api` sẽ trả S3 pre-signed GET URL.
- `CLOUDFRONT_PRIVATE_KEY_BASE64` là private key đã mã hóa Base64, không phải nội dung PEM thuần.
- `RESEND_API_KEY` và `EMAIL_FROM` chỉ cần khi bật xác minh email.

`web/.env.example`:

```dotenv
NEXT_PUBLIC_API_URL=
NEXT_PUBLIC_APP_NAME=
NEXT_PUBLIC_GOOGLE_CLIENT_ID=
NEXT_PUBLIC_CLOUDFRONT_DOMAIN=
```

Các biến `NEXT_PUBLIC_*` phải được cung cấp trước khi build Next.js vì giá trị được đóng gói vào client bundle.

#### 5. Quyền cần thiết

IAM principal thực hiện workshop cần quyền tạo/quản lý các tài nguyên được sử dụng: S3, CloudFront, EC2, IAM role, RDS, Secrets Manager, CloudWatch Logs, Rekognition và Bedrock.

> **Lưu ý:** Không dùng policy `AdministratorAccess` cho ứng dụng. Ở phần 5.3.1, bạn sẽ tạo role riêng cho EC2 theo nguyên tắc least privilege.

#### 6. Quy ước tài nguyên

| Tài nguyên | Tên gợi ý |
| ---------- | ---------- |
| Image bucket | `my-foodie-ai-images` hoặc tên duy nhất tương đương |
| EC2 role | `FoodieRecipeBackendRole` |
| EC2 Security Group | `FoodieRecipeEc2Sg` |
| RDS Security Group | `FoodieRecipeRdsSg` |
| Database secret | `prod/foodie-recipe/db` |
| Application secret | `prod/foodie-recipe/app` |
| Log group | `foodie-recipe-log` |
| Production domain | `myapps.io.vn` |

Sau khi hoàn tất, chuyển sang [Xây dựng hạ tầng cốt lõi](../5.3-core-infrastructure/).
