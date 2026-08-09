---
title: "Dọn dẹp tài nguyên"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Tổng kết

Sau Workshop, FoodieRecipe đã chạy Next.js và NestJS bằng Docker trên EC2/Nginx, dùng RDS PostgreSQL, S3 private bucket, Rekognition, Bedrock, CloudFront signed URL, Secrets Manager, IAM và CloudWatch. Nếu đây là môi trường thực hành, dọn tài nguyên để tránh tiếp tục phát sinh chi phí.

> **Nguy hiểm:** Các bước dưới đây xóa dữ liệu. Chỉ thực hiện sau khi đã xác nhận đúng AWS account, Region và tên tài nguyên; tạo snapshot/backup nếu cần giữ dữ liệu.

#### 1. Ngừng request mới

1. Dừng demo và các job gọi Bedrock/Rekognition.
2. Dừng container `foodierecipe-web` và `foodierecipe-api`.
3. Gỡ hoặc thay bản ghi DNS của `myapps.io.vn` nếu môi trường không còn hoạt động.

#### 2. Xóa CloudFront

1. Lấy cấu hình và ETag của distribution.
2. Đặt `Enabled=false`, cập nhật và chờ trạng thái `Deployed`.
3. Xóa distribution bằng ETag mới.
4. Xóa trusted key group và public key sau khi không còn behavior tham chiếu.
5. Xóa OAC sau khi distribution đã bị xóa.

```bash
aws cloudfront get-distribution-config --id <distribution-id>
aws cloudfront delete-distribution \
  --id <distribution-id> \
  --if-match <etag-after-disable>
```

#### 3. Làm rỗng và xóa S3 bucket

```bash
aws s3 rm s3://my-foodie-ai-images --recursive
aws s3api delete-bucket --bucket my-foodie-ai-images
```

Nếu bucket bật versioning, cần xóa toàn bộ version và delete marker trước khi xóa bucket.

#### 4. Xóa Backend và database

1. Dừng/terminate EC2.
2. Release Elastic IP nếu không còn sử dụng.
3. Xóa RDS; chỉ tạo final snapshot khi cần giữ dữ liệu.
4. Xóa DB subnet group và Security Group sau khi không còn dependency.
5. Chỉ xóa VPC/subnet/route table nếu chúng được tạo riêng cho Workshop.

#### 5. Xóa secret, IAM và log

1. Xóa `prod/foodie-recipe/db` và `prod/foodie-recipe/app` theo thời gian phục hồi phù hợp.
2. Xóa Parameter Store path `/foodierecipe/prod/` nếu có.
3. Detach policy rồi xóa IAM Role sau khi EC2 đã bị xóa.
4. Xóa `foodie-recipe-log` và các alarm/SNS topic không còn dùng.
5. Không xóa `RDSOSMetrics` khi tài nguyên khác vẫn cần log group này.

#### 6. Xác nhận chi phí

Mở **Billing and Cost Management** và kiểm tra EC2, RDS, S3, CloudFront, Secrets Manager, CloudWatch, Rekognition và Bedrock. Budget alert có thể tiếp tục gửi cảnh báo sau khi dọn dẹp vì dữ liệu chi phí cập nhật có độ trễ.

#### Kết quả

- Không còn EC2/RDS, Elastic IP, image object, CloudFront distribution hoặc secret thử nghiệm.
- DNS không còn trỏ đến máy chủ đã xóa.
- Không còn workload gọi Rekognition/Bedrock hoặc ghi CloudWatch Logs.
- Snapshot và tài nguyên dùng chung được giữ lại có chủ đích.
