---
title: "Workflow ảnh AI"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Phần này triển khai đúng workflow **Next.js → NestJS → S3 → Rekognition → Bedrock → RDS → CloudFront** đang có trong FoodieRecipe.

#### Nội dung

1. [Upload và tối ưu ảnh qua NestJS](5.4.1-upload-flow/)
2. [Nhận diện nguyên liệu với Rekognition](5.4.2-rekognition/)
3. [Tạo và lưu công thức với Bedrock](5.4.3-bedrock/)
4. [Phân phối ảnh private qua CloudFront](5.4.4-cloudfront/)

#### Quy tắc workflow

- Frontend gửi file multipart; Backend chịu trách nhiệm validate, tối ưu và upload S3.
- Rekognition chỉ gọi `DetectLabels`; không ghi nhận moderation khi chưa triển khai API đó.
- Chỉ nhãn có confidence từ 80% và không nằm trong danh sách nhãn chung mới trở thành nguyên liệu.
- Bedrock nhận prompt text, output phải parse được thành JSON theo schema công thức.
- Lịch sử AI hỗ trợ `PENDING`, `PROCESSING`, `SUCCESS`, `FAILED`; không tạo thêm trạng thái ảnh không tồn tại trong schema.
- S3 bucket luôn private; CloudFront signed URL hoặc S3 pre-signed GET URL cung cấp quyền đọc có thời hạn.
