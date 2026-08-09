---
title: "AWS Lambda Cold Start"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# AWS LAMBDA – COLD START VÀ CÁCH TỐI ƯU (PROVISIONED CONCURRENCY, SNAPSTART)

Trong quá trình tìm hiểu về kiến trúc Serverless trên AWS, em nhận thấy AWS Lambda mang lại lợi ích lớn về khả năng mở rộng và tối ưu chi phí (chỉ trả tiền theo lượt gọi), nhưng đi kèm với đó là một vấn đề hiệu năng quan trọng cần hiểu rõ trước khi đưa vào hệ thống thực tế: Cold Start.

### Cold Start là gì?

Khi một function Lambda không được gọi trong một khoảng thời gian, hoặc khi lưu lượng tăng đột biến vượt quá số execution environment đang "ấm" (warm), AWS phải khởi tạo một execution environment hoàn toàn mới trước khi xử lý request. Quá trình này gồm các bước: cấp phát tài nguyên (microVM), tải và mount gói triển khai (deployment package), khởi động runtime (Node.js, Python, JVM...), rồi chạy code khởi tạo (init phase) trước khi mới đến handler xử lý request thực tế. Toàn bộ chuỗi này gọi là "cold start" và là phần chi phí độ trễ phát sinh ngoài thời gian xử lý logic thực sự.

Mức độ ảnh hưởng khác nhau tùy runtime: với Node.js hoặc Python, cold start thường chỉ khoảng 200–800ms; nhưng với Java (đặc biệt các ứng dụng dùng Spring Boot), thời gian khởi tạo có thể lên tới 5–15 giây — mức delay không thể chấp nhận với các API phục vụ người dùng trực tiếp.

### Hai giải pháp chính của AWS: Provisioned Concurrency và SnapStart

#### 1. Provisioned Concurrency
* **Cách hoạt động**: AWS cho phép "giữ ấm" sẵn một số lượng execution environment nhất định, đã hoàn tất init phase từ trước, sẵn sàng xử lý request ngay lập tức mà không phải trải qua cold start.
* **Ưu điểm**: Áp dụng được cho mọi runtime, không giới hạn ngôn ngữ, phù hợp cho các luồng nghiệp vụ đòi hỏi độ trễ thấp ổn định.
* **Đánh đổi**: Phải trả phí cho số lượng environment được giữ ấm ngay cả khi không có request nào đang xử lý (khác với Lambda on-demand thông thường vốn chỉ tính phí theo lượt gọi thực tế), nên cần cấu hình đúng số lượng để tránh lãng phí.

#### 2. SnapStart
* **Cách hoạt động**: Thay vì lặp lại toàn bộ quá trình khởi tạo ở mỗi cold start, Lambda chụp lại (snapshot) trạng thái bộ nhớ và ổ đĩa của execution environment ngay sau khi init phase hoàn tất, rồi lưu cache lại. Các cold start tiếp theo sẽ khôi phục (resume) trực tiếp từ snapshot này thay vì chạy lại toàn bộ init phase, giúp giảm độ trễ khởi động tới mức gần như tức thời.
* **Hiệu quả**: Có thể giảm tới 90% độ trễ init phase trong các trường hợp tối ưu, và không phát sinh thêm chi phí ngoài phần lưu trữ snapshot.

**Giới hạn quan trọng cần lưu ý khi triển khai:**
* Chỉ hỗ trợ các runtime được quản lý: Java 11 trở lên, Python 3.12 trở lên, và .NET 8 trở lên. Không hỗ trợ Node.js, Ruby, các runtime chỉ dùng hệ điều hành, hay Lambda container image.
* SnapStart và Provisioned Concurrency loại trừ lẫn nhau — không thể bật đồng thời cả hai trên cùng một function.
* Không hỗ trợ Amazon EFS, và ephemeral storage tối đa 512MB khi dùng SnapStart.
* Với Java, ARM64 cũng được hỗ trợ, mang lại thêm lợi thế về giá/hiệu năng so với x86.

Ngoài hai giải pháp trên, một số kỹ thuật bổ trợ khác cũng giúp giảm cold start: chạy trên kiến trúc Graviton (ARM64) để tăng tốc và giảm chi phí, giảm kích thước deployment package, và đưa các tác vụ nặng ra khỏi init phase.

### Khi nào cần quan tâm đến Cold Start?

Vấn đề này đặc biệt quan trọng với:
* API phục vụ người dùng trực tiếp, đòi hỏi phản hồi nhanh và nhất quán.
* Hệ thống có lưu lượng không đều, xen kẽ giữa các đợt traffic tăng đột biến và thời gian rảnh.
* Ứng dụng dùng Java hoặc các framework có init phase nặng (Spring Boot, các thư viện ML).

Ngược lại, với các tác vụ nền (background processing), xử lý bất đồng bộ qua SQS như đã tìm hiểu ở bài trước, cold start ít ảnh hưởng hơn vì người dùng không chờ phản hồi trực tiếp.

### Kiến thức rút ra

Qua tìm hiểu về Cold Start, em hiểu rõ hơn về:
* Sự đánh đổi giữa mô hình serverless "pay-per-use" thuần túy và nhu cầu độ trễ thấp ổn định.
* Cách AWS giải quyết cùng một vấn đề bằng hai cách tiếp cận khác nhau (giữ ấm sẵn vs. snapshot/resume), mỗi cách phù hợp với ngữ cảnh riêng.
* Tầm quan trọng của việc chọn đúng runtime và kiến trúc ngay từ đầu khi thiết kế hệ thống serverless nhạy cảm về độ trễ.

### Kết luận

Cold Start là chi phí ẩn dễ bị bỏ qua khi thiết kế kiến trúc serverless, nhưng có thể ảnh hưởng trực tiếp đến trải nghiệm người dùng nếu không được xử lý đúng cách. Provisioned Concurrency và SnapStart là hai công cụ bổ trợ cho nhau: một bên đảm bảo độ trễ thấp ổn định cho mọi runtime với chi phí duy trì, bên còn lại tối ưu miễn phí cho các runtime được hỗ trợ. Việc lựa chọn cần dựa trên đặc điểm traffic, runtime đang dùng và ngân sách vận hành.

### Tham khảo

* AWS Lambda Developer Guide – SnapStart: [https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)
* AWS Lambda Developer Guide – Provisioned Concurrency: [https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)
* Link bài viết: [Facebook](https://web.facebook.com/groups/awsstudygroupfcj/posts/2234083070689986/?notif_id=1785905871334772&notif_t=group_post_approved&ref=notif&_rdc=1&_rdr#)