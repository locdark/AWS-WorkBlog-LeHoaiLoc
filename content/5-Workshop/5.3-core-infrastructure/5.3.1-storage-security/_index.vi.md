---
title: "Chuẩn bị S3, IAM và Secrets Manager"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### 1. Tạo S3 image bucket

1. Mở **Amazon S3 → Create bucket**.
2. Đặt tên duy nhất, ví dụ `my-foodie-ai-images`.
3. Chọn cùng Region với EC2, Rekognition và Bedrock khi model hỗ trợ.
4. Giữ **Object Ownership: Bucket owner enforced** và tắt ACL.
5. Bật toàn bộ **Block Public Access** và bật default encryption.

![S3 bucket my-foodie-ai-images đã được tạo](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.2-Week2/s3-bucket-created.png)

![Block Public Access được bật cho bucket](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.2-Week2/s3-block-public-access.png)

Backend lưu ảnh AI dưới prefix `ai-images/`; ảnh công thức và avatar có thể dùng folder riêng nhưng vẫn trong bucket private.

> **Lưu ý:** Workflow hiện tại upload qua NestJS, không upload trực tiếp từ trình duyệt tới S3. Vì vậy image bucket không cần CORS cho pre-signed PUT. Chỉ thêm CORS nếu sau này triển khai direct-upload flow.

#### 2. Lifecycle và quyền truy cập

Có thể tạo lifecycle rule cho ảnh thử nghiệm hoặc object không còn được tham chiếu. Không đặt rule xóa chung cho ảnh công thức đang được sử dụng.

IAM policy tối thiểu của Backend:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "FoodieRecipeImageObjects",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::my-foodie-ai-images/*"
    },
    {
      "Sid": "DetectIngredients",
      "Effect": "Allow",
      "Action": "rekognition:DetectLabels",
      "Resource": "*"
    },
    {
      "Sid": "GenerateRecipe",
      "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "arn:aws:bedrock:<region>::foundation-model/<model-id>"
    },
    {
      "Sid": "ReadApplicationSecrets",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": [
        "arn:aws:secretsmanager:<region>:<account-id>:secret:prod/foodie-recipe/db-*",
        "arn:aws:secretsmanager:<region>:<account-id>:secret:prod/foodie-recipe/app-*"
      ]
    }
  ]
}
```

Gắn policy vào IAM Role dành cho EC2. Không tạo access key cho Role và không dùng `AdministratorAccess` cho ứng dụng.

#### 3. Tạo bí mật

Tạo hai secret:

- `prod/foodie-recipe/db`: `DATABASE_URL` hoặc thông tin kết nối RDS.
- `prod/foodie-recipe/app`: `JWT_SECRET` và `CLOUDFRONT_PRIVATE_KEY_BASE64`.

![Secrets Manager chứa secret của FoodieRecipe](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.2-Week2/secrets-manager-created.png)

Các giá trị không nhạy cảm như bucket, model ID, CloudFront domain và TTL có thể lưu trong Systems Manager Parameter Store.

#### 4. Kiểm tra

```bash
aws sts get-caller-identity
aws s3api get-public-access-block --bucket my-foodie-ai-images
aws secretsmanager describe-secret --secret-id prod/foodie-recipe/db
```

Kết quả đạt yêu cầu khi bucket private, EC2 Role chỉ có quyền cần thiết và không có secret thật trong Git/Docker image.
