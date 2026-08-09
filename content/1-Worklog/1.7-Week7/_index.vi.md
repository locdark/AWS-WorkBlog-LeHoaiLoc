---
title: "Worklog Tuần 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---


### Mục tiêu tuần 7:
* Viết báo cáo tham gia sự kiện AWS FCAJ Agent Forge - Deepdive.
* Cấu hình tên miền tùy chỉnh `myapps.io.vn` trỏ về Elastic IP của EC2, thiết lập Nginx Reverse Proxy và cài đặt chứng chỉ bảo mật SSL/HTTPS (Let's Encrypt / Certbot).
* Thực hiện kiểm thử liên thông (End-to-End Testing) toàn bộ hệ thống FoodieRecipe trên tên miền HTTPS thực tế.
* Giám sát hệ thống và xem logs của container qua Amazon CloudWatch Logs để phát hiện lỗi và tối ưu hóa mã nguồn.
* Hoàn thành quay và dựng video demo giới thiệu giao diện và các chức năng của website.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết báo cáo tham gia sự kiện AWS FCAJ Agent Forge - Deepdive. <br>- Thiết lập bản ghi DNS (A record) trỏ tên miền `myapps.io.vn` về IP tĩnh của máy chủ EC2. <br>- Cấu hình Nginx làm Reverse Proxy để chuyển hướng request từ cổng 80/443 về Next.js và NestJS. | 03/08/2026 | 03/08/2026 | [Nginx Configuration Guide](https://nginx.org/en/docs/) <br><br> [Báo cáo sự kiện 2](../../4-eventparticipated/4.2-event2/) |
| 3 | - Cài đặt Certbot và yêu cầu chứng chỉ SSL miễn phí từ Let's Encrypt. <br>- Cấu hình tự động gia hạn SSL và bật HTTPS cho toàn bộ hệ thống, cấu hình cookie secure cho phiên đăng nhập. | 03/08/2026 | 03/08/2026 | [Certbot Let's Encrypt Guide](https://certbot.eff.org/) |
| 4 | - Thực hiện kiểm thử toàn diện hệ thống (End-to-End Testing): Đăng ký/đăng nhập, upload ảnh món ăn lên S3, Rekognition nhận dạng nguyên liệu, Bedrock Nova Lite tạo công thức, lưu logs. | 03/08/2026 | 03/08/2026 | [AWS End-to-End Testing Guides](https://aws.amazon.com/) |
| 5 | - Đánh giá hiệu năng tải trang trên tên miền production, kiểm tra và debug các lỗi phát sinh từ logs container trên CloudWatch. Refactor tối ưu mã nguồn. | 03/08/2026 | 03/08/2026 | [Amazon CloudWatch Logs Guide](https://docs.aws.amazon.com/AmazonWatch/latest/logs/) |
| 6 | - Quay video demo chi tiết các tính năng chính của hệ thống FoodieRecipe trên môi trường production phục vụ cho phần báo cáo. | 03/08/2026 | 03/08/2026 | [AWS Presentation Best Practices](https://aws.amazon.com/) |

### Kết quả đạt được tuần 7:
* Hoàn thành báo cáo tham gia sự kiện AWS FCAJ Agent Forge - Deepdive.
* Cấu hình thành công tên miền tùy chỉnh `myapps.io.vn` với kết nối bảo mật HTTPS (Let's Encrypt SSL).
* Xác nhận toàn bộ hệ thống hoạt động ổn định và tối ưu trên hạ tầng AWS EC2 thông qua tên miền chính thức.
* Tập trung hóa việc giám sát và theo dõi lỗi thông qua việc kiểm tra log trên Amazon CloudWatch Logs.
* Hoàn thành quay và biên tập video demo giới thiệu chi tiết giao diện và chức năng của website trên môi trường production.
