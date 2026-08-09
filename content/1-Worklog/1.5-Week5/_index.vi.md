---
title: "Worklog Tuần 5"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---


### Mục tiêu tuần 5:
* Nghiên cứu và cấu hình Amazon S3 làm kho lưu trữ hình ảnh món ăn và hình ảnh sinh ra bởi AI.
* Tích hợp Amazon Rekognition SDK vào backend NestJS để nhận diện nguyên liệu thực phẩm từ hình ảnh.
* Nghiên cứu tích hợp Amazon Bedrock (mô hình Amazon Nova Lite) để xử lý logic tự động tạo công thức nấu ăn từ danh sách nguyên liệu đã nhận diện.
* Tích hợp AWS Secrets Manager để quản lý và bảo mật tập trung các biến môi trường (credentials database, API keys).
* Nghiên cứu và hoàn thành bài viết Blog 1: *AWS Fargate vs Lambda – Container Serverless vs Function Serverless*.
* Đồng bộ mã nguồn và báo cáo tiến độ tuần lên GitHub.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu dịch vụ Amazon S3, cấu hình AWS SDK trên NestJS Backend để upload hình ảnh món ăn và ảnh AI trực tiếp lên S3 bucket. <br>- Thiết lập CORS và Access Policy cho S3 bucket để đảm bảo an toàn truy cập. | 20/07/2026 | 20/07/2026 | [Amazon S3 SDK Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |
| 3 | - Tìm hiểu Amazon Rekognition SDK. <br>- Phát triển API backend NestJS nhận diện vật thể/nguyên liệu từ hình ảnh nguyên liệu do người dùng upload và xuất ra danh sách tên nguyên liệu dạng JSON. | 20/07/2026 | 20/07/2026 | [Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html) |
| 4 | - Tìm hiểu Amazon Bedrock và mô hình Amazon Nova Lite. <br>- Thiết kế prompt chuyên sâu và tích hợp code gọi API Bedrock để tự động tạo công thức nấu ăn, hướng dẫn chế biến dựa trên các nguyên liệu được nhận diện. | 20/07/2026 | 20/07/2026 | [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html) |
| 5 | - Nghiên cứu AWS Secrets Manager. <br>- Cấu hình ứng dụng NestJS sử dụng ConfigService để nạp trực tiếp các credentials database, API keys và AWS credentials bảo mật từ Secrets Manager thay vì tệp `.env` cục bộ. | 20/07/2026 | 20/07/2026 | [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) |
| 6 | - Kiểm thử liên thông toàn bộ luồng xử lý AI (Upload ảnh -> Nhận diện Rekognition -> Bedrock Nova Lite sinh công thức -> S3 lưu ảnh kết quả). <br>- **Hoàn thành và đăng bài Blog 1: AWS Fargate vs Lambda – Khi nào chọn Container Serverless thay vì Function Serverless.** | 20/07/2026 | 20/07/2026 | [AWS Lambda vs Fargate Blog](https://aws.amazon.com/) <br><br> [GitHub Workspace](https://github.) |

### Kết quả đạt được tuần 5:
* Triển khai thành công module lưu trữ hình ảnh trên Amazon S3, cấu hình phân quyền và truy cập thông qua SDK bảo mật.
* Hoàn thành API nhận diện nguyên liệu thực phẩm tự động sử dụng Amazon Rekognition với độ chính xác cao từ hình ảnh đầu vào.
* Tích hợp thành công Amazon Bedrock (sử dụng mô hình Amazon Nova Lite) để tự động sinh công thức nấu ăn trực quan và chi tiết từ danh sách nguyên liệu đã nhận diện.
* Bảo mật hệ thống thông tin bằng cách cấu hình nạp các biến môi trường nhạy cảm trực tiếp từ AWS Secrets Manager.
* Hoàn thành bài viết Blog 1 so sánh trực quan giữa AWS Fargate và Lambda, đăng lên trang blog cá nhân của báo cáo thực tập.
* Đồng bộ toàn bộ mã nguồn tích hợp AI lên GitHub thành công.
