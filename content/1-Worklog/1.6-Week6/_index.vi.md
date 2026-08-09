---
title: "Worklog Tuần 6"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:
* Thiết kế hạ tầng mạng bảo mật với Amazon VPC (Public/Private Subnets, Internet Gateway, Security Groups).
* Triển khai Amazon RDS Instance làm cơ sở dữ liệu chính của dự án.
* Đóng gói các dịch vụ ứng dụng (Next.js Frontend & NestJS Backend) bằng Docker và triển khai lên Amazon EC2 thông qua Docker Compose.
* Tích hợp Amazon CloudFront CDN đứng trước S3 để tối ưu hóa tốc độ tải hình ảnh tĩnh.
* Cấu hình Amazon CloudWatch Logs để thu thập và theo dõi log từ các container chạy trên EC2.
* Viết bài Blog 2: *Kiểm soát và thương mại hóa lưu lượng AI Bot với AWS WAF* và Blog 3: *Tối ưu hóa Cold Start trong AWS Lambda (Provisioned Concurrency & SnapStart)*.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế và khởi tạo hệ thống mạng Amazon VPC. <br>- Phân bổ Public/Private Subnets, cấu hình Internet Gateway và thiết lập Security Groups để mở các cổng cần thiết (80, 443, 3000, 5432). <br>- Khởi tạo Database Instance trên Amazon RDS và kiểm tra kết nối từ local/VPC. | 27/07/2026 | 27/07/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/) <br><br> [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/) |
| 3 | - Viết Dockerfile tối ưu cho cả Next.js và NestJS. <br>- Khởi tạo Amazon EC2 Instance chạy hệ điều hành Ubuntu trong Public Subnet của VPC, cài đặt Docker Engine và Docker Compose. | 28/07/2026 | 28/07/2026 | [Docker Documentation](https://docs.docker.com/) <br><br> [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/) |
| 4 | - Viết tệp `docker-compose.yml` cho dự án, cấu hình Nginx làm Reverse Proxy. <br>- Thực hiện deploy dự án lên EC2 và trỏ kết nối database về RDS. | 29/07/2026 | 29/07/2026 | [Docker Compose Docs](https://docs.docker.com/compose/) <br><br> [Nginx Reverse Proxy Guide](https://nginx.org/en/docs/) |
| 5 | - Cấu hình Amazon CloudFront liên kết với Amazon S3 để tăng tốc độ phân phối hình ảnh món ăn. <br>- Cài đặt và cấu hình CloudWatch Logs agent trên EC2 để tự động đẩy logs của các docker container lên CloudWatch Logs nhóm. <br>- **Hoàn thành và đăng bài Blog 2: Cập nhật kỹ thuật AWS: Kiểm soát và thương mại hóa lưu lượng truy cập từ AI Bot với AWS WAF.** | 30/07/2026 | 30/07/2026 | [Amazon CloudFront Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/) <br><br> [AWS WAF Bot Control News](https://aws.amazon.com/blogs/aws/) |
| 6 | - Thực hiện kiểm tra hiệu suất (performance testing), tối ưu hóa cấu hình Nginx và giám sát tài nguyên qua CloudWatch Metrics. <br>- **Hoàn thành và đăng bài Blog 3: AWS LAMBDA – COLD START VÀ CÁCH TỐI ƯU (PROVISIONED CONCURRENCY, SNAPSTART).** | 31/07/2026 | 31/07/2026 | [Amazon CloudWatch Logs Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/) <br><br> [AWS Lambda SnapStart Docs](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html) |

### Kết quả đạt được tuần 6:
* Thiết kế và xây dựng thành công kiến trúc mạng VPC bảo mật, đưa RDS database vào Private Subnet để nâng cao tính an toàn.
* Đóng gói Docker hoàn chỉnh và triển khai ứng dụng Next.js & NestJS thành công trên Amazon EC2 bằng Docker Compose.
* Tích hợp thành công Amazon CloudFront giúp tối ưu hóa tốc độ tải tài nguyên tĩnh (hình ảnh) của ứng dụng.
* Tập trung hóa việc giám sát và theo dõi lỗi thông qua việc đẩy log container lên Amazon CloudWatch Logs.
* Hoàn thành và đăng tải hai bài viết blog kỹ thuật chất lượng cao (Blog 2 về AWS WAF Bot Control và Blog 3 về AWS Lambda Cold Start).
* Đồng bộ mã nguồn triển khai hạ tầng lên GitHub của nhóm.
