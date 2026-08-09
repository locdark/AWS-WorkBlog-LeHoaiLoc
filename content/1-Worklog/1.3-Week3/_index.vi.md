---
title: "Worklog Tuần 3"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---


### Mục tiêu tuần 3:
* Tìm hiểu mô hình CI/CD trên AWS nhằm tự động hóa quá trình tích hợp và triển khai ứng dụng bằng AWS CodePipeline, GitHub và Amazon S3.
* Khảo sát và thực hành các dịch vụ Amazon CloudWatch, AWS Lambda và Amazon Cognito, đồng thời nghiên cứu cách kết hợp Lambda, Amazon S3 và Amazon DynamoDB để phát triển các ứng dụng theo kiến trúc serverless.
* Bắt đầu phát triển khung mã nguồn cho dự án FoodieRecipe, xây dựng layout trang Social bằng Next.js và cấu trúc thư mục module backend NestJS kết nối cơ sở dữ liệu RDS.
* Triển khai cơ chế xác thực và bảo mật (Authentication/Authorization) cho ứng dụng sử dụng JWT và NestJS Guard kết hợp giao diện đăng nhập Social Module.
* Cập nhật nhật ký công việc và đồng bộ toàn bộ mã nguồn của dự án lên GitHub.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu quy trình CI/CD trên AWS. <br>- Nghiên cứu cách tích hợp GitHub, AWS CodePipeline và Amazon S3 để tự động hóa quá trình build và triển khai ứng dụng. <br>- Thực hành tạo Pipeline mẫu và tìm hiểu luồng hoạt động của CI/CD. | 06/07/2026 | 06/07/2026 | [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/) |
| 3 | - Nghiên cứu Amazon CloudWatch, tìm hiểu cơ chế theo dõi tài nguyên, thu thập log và thiết lập cảnh báo. <br>- Tìm hiểu AWS Lambda và mô hình Serverless Computing, thực hành tạo một Lambda Function đơn giản. | 07/07/2026 | 07/07/2026 | [Amazon CloudWatch Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/) <br><br> [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/) |
| 4 | - Tìm hiểu Amazon Cognito và cơ chế xác thực người dùng. <br>- Nghiên cứu cách tích hợp AWS Lambda với Amazon S3 và Amazon DynamoDB để xử lý dữ liệu theo kiến trúc serverless. | 08/07/2026 | 08/07/2026 | [Amazon Cognito Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/) <br><br> [Amazon DynamoDB Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/) |
| 5 | - Bắt đầu xây dựng khung dự án FoodieRecipe: Thiết lập dự án Next.js (Social Module layout, Sidebar, Header, Dashboard layout) và NestJS (cấu hình thư mục module, controller, service kết nối database RDS). | 09/07/2026 | 09/07/2026 | [Next.js Routing](https://nextjs.org/docs/app/building-your-application/routing) <br><br> [NestJS Controllers & Modules](https://docs.nestjs.com/controllers) |
| 6 | - Triển khai cơ chế đăng nhập và phân quyền truy cập (Authentication/Authorization) cho Social Module sử dụng NestJS Guard kết hợp JWT và kết nối với màn hình đăng nhập Social Module phía Next.js. <br>- Đồng bộ toàn bộ mã nguồn lên GitHub. | 10/07/2026 | 10/07/2026 | [NestJS Security Authentication](https://docs.nestjs.com/security/authentication) |

### Kết quả đạt được tuần 3:
* Hiểu được quy trình triển khai ứng dụng theo mô hình CI/CD trên AWS, nắm được vai trò của AWS CodePipeline, GitHub và Amazon S3 trong việc tự động hóa quá trình triển khai.
* Nắm được kiến thức cơ bản về Amazon CloudWatch, AWS Lambda và Amazon Cognito, đồng thời hiểu cách các dịch vụ này phối hợp trong việc giám sát hệ thống, xử lý tác vụ serverless và quản lý xác thực người dùng.
* Tìm hiểu được phương thức tích hợp AWS Lambda, Amazon S3 và Amazon DynamoDB để xây dựng các ứng dụng theo kiến trúc serverless.
* Hoàn thành khung giao diện Social Module (Layout chung, Sidebar, Header) trên Next.js và thiết lập khung cấu trúc các module backend trên NestJS.
* Tích hợp thành công cơ chế đăng nhập và phân quyền (Authentication) bằng JWT, bảo mật luồng API Social Module với NestJS Guard và kết nối liên thông với giao diện đăng nhập Client.
* Đồng bộ toàn bộ mã nguồn lên GitHub phục vụ việc phát triển và theo dõi tiến độ của nhóm.
