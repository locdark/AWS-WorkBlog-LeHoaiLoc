---
title: "Tổng quan Workshop"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu FoodieRecipe

FoodieRecipe là nền tảng chia sẻ công thức nấu ăn có chức năng tài khoản, tìm kiếm, quản lý công thức, thích, bình luận và tạo công thức bằng AI từ ảnh nguyên liệu.

![FoodieRecipe hoạt động trên domain production](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

#### Kiến trúc Workshop

![Kiến trúc FoodieRecipe trên AWS](/images/2-Proposal/foodie-recipe-architecture.png)

| Thành phần | Trách nhiệm |
| ---------- | ----------- |
| Next.js `web` | Giao diện, xác thực, tìm kiếm, thích, bình luận và gửi ảnh AI |
| NestJS `api` | API nghiệp vụ, upload ảnh, gọi dịch vụ AWS và lưu dữ liệu |
| EC2, Docker, Nginx | Chạy hai container và phục vụ domain production |
| Amazon RDS PostgreSQL | Lưu người dùng, công thức, nguyên liệu, tương tác và lịch sử AI |
| Amazon S3 | Lưu ảnh đã được Backend resize và chuyển thành JPEG |
| Amazon Rekognition | Phát hiện label trong ảnh bằng `DetectLabels` |
| Amazon Bedrock | Sinh JSON công thức từ prompt chứa nguyên liệu |
| Amazon CloudFront | Trả signed URL có thời hạn cho ảnh private |
| Secrets Manager, IAM | Quản lý bí mật và temporary credential |
| CloudWatch | Thu thập log ứng dụng và RDS metrics |

#### Luồng end-to-end thực tế

1. Người dùng đăng nhập và chọn ảnh tại trang **Tủ lạnh AI**.
2. Next.js gửi `multipart/form-data` đến `POST /api/ai/analyze-image`.
3. NestJS dùng `sharp` resize ảnh tối đa 1024 px, nén JPEG chất lượng 85 và upload vào `ai-images/` trên S3.
4. Rekognition đọc object S3, trả tối đa 30 label với ngưỡng dịch vụ 50%.
5. Backend chỉ giữ nhãn nguyên liệu từ 80%, loại nhãn chung, chuẩn hóa và loại trùng.
6. Bedrock nhận prompt dạng text và trả JSON công thức.
7. NestJS validate kết quả, lưu recipe, ingredients, steps và generation history trong RDS.
8. Khi đọc ảnh, Backend trả CloudFront signed URL; local fallback dùng S3 pre-signed GET URL.

#### Mục tiêu học tập

- Hiểu ranh giới trách nhiệm giữa Next.js, NestJS và các dịch vụ AWS.
- Dùng IAM Role thay cho static access key trên EC2.
- Xây dựng workflow S3–Rekognition–Bedrock đúng với source.
- Deploy ứng dụng container hóa sau Nginx và domain riêng.
- Kiểm thử quyền truy cập ảnh, log và các chức năng nghiệp vụ.
