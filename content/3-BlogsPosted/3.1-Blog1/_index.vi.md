---
title: "AWS Fargate vs Lambda"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AWS FARGATE VS LAMBDA – KHI NÀO CHỌN CONTAINER SERVERLESS THAY VÌ FUNCTION SERVERLESS

Trong quá trình tìm hiểu các lựa chọn compute serverless trên AWS, em nhận thấy nhiều bạn mới tiếp cận thường mặc định "serverless = Lambda", nhưng thực tế AWS có một lựa chọn serverless khác ở tầng container: AWS Fargate. Hai dịch vụ này cùng chia sẻ triết lý "không quản lý server", nhưng khác nhau về đơn vị thực thi, giới hạn tài nguyên và use case phù hợp.

### Khác biệt cốt lõi

* **Lambda**: đơn vị thực thi là function — một đoạn code ngắn, chạy trong môi trường được AWS quản lý hoàn toàn, kích hoạt theo sự kiện (event-driven).
* **Fargate**: đơn vị thực thi là container — chạy trên nền ECS hoặc EKS, cho phép đóng gói toàn bộ ứng dụng (kèm dependency, runtime tùy chỉnh) mà không cần tự quản lý EC2 instance hay cluster bên dưới.

Nói cách khác, Lambda phù hợp với các tác vụ nhỏ, ngắn hạn; Fargate phù hợp với các workload cần chạy như một "service" đúng nghĩa, với vòng đời dài hơn và yêu cầu tài nguyên linh hoạt hơn.

### Giới hạn kỹ thuật quan trọng của Lambda

* Thời gian thực thi tối đa cho mỗi lần gọi (invocation) là 15 phút (900 giây) — đây là giới hạn cứng, không thể điều chỉnh.
* Bộ nhớ có thể cấu hình từ 128 MB đến 10.240 MB, CPU được cấp phát tỉ lệ thuận theo mức bộ nhớ đã chọn.
* Với các tác vụ cần chạy lâu hơn 15 phút mà vẫn muốn giữ kiến trúc serverless, cách xử lý phổ biến là chia nhỏ logic thành nhiều function và dùng Step Functions hoặc Lambda Destinations để nối các bước lại với nhau — thay vì cố kéo dài một function duy nhất.
* Đáng chú ý, AWS gần đây đã giới thiệu Lambda durable functions cho phép một tiến trình logic tạm dừng và tiếp tục qua nhiều lần gọi, kéo dài tới một năm, phù hợp cho các workflow cần chờ phê duyệt hoặc xử lý đa ngày — nhưng bản thân mỗi lần gọi (invocation) vẫn bị giới hạn 15 phút.

### Fargate không có các giới hạn này

Vì chạy container thay vì function ngắn hạn, Fargate không bị giới hạn 15 phút — container có thể chạy liên tục (long-running service), phù hợp cho web server, API backend truyền thống được container hóa, hoặc các job xử lý kéo dài hàng giờ. Fargate cũng cho phép cấu hình tài nguyên CPU/RAM linh hoạt hơn nhiều so với trần 10GB của Lambda, phù hợp với các workload cần nhiều tài nguyên tính toán liên tục.

### Khi nào nên chọn Lambda?

* **Xử lý sự kiện rời rạc**: upload file, thay đổi dữ liệu trong DynamoDB, message từ SQS.
* **API nhẹ**, thời gian xử lý ngắn (vài giây đến vài phút).
* **Traffic không đều**, có lúc gần như bằng 0 — Lambda chỉ tính phí khi thực sự có lượt gọi, tránh lãng phí tài nguyên nhàn rỗi.
* **Nhóm muốn tối giản việc quản lý hạ tầng**, chấp nhận ràng buộc về runtime và thời gian chạy.

### Khi nào nên chọn Fargate?

* **Ứng dụng đã được đóng gói thành container (Docker)** từ trước, muốn tái sử dụng mà không viết lại theo mô hình function.
* **Workload cần chạy liên tục hoặc lâu hơn 15 phút** (ví dụ: xử lý batch nặng, worker liên tục lắng nghe queue trong thời gian dài).
* **Cần kiểm soát môi trường runtime chi tiết hơn** (nhiều tiến trình song song trong cùng container, custom OS-level dependency).
* **Traffic ổn định**, đủ lớn để mô hình tính phí theo tài nguyên cấp phát của Fargate hợp lý hơn so với tính theo từng lượt gọi của Lambda.

### Best Practices khi quyết định

* Không nên xem đây là lựa chọn nhị phân cố định cho toàn hệ thống — nhiều kiến trúc thực tế dùng cả hai: Lambda cho các tác vụ sự kiện nhẹ, Fargate cho các service lõi chạy dài hạn.
* Nếu một function Lambda thường xuyên phải chia nhỏ vì vướng giới hạn 15 phút, đó là dấu hiệu nên cân nhắc chuyển phần đó sang Fargate thay vì cố "lách" giới hạn.
* Đánh giá chi phí thực tế theo pattern traffic: traffic rời rạc/không đều thường rẻ hơn với Lambda; traffic ổn định liên tục thường rẻ hơn với Fargate.

### Kiến thức rút ra

Qua so sánh Lambda và Fargate, em hiểu rõ hơn về:
* Sự khác biệt giữa "serverless" ở tầng function và tầng container — cả hai đều serverless nhưng phục vụ nhóm bài toán khác nhau.
* Giới hạn thời gian thực thi là yếu tố quyết định quan trọng khi lựa chọn giữa hai dịch vụ.
* Trong kiến trúc enterprise thực tế, việc kết hợp nhiều dịch vụ compute theo đúng đặc điểm từng workload quan trọng hơn việc chọn một dịch vụ "tốt nhất" duy nhất.

### Kết luận

AWS Lambda và Fargate không cạnh tranh trực tiếp mà bổ sung cho nhau trong bức tranh serverless của AWS. Lambda phù hợp với các tác vụ ngắn, theo sự kiện, còn Fargate phù hợp với các service chạy dài hạn hoặc đã được container hóa sẵn. Việc lựa chọn đúng dịch vụ cho đúng loại workload giúp tối ưu cả chi phí lẫn độ phức tạp vận hành.

### Tham khảo

* AWS Lambda FAQs: [https://aws.amazon.com/lambda/faqs/](https://aws.amazon.com/lambda/faqs/)
* AWS Lambda Developer Guide – Configuring function timeout: [https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html](https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html)
* Link bài viết: [Facebook](https://web.facebook.com/groups/awsstudygroupfcj/posts/2234471933984433/?_rdc=1&_rdr#)