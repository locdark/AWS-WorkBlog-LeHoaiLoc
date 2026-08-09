---
title: "Workshop"
date: 2026-06-22
weight: 5
chapter: true
pre: " <b> 5. </b> "
---

# Xây dựng và triển khai FoodieRecipe trên AWS

#### Tổng quan

Workshop hướng dẫn xây dựng sản phẩm **FoodieRecipe** bằng Next.js trong thư mục `web` và NestJS trong thư mục `api`. Người dùng có thể đăng ký, đăng nhập, khám phá, tạo, thích và bình luận công thức; tính năng **Tủ lạnh AI** nhận ảnh nguyên liệu và tự động tạo công thức.

Luồng AI thực tế nhận file multipart tại NestJS, tối ưu ảnh và upload vào Amazon S3, gọi Amazon Rekognition `DetectLabels`, chuẩn hóa nhãn nguyên liệu rồi gửi prompt dạng text đến Amazon Bedrock. Công thức và lịch sử được lưu trong PostgreSQL trên Amazon RDS. Ảnh private được trả về bằng CloudFront signed URL, hoặc S3 pre-signed GET URL khi chạy local.

Kiến trúc production chạy Next.js và NestJS bằng Docker trên EC2 sau Nginx, sử dụng domain `myapps.io.vn`. AWS Secrets Manager quản lý bí mật, IAM Role cấp temporary credential và Amazon CloudWatch thu thập log.

> **Lưu ý:** Workshop bám theo implementation hiện có: Rekognition chỉ dùng `DetectLabels`; chưa triển khai `DetectModerationLabels` hoặc upload trực tiếp bằng pre-signed PUT.

#### Nội dung

1. [Tổng quan Workshop](5.1-workshop-overview/)
2. [Điều kiện tiên quyết](5.2-prerequisites/)
3. [Hạ tầng cốt lõi](5.3-core-infrastructure/)
4. [Workflow ảnh AI](5.4-ai-image-workflow/)
5. [Kiểm thử và giám sát](5.5-validation-monitoring/)
6. [Dọn dẹp tài nguyên](5.6-cleanup/)

#### Kết quả sau Workshop

- Chuẩn bị được S3 private bucket, IAM Role, Secrets Manager, VPC, EC2 và RDS.
- Deploy được Next.js và NestJS bằng Docker/Nginx trên EC2.
- Gửi ảnh từ Next.js tới NestJS bằng multipart form data và lưu ảnh tối ưu trên S3.
- Nhận diện nguyên liệu với Rekognition và tạo công thức với Bedrock.
- Phân phối ảnh private qua CloudFront signed URL.
- Kiểm thử đăng nhập, công thức, lượt thích, bình luận, AI workflow và CloudWatch Logs.
