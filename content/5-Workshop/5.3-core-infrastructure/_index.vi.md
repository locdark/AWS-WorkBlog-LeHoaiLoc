---
title: "Xây dựng hạ tầng cốt lõi"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Phần này tạo và triển khai các thành phần nền tảng để NestJS lưu ảnh, gọi dịch vụ AI và ghi dữ liệu FoodieRecipe.

#### Nội dung

1. [Tạo S3, IAM và Secrets Manager](5.3.1-storage-security/)
2. [Chuẩn bị NestJS, EC2 và Amazon RDS](5.3.2-backend-data/)
3. [Deploy FoodieRecipe lên production](5.3.3-production-deployment/)

#### Kiến trúc sau phần này

- Một S3 image bucket private với prefix `ai-images/` đã sẵn sàng.
- EC2 có IAM role theo nguyên tắc least privilege.
- Database credential được lưu trong Secrets Manager.
- NestJS chạy trong Docker sau Nginx và kết nối được RDS.
- Next.js chạy bằng standalone container; domain `myapps.io.vn` được phục vụ qua HTTPS.
- Migration đã tạo database schema và health check xác nhận API sau deploy.

> **Lưu ý:** Hoàn thành S3, IAM, Secrets Manager, EC2, RDS, Docker và Nginx trước khi triển khai workflow AI. CloudWatch được cấu hình và kiểm tra ở phần 5.5.
