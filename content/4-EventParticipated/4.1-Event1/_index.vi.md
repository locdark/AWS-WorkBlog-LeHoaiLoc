---
title: "Cloud Architect"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch “Cloud Architect”

### Mục đích của sự kiện

- Giải thích vai trò và trách nhiệm của một cloud architect
- Chia sẻ nguyên tắc thiết kế hệ thống cloud an toàn và có khả năng mở rộng
- Giới thiệu các mô hình kiến trúc cloud phổ biến và sự đánh đổi giữa chúng
- Cho thấy cách tự động hóa và công cụ AI hỗ trợ triển khai cloud

### Danh sách diễn giả

- **Cloud Solutions Architect** - Chia sẻ kiến trúc tham chiếu và định hướng thiết kế
- **Platform Engineering Lead** - Trình bày thực hành về hạ tầng và governance
- **Modernization Specialist** - Nêu kinh nghiệm lập kế hoạch migration và adoption cloud

### Nội dung nổi bật

#### Vai trò của cloud architect

- Kết nối mục tiêu kinh doanh với quyết định kỹ thuật
- Định nghĩa landing zone, guardrails và tiêu chuẩn governance
- Cân bằng giữa bảo mật, chi phí, hiệu năng và sự đơn giản khi vận hành

#### Thiết kế nền tảng cloud sẵn sàng cho triển khai

Một kiến trúc cloud tốt cần bắt đầu từ nền tảng vững chắc:

- **Thiết kế mạng**: phân tách mạng, truy cập riêng tư, kiểm soát luồng ra ngoài
- **Danh tính và phân quyền**: nguyên tắc least privilege, role-based access, quản lý secrets tập trung
- **Quan sát hệ thống**: logs, metrics, traces và cảnh báo ngay từ đầu

#### Các mô hình kiến trúc được nhắc đến

- **Hub-and-spoke** để quản trị tập trung
- **Microservices** cho các năng lực nghiệp vụ triển khai độc lập
- **Serverless** cho workload theo sự kiện và tải tăng giảm thất thường
- **Multi-tier application** cho hệ thống doanh nghiệp có cấu trúc đơn giản hơn

#### Bảo mật và governance

- Dùng policy as code để áp đặt tiêu chuẩn tự động
- Bảo vệ dữ liệu bằng mã hóa khi truyền và khi lưu trữ
- Tách biệt các môi trường và kiểm soát quy trình phê duyệt triển khai
- Theo dõi chi phí bằng tagging và budget

#### Tự động hóa và triển khai

- Infrastructure as Code giúp môi trường có thể tái tạo và kiểm toán được
- CI/CD giảm lỗi thủ công và tăng tốc độ phát hành
- Template chuẩn giúp đội nhóm mở rộng mà không phải xây lại từ đầu

#### AI hỗ trợ vận hành cloud

- AI có thể hỗ trợ phân tích, viết tài liệu và xử lý sự cố nhanh hơn
- Chat-based assistant giúp so sánh phương án kiến trúc trong thời gian ngắn
- Tự động hóa nên hỗ trợ công việc lặp lại, không thay thế hoàn toàn tư duy kiến trúc

### Những gì học được

#### Tư duy chiến lược

- Luôn bắt đầu từ yêu cầu kinh doanh thay vì chọn sẵn dịch vụ cloud
- Nêu rõ các đánh đổi khi chọn mô hình kiến trúc
- Thiết kế để hệ thống dễ thay đổi trong tương lai

#### Tư duy kỹ thuật

- Xây dựng nền tảng an toàn ngay từ đầu
- Ưu tiên tự động hóa thay vì thao tác thủ công
- Đánh giá đồng thời độ tin cậy, hiệu năng và chi phí
- Đưa observability và governance vào kiến trúc từ sớm

#### Tư duy triển khai

- Sử dụng lộ trình migration theo giai đoạn thay vì thay đổi toàn bộ cùng lúc
- Kiểm chứng thiết kế bằng các proof of concept nhỏ
- Tái sử dụng template và tiêu chuẩn giữa các đội nhóm

### Ứng dụng vào công việc

- Rà soát các dự án hiện tại để tìm khoảng trống về mạng, danh tính và bảo mật
- Tạo tài liệu landing zone tham chiếu cho các workload mới
- Chuẩn hóa pipeline triển khai và cấu hình môi trường
- Bổ sung observability ngay khi đưa dịch vụ mới vào hệ thống
- Đánh giá việc dùng AI và tự động hóa cho tài liệu, phân tích và vận hành

### Trải nghiệm trong event

Tham gia sự kiện **Cloud Architect** là một trải nghiệm rất bổ ích, giúp tôi hiểu rõ hơn vai trò của kiến trúc sư cloud trong việc cân bằng giữa bảo mật, chi phí và tốc độ triển khai. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả có kinh nghiệm
- Các diễn giả đã giải thích cách cloud architect kết nối mục tiêu kinh doanh với quyết định kỹ thuật.
- Những ví dụ thực tế cho thấy vì sao governance và tiêu chuẩn kiến trúc rất quan trọng trong môi trường lớn.

#### Nhận được góc nhìn kiến trúc thực tế
- Tôi hiểu rõ hơn cách thiết kế landing zone, phân chia danh tính và tổ chức mạng dùng chung.
- Phần nói về tự động hóa giúp tôi thấy Infrastructure as Code là nền tảng quan trọng để giữ tính nhất quán.
- Tôi cũng nhận ra observability và cost control nên được đưa vào thiết kế từ đầu, không phải bổ sung sau.

#### Công cụ và tự động hóa
- Sự kiện cho thấy AI và automation có thể hỗ trợ phân tích, viết tài liệu và vận hành.
- Tôi có thêm góc nhìn về cách các đội cloud có thể làm việc nhanh hơn mà vẫn giữ được governance.

#### Kết nối và trao đổi
- Phiên thảo luận tạo cơ hội so sánh cách tiếp cận giữa các người tham dự và người làm thực tế.
- Tôi thấy rõ hơn rằng cloud architecture là sự cân bằng giữa kỷ luật kỹ thuật và tính thực dụng trong kinh doanh.

#### Bài học rút ra
- Kiến trúc cloud cần được thiết kế xoay quanh bảo mật, độ tin cậy và governance.
- Chuẩn hóa giúp giảm rủi ro vận hành và giúp đội nhóm di chuyển nhanh hơn.
- Kiến trúc tốt phải làm cho việc thay đổi trong tương lai trở nên dễ hơn, không khó hơn.

#### Một số hình ảnh khi tham gia sự kiện
![Ảnh tập thể tại sự kiện Cloud Architect](/images/4-EventParticipated/4.1-Event1/picture_event.png)

> Tổng thể, sự kiện giúp tôi hiểu rõ hơn về vai trò cloud architect và cách những quyết định thiết kế đúng đắn tạo ra hệ thống an toàn, mở rộng tốt và dễ vận hành.
