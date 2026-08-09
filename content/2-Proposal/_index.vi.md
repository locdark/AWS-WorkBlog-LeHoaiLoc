---
title: "Đề xuất dự án"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# FoodieRecipe

## Ứng dụng chia sẻ công thức nấu ăn tích hợp AI

## 1. Tóm tắt đề xuất

**FoodieRecipe** là một sản phẩm web hoàn chỉnh dành cho cộng đồng quan tâm đến nấu ăn. Người dùng có thể đăng ký tài khoản, tạo và quản lý công thức, đăng ảnh món ăn, tìm kiếm theo tên món hoặc nguyên liệu, thích và bình luận về công thức, đồng thời khám phá nội dung từ những người dùng khác. Hệ thống sử dụng **Next.js** cho Frontend, **NestJS** cho Backend và cơ sở dữ liệu quan hệ để quản lý người dùng, công thức, danh mục, nguyên liệu, lượt thích, bình luận và hình ảnh.

Điểm nổi bật của sản phẩm là pipeline xử lý ảnh tích hợp AI: ảnh được upload lên Amazon S3, kiểm duyệt và nhận diện bằng Amazon Rekognition, phân tích bằng mô hình đa phương thức qua Amazon Bedrock, sau đó đưa sang vùng lưu trữ ảnh sẵn sàng và phân phối qua Amazon CloudFront để cải thiện tốc độ truy cập. Bản build Next.js được lưu trong S3 web bucket và phát hành qua CloudFront; NestJS chạy trong Docker trên EC2 sau Nginx, Amazon RDS lưu dữ liệu quan hệ, Secrets Manager quản lý bí mật, IAM kiểm soát quyền và CloudWatch thu thập log, metrics.

Toàn bộ dự án được em thực hiện xuyên suốt từ phân tích yêu cầu, thiết kế kiến trúc, phát triển Frontend Next.js và Backend NestJS đến xây dựng cơ sở dữ liệu, tích hợp pipeline AI và triển khai hệ thống trên AWS. Công việc bao gồm cấu hình IAM, S3, Rekognition, Bedrock, CloudFront, EC2, Docker, Nginx, RDS, Secrets Manager và CloudWatch; kiểm thử end-to-end; rà soát bảo mật, hiệu năng, chi phí; và hoàn thiện tài liệu kỹ thuật.

## 2. Bối cảnh và vấn đề

### 2.1. Bối cảnh

Các nền tảng công thức truyền thống thường yêu cầu người dùng nhập thủ công tên món, mô tả, nguyên liệu và tag. Khi số lượng bài viết tăng, việc kiểm tra ảnh không phù hợp, chuẩn hóa dữ liệu và đảm bảo tốc độ tải ảnh trở nên khó khăn.

### 2.2. Vấn đề cần giải quyết

- Quy trình đăng công thức có nhiều trường nhập liệu và tốn thời gian.
- Ảnh do người dùng tải lên có thể sai định dạng, quá lớn hoặc không phù hợp.
- Hệ thống khó tự động xác định món ăn, nguyên liệu và tag liên quan.
- Việc cung cấp trực tiếp ảnh từ kho lưu trữ có thể làm tăng độ trễ và request về origin.
- Kết quả AI có thể thiếu chính xác, không đúng cấu trúc hoặc phát sinh chi phí nếu gọi lặp lại.

### 2.3. Giải pháp đề xuất

FoodieRecipe cung cấp đầy đủ chức năng cốt lõi của một nền tảng chia sẻ công thức: quản lý tài khoản, công thức, nguyên liệu, danh mục, tìm kiếm, lượt thích, bình luận và quản trị nội dung. Trong luồng đăng công thức, Backend NestJS cấp pre-signed URL để Next.js upload ảnh trực tiếp lên Amazon S3. Rekognition nhận diện nhãn và kiểm duyệt; ảnh hợp lệ cùng các nhãn có độ tin cậy cao được chuyển cho Bedrock để gợi ý tên món, mô tả, nguyên liệu và tag. Người dùng xem lại, chỉnh sửa và xác nhận trước khi lưu. Ảnh được hiển thị thông qua CloudFront thay vì URL S3 trực tiếp.

## 3. Mục tiêu dự án

### 3.1. Mục tiêu tổng quát

Xây dựng FoodieRecipe thành một sản phẩm chia sẻ công thức hoàn chỉnh, đồng thời phát triển và đánh giá pipeline xử lý ảnh thông minh nhằm giảm thao tác nhập liệu, nâng cao chất lượng nội dung và cải thiện tốc độ tải ảnh.

### 3.2. Mục tiêu cụ thể

- Xây dựng giao diện upload ảnh dễ sử dụng bằng Next.js.
- Xây dựng API quản lý vòng đời ảnh bằng NestJS.
- Hỗ trợ quản lý tài khoản, công thức, nguyên liệu, danh mục, tìm kiếm, lượt thích và bình luận.
- Lưu dữ liệu nghiệp vụ trong cơ sở dữ liệu quan hệ với ràng buộc rõ ràng.
- Lưu ảnh riêng tư trên S3 với object key và metadata nhất quán.
- Dùng Rekognition nhận diện label và kiểm duyệt nội dung.
- Dùng Bedrock gợi ý dữ liệu công thức có cấu trúc.
- Dùng CloudFront cache và phân phối ảnh từ S3.
- Áp dụng IAM least privilege, validation, fallback và human review.
- Theo dõi latency và các yếu tố ảnh hưởng đến chi phí dịch vụ AI.

## 4. Phạm vi dự án

### 4.1. Phạm vi sản phẩm hoàn chỉnh

- Đăng ký, đăng nhập, quản lý hồ sơ và phân quyền người dùng/quản trị viên.
- Tạo, xem, chỉnh sửa và xóa công thức nấu ăn.
- Quản lý nguyên liệu, bước thực hiện, khẩu phần, thời gian nấu và danh mục.
- Tìm kiếm, lọc và phân trang công thức.
- Thích hoặc bỏ thích công thức; hiển thị tổng lượt thích và trạng thái thích của người dùng hiện tại.
- Tạo, xem và xóa bình luận theo quyền sở hữu; quản trị viên có thể kiểm duyệt bình luận không phù hợp.
- Quản lý và kiểm duyệt nội dung công thức do người dùng đăng tải.
- Lưu dữ liệu quan hệ và cung cấp API qua NestJS.
- Kiến trúc vận hành gồm CloudFront, một S3 image bucket tách prefix `uploads/` và `delivery/`, EC2, Docker, Nginx, RDS, Secrets Manager, IAM và CloudWatch.
- Tích hợp AI để nhận diện, kiểm duyệt và gợi ý nội dung từ ảnh.

### 4.2. Phạm vi công việc thực hiện

- Phân tích yêu cầu, thiết kế kiến trúc tổng thể và mô hình dữ liệu FoodieRecipe.
- Xây dựng Frontend Next.js trong `web` và Backend NestJS trong `api`.
- Xây dựng xác thực, phân quyền, quản lý người dùng, công thức, nguyên liệu, danh mục, tìm kiếm, lượt thích và bình luận.
- Tạo Amazon RDS for PostgreSQL, thiết kế schema, migration và kết nối an toàn từ Backend.
- Tạo S3 image bucket; cấu hình object key, CORS, lifecycle, encryption và quyền truy cập.
- Xây dựng upload trực tiếp bằng pre-signed URL; kiểm tra định dạng, dung lượng, metadata và quyền sở hữu ảnh.
- Tích hợp Rekognition để nhận diện nhãn, kiểm duyệt ảnh và chuẩn hóa kết quả.
- Tích hợp Bedrock để gợi ý tên món, mô tả, nguyên liệu và tag có cấu trúc.
- Cho phép người dùng xem lại, chỉnh sửa hoặc từ chối nội dung AI gợi ý.
- Tạo EC2, gắn IAM Role, đóng gói NestJS bằng Docker và cấu hình Nginx làm reverse proxy.
- Quản lý database credential và thông tin nhạy cảm bằng Secrets Manager.
- Cấu hình CloudFront với S3 private origin, signed URL và cache policy để phân phối ảnh.
- Cấu hình CloudWatch Logs, metrics, alarm và dashboard để theo dõi ứng dụng.
- Kiểm thử end-to-end, xử lý retry/timeout/lỗi, rà soát bảo mật, tối ưu chi phí và viết tài liệu bàn giao.

### 4.3. Ngoài phạm vi sản phẩm

- Xây dựng hệ thống thanh toán, giao hàng hoặc thương mại điện tử.
- Tự huấn luyện mô hình thị giác máy tính riêng.
- Cam kết AI nhận diện chính xác tuyệt đối hoặc thay thế kiểm duyệt thủ công.
- Tự vận hành mô hình AI hoặc xây dựng hạ tầng huấn luyện mô hình riêng.

## 5. Người dùng và chức năng chính

| Đối tượng | Nhu cầu | Chức năng liên quan |
| --------- | ------- | ------------------- |
| Người dùng | Khám phá và tương tác với công thức | Xem, tìm kiếm, thích, bình luận và tải ảnh qua CloudFront |
| Người đóng góp | Đăng công thức với ít thao tác nhập liệu | Upload ảnh, nhận gợi ý AI, chỉnh sửa, xác nhận và theo dõi tương tác |
| Người quản trị | Kiểm soát chất lượng và nội dung ảnh | Xem ảnh cần review, kết quả Rekognition và trạng thái xử lý |
| Nhóm phát triển | Theo dõi và xử lý lỗi pipeline | Kiểm tra image ID, trạng thái, thời gian xử lý và lỗi chuẩn hóa |

## 6. Kiến trúc giải pháp

![Kiến trúc tổng thể FoodieRecipe](/images/2-Proposal/foodie-recipe-architecture.png)

### 6.1. Kiến trúc sản phẩm tổng thể

- **Frontend:** Next.js cung cấp giao diện người dùng, gọi NestJS API và tải ảnh thông qua CloudFront.
- **Backend:** NestJS chạy trong Docker trên EC2; Nginx làm reverse proxy cho API.
- **Dữ liệu:** Amazon RDS lưu dữ liệu người dùng, công thức, nguyên liệu, danh mục, lượt thích, bình luận và metadata ảnh.
- **Ảnh:** một S3 image bucket dùng prefix `uploads/` cho ảnh gốc và `delivery/` cho ảnh đã sẵn sàng làm origin cho CloudFront.
- **Triển khai web:** static export Next.js nằm trong private S3 web bucket và được CloudFront phân phối qua HTTPS.
- **AI:** NestJS gọi Rekognition để nhận diện/kiểm duyệt và Bedrock để gợi ý nội dung công thức.
- **Bảo mật:** IAM role cấp quyền AWS cho EC2; Secrets Manager lưu thông tin nhạy cảm.
- **Giám sát:** CloudWatch và CloudWatch Logs thu thập metrics, log hệ thống và log ứng dụng.

### 6.2. Diễn giải các luồng trên sơ đồ

| Bước | Luồng xử lý |
| :--: | ----------- |
| 1 | Người dùng truy cập website Next.js và nội dung FoodieRecipe thông qua CloudFront. |
| 2 | CloudFront lấy ảnh đã sẵn sàng từ prefix `delivery/` của S3 image bucket và cache tại edge location. |
| 3 | Ứng dụng Next.js gọi API NestJS chạy trên EC2 thông qua Nginx. |
| 4 | Next.js upload ảnh trực tiếp lên S3 image bucket bằng pre-signed URL do Backend cấp. |
| 5 | NestJS xác minh ảnh trong prefix `uploads/`, đọc metadata và quản lý vòng đời ảnh. Sau khi xử lý thành công, ảnh được chuyển sang prefix `delivery/` để CloudFront phân phối. |
| 6 | NestJS đọc/ghi dữ liệu nghiệp vụ và trạng thái ảnh trên Amazon RDS. |
| 7 | NestJS gọi Amazon Bedrock để phân tích ảnh và gợi ý nội dung công thức. |
| 8 | NestJS gọi Amazon Rekognition để nhận diện nhãn và kiểm duyệt ảnh. |
| 9 | Ứng dụng trên EC2 lấy database credential và cấu hình nhạy cảm từ Secrets Manager. |

IAM và CloudWatch là các thành phần xuyên suốt: IAM quyết định EC2 được phép gọi dịch vụ nào; CloudWatch tiếp nhận log/metrics từ EC2, Docker, Nginx và NestJS.

### 6.3. Luồng xử lý ảnh

1. Next.js gửi thông tin file đến NestJS để tạo image record.
2. NestJS kiểm tra quyền, sinh object key và trả pre-signed URL có thời hạn.
3. Trình duyệt upload trực tiếp ảnh lên S3 và gọi API xác nhận.
4. NestJS xác minh object, chuyển trạng thái sang `processing` và gọi Rekognition.
5. Rekognition trả label và moderation label kèm confidence score.
6. Ảnh không phù hợp chuyển sang `failed` và lưu mã lý do kiểm duyệt.
7. Với ảnh hợp lệ, NestJS gửi ảnh và label đến mô hình đa phương thức qua Bedrock.
8. Backend validate kết quả JSON; người dùng xem lại và xác nhận gợi ý.
9. NestJS trả CloudFront URL để Next.js hiển thị ảnh đã sẵn sàng.

### 6.4. Trạng thái ảnh

| Trạng thái | Ý nghĩa |
| ---------- | ------- |
| `pending` | Đã tạo record, đang chờ hoặc đang upload ảnh |
| `processing` | Ảnh đã upload và đang được Rekognition/Bedrock xử lý |
| `completed` | Xử lý hoàn tất và ảnh có thể hiển thị qua CloudFront |
| `failed` | Upload, xử lý AI hoặc kiểm duyệt thất bại; chi tiết nằm trong mã lỗi |

## 7. Thiết kế thành phần

### 7.1. Next.js Frontend

- Component upload hỗ trợ chọn file và drag-and-drop.
- Preview ảnh, kiểm tra dung lượng/định dạng và hiển thị progress.
- Tích hợp API NestJS để lấy pre-signed URL và xác nhận upload.
- Poll hoặc refresh trạng thái phân tích theo image ID.
- Hiển thị gợi ý AI ở dạng có thể chỉnh sửa trước khi lưu.
- Dùng CloudFront URL, kích thước ảnh phù hợp và lazy loading.

### 7.2. NestJS Backend

- `ImageModule` quản lý controller, service, DTO và trạng thái ảnh.
- S3 service tạo pre-signed URL, kiểm tra, đọc metadata và xóa object.
- Rekognition service gọi nhận diện label và kiểm duyệt.
- Bedrock service xây prompt, gọi mô hình và validate output.
- Chuẩn hóa exception, retry giới hạn và idempotency theo image ID.
- Không trả AWS credential hoặc thông tin nội bộ cho trình duyệt.

### 7.3. Amazon S3

- Prefix **`uploads/`** nhận ảnh gốc từ Next.js qua pre-signed URL và chỉ cho Backend xử lý.
- Prefix **`delivery/`** lưu ảnh đã được xác nhận để CloudFront phân phối.
- Image bucket giữ trạng thái private và bật Block Public Access; bucket policy chỉ cho CloudFront đọc `delivery/*`.
- S3 web bucket lưu static export Next.js, giữ private và chỉ cho web distribution đọc bằng Origin Access Control.
- Object key đề xuất: `recipes/{userId}/{recipeId}/{imageId}-{version}.{ext}`.
- Lưu `Content-Type`, owner ID, recipe ID và checksum trong metadata phù hợp.
- Bật encryption; áp dụng versioning/lifecycle để dọn ảnh tạm, ảnh lỗi và version cũ.

### 7.4. Amazon Rekognition

- `DetectLabels` dùng để nhận diện món ăn, nguyên liệu và đối tượng liên quan.
- `DetectModerationLabels` dùng để phát hiện nội dung có thể không phù hợp.
- Confidence threshold được cấu hình và đánh giá qua bộ ảnh thử nghiệm.
- Trường hợp không chắc chắn được chuyển sang human review.

### 7.5. Amazon Bedrock

- Dùng mô hình đa phương thức có khả năng hiểu ảnh.
- Prompt kết hợp ảnh với top label từ Rekognition.
- Output dự kiến gồm `dishName`, `description`, `ingredients`, `tags` và `confidence`.
- Backend kiểm tra JSON schema và dùng fallback khi response không hợp lệ.
- Kết quả được cache theo image hash và prompt version để hạn chế gọi lặp.

### 7.6. Amazon CloudFront

- S3 là private origin, CloudFront truy cập bằng Origin Access Control.
- Chỉ cho phép method cần thiết để đọc ảnh.
- Dùng object key có version/hash để tận dụng TTL dài.
- Hạn chế invalidation bằng cách thay đổi key khi ảnh được cập nhật.
- Web distribution phục vụ `index.html` và static asset Next.js; image distribution phục vụ ảnh trong `delivery/`.

### 7.7. Hạ tầng vận hành sản phẩm

- S3 image bucket lưu ảnh gốc dưới `uploads/`; prefix `delivery/` là private origin path để CloudFront phân phối ảnh đã sẵn sàng.
- Amazon EC2 chạy NestJS trong Docker; Nginx làm reverse proxy và điểm vào API.
- Amazon RDS lưu người dùng, công thức, nguyên liệu, danh mục, lượt thích, bình luận và metadata hình ảnh.
- AWS Secrets Manager cung cấp bí mật cho ứng dụng mà không đưa chúng vào source code hoặc image Docker.
- IAM role gắn với EC2 cung cấp quyền tạm thời để truy cập S3, RDS, Bedrock, Rekognition, Secrets Manager và CloudWatch.
- CloudWatch thu thập metrics; CloudWatch Logs tập trung log của NestJS, Nginx và container.
- Toàn bộ hạ tầng trên được triển khai, cấu hình và kiểm thử trong quá trình thực hiện FoodieRecipe.

## 8. Yêu cầu phi chức năng

### 8.1. Bảo mật

- IAM policy tuân thủ nguyên tắc least privilege.
- Không lưu AWS access key trong repository hoặc Frontend.
- Pre-signed URL có thời hạn ngắn và gắn với object key cụ thể.
- S3 bucket không public; ảnh được đọc qua CloudFront.
- S3 web bucket không public; website được truy cập qua CloudFront HTTPS.
- EC2 chỉ mở các cổng cần thiết; Nginx kiểm soát request đến NestJS. RDS không public và chỉ nhận kết nối từ Backend.
- Ứng dụng lấy credential từ Secrets Manager bằng IAM role, không dùng access key cố định trên EC2.
- Dữ liệu truyền qua HTTPS và thông tin bí mật được quản lý ngoài mã nguồn.
- File được kiểm tra MIME type, phần mở rộng, kích thước và chữ ký file.
- Log không chứa credential, token, pre-signed URL đầy đủ hoặc dữ liệu nhạy cảm.

### 8.2. Hiệu năng

- Ảnh được upload trực tiếp từ trình duyệt lên S3.
- CloudFront cache ảnh tại edge location để giảm latency và tải origin.
- Next.js sử dụng lazy loading và kích thước ảnh phù hợp.
- Rekognition và Bedrock được đo thời gian riêng để xác định bottleneck.

### 8.3. Độ tin cậy

- Các bước sử dụng chung image ID và state machine rõ ràng.
- Retry có giới hạn, backoff và không tạo duplicate object.
- Có tác vụ dọn record/object ở trạng thái `pending` hoặc `failed` quá lâu.
- AI output luôn được validate và cần người dùng xác nhận.

## 9. Kế hoạch thực hiện

Thời gian dự kiến: **8 tuần, từ 22/06/2026 đến 14/08/2026**.

Bảng dưới đây mô tả kế hoạch thực hiện toàn bộ FoodieRecipe trong tám tuần, từ phát triển ứng dụng đến triển khai và vận hành trên AWS.

| Tuần | Nội dung | Mốc hoàn thành |
| :--: | -------- | -------------- |
| 1 | Tìm hiểu AWS, IAM, Budget Alert và thiết kế pipeline ảnh | Yêu cầu, phạm vi và sơ đồ ban đầu |
| 2 | Thiết kế S3, RDS, Secrets Manager và mô hình dữ liệu | Hạ tầng dữ liệu, lưu trữ và quyền truy cập được xác định |
| 3 | Xây NestJS API, xác thực, công thức, lượt thích, bình luận và pre-signed URL | Backend cùng vòng đời dữ liệu hoạt động |
| 4 | Xây Next.js và tích hợp với NestJS | Giao diện tài khoản, công thức, tương tác, tìm kiếm và upload hoạt động |
| 5 | Tích hợp Rekognition | Label detection và moderation hoạt động |
| 6 | Tích hợp Bedrock | Gợi ý công thức có cấu trúc và validation |
| 7 | Tạo EC2/RDS, deploy NestJS bằng Docker/Nginx và cấu hình CloudFront | API và ảnh được cung cấp từ hạ tầng AWS |
| 8 | Deploy Frontend, cấu hình CloudWatch, kiểm thử và tài liệu | Sản phẩm end-to-end hoạt động và có thể giám sát |

## 10. Tiêu chí nghiệm thu

### 10.1. Tiêu chí ở cấp sản phẩm

- Người dùng có thể đăng ký, đăng nhập và quản lý hồ sơ.
- Người dùng có thể tạo, xem, tìm kiếm, chỉnh sửa và xóa công thức theo quyền.
- Người dùng đã đăng nhập có thể thích hoặc bỏ thích một công thức mà không tạo bản ghi trùng lặp.
- Người dùng có thể xem và tạo bình luận; chỉ chủ sở hữu hoặc quản trị viên được xóa bình luận theo quyền.
- Công thức hỗ trợ nguyên liệu, bước nấu, danh mục, thời gian, khẩu phần và ảnh.
- Quản trị viên có thể kiểm tra và quản lý công thức hoặc hình ảnh không phù hợp.
- Frontend Next.js giao tiếp ổn định với Backend NestJS và dữ liệu quan hệ được lưu nhất quán.
- Website Next.js được build, đồng bộ lên S3 web bucket và truy cập thành công qua CloudFront.
- Next.js và NestJS được triển khai, kết nối qua Nginx và hoạt động ổn định trên hạ tầng AWS.
- EC2, Docker, Nginx, S3, RDS, Bedrock, Rekognition, CloudFront, Secrets Manager, IAM và CloudWatch được cấu hình và kiểm thử theo kiến trúc.

### 10.2. Tiêu chí cho phần xử lý ảnh AI

- Người dùng chọn và upload được ảnh hợp lệ từ Next.js.
- Ảnh được upload trực tiếp lên đúng S3 object key bằng pre-signed URL.
- File không hợp lệ bị từ chối với thông báo rõ ràng.
- Rekognition trả label và moderation result được lưu ở dạng chuẩn hóa.
- Ảnh không phù hợp không được chuyển sang trạng thái `completed`.
- Bedrock trả gợi ý đúng schema hoặc Backend sử dụng fallback an toàn.
- Người dùng có thể chỉnh sửa nội dung AI trước khi xác nhận.
- Ảnh sẵn sàng được truy cập qua CloudFront, không expose S3 URL trực tiếp.
- Retry không tạo duplicate record/object và lỗi được ghi nhận theo image ID.
- Tài liệu mô tả kiến trúc, API, trạng thái, bảo mật và quy trình kiểm thử.

## 11. Chi phí và kiểm soát ngân sách

Chi phí sản phẩm hoàn chỉnh gồm hai nhóm chính:

- **Chi phí nền tảng ứng dụng:** EC2, Amazon RDS, S3 image bucket, CloudFront, Secrets Manager, CloudWatch, lưu lượng mạng, log và bản sao lưu.
- **Chi phí tính năng ảnh AI:** dung lượng/request S3, CloudFront, số ảnh phân tích bằng Rekognition, model Bedrock, kích thước input/output và truyền dữ liệu.

Trước khi đưa sản phẩm vào vận hành, em nhập số người dùng, số công thức, dung lượng ảnh và tần suất gọi AI dự kiến vào AWS Pricing Calculator. Chi phí được tách theo tag thành chi phí nền tảng và chi phí AI để đánh giá độc lập.

Các biện pháp kiểm soát:

- Thiết lập AWS Budget Alert và theo dõi chi phí theo tag dự án.
- Giới hạn kích thước và số ảnh mỗi người dùng.
- Chỉ gọi AI sau khi upload được xác nhận và ảnh vượt qua validation.
- Cache kết quả theo checksum/image hash và prompt version.
- Giới hạn số lần retry, token output và số request đồng thời.
- Dùng lifecycle để xóa ảnh lỗi, ảnh tạm hoặc version cũ.
- Theo dõi cache hit của CloudFront để giảm request về S3.
- Right-size EC2/RDS và lựa chọn chế độ vận hành phù hợp với lưu lượng thực tế.
- Đặt retention cho log/backup và loại bỏ tài nguyên thử nghiệm không sử dụng.

## 12. Rủi ro và phương án giảm thiểu

| Rủi ro | Ảnh hưởng | Giảm thiểu |
| ------ | --------- | ---------- |
| EC2, Nginx hoặc RDS không khả dụng | Ứng dụng tổng thể bị gián đoạn | Health check, backup, CloudWatch và quy trình khởi động hoặc khôi phục đã kiểm thử |
| AI nhận diện sai món hoặc nguyên liệu | Nội dung gợi ý không chính xác | Hiển thị dạng gợi ý, confidence và yêu cầu người dùng xác nhận |
| Rekognition moderation false positive/negative | Ảnh hợp lệ bị chặn hoặc ảnh xấu lọt qua | Dùng ngưỡng thận trọng; lưu `failed` kèm lý do để hỗ trợ kiểm tra thủ công |
| Bedrock trả output sai schema | Backend không xử lý được | JSON schema validation, retry giới hạn và fallback |
| Upload bị gián đoạn | Record và object không đồng bộ | Trạng thái rõ ràng, confirm API và cleanup job |
| CloudFront hiển thị ảnh cũ | Nội dung không cập nhật | Versioned object key, TTL phù hợp và invalidation chọn lọc |
| Lộ AWS credential hoặc URL nhạy cảm | Mất an toàn dữ liệu | Biến môi trường, least privilege, URL ngắn hạn và redaction log |
| Chi phí AI tăng ngoài dự kiến | Vượt ngân sách | Budget Alert, quota, cache, giới hạn retry và theo dõi usage |

## 13. Kết quả kỳ vọng

- Có một sản phẩm FoodieRecipe hoàn chỉnh cho phép quản lý tài khoản, công thức, nguyên liệu, danh mục, tìm kiếm, lượt thích và bình luận.
- Kiến trúc đã triển khai mô tả đầy đủ lớp ứng dụng, dữ liệu, lưu trữ ảnh, AI và phân phối nội dung.
- Có một pipeline xử lý ảnh hoàn chỉnh, tách rõ trách nhiệm giữa Next.js, NestJS và AWS.
- Giảm thời gian nhập tên món, mô tả, nguyên liệu và tag khi đăng công thức.
- Tự động sàng lọc một phần nội dung ảnh trước khi hiển thị.
- Cải thiện tốc độ truy cập ảnh nhờ CloudFront cache.
- Tạo nền tảng có thể mở rộng cho tìm kiếm theo hình ảnh, gợi ý món ăn và cá nhân hóa.
- Toàn bộ sản phẩm được triển khai end-to-end, có giám sát, tài liệu vận hành và quy trình xử lý sự cố.

## 14. Hướng phát triển

- Tìm kiếm công thức bằng ảnh hoặc danh sách nguyên liệu.
- Gợi ý món dựa trên nguyên liệu người dùng đang có.
- Sinh hướng dẫn nấu theo khẩu phần và chế độ ăn.
- Hỗ trợ nhiều ảnh cho một công thức và tối ưu kích thước ảnh tự động.
- Xây dựng cơ chế đánh giá chất lượng gợi ý AI từ phản hồi người dùng.
- Bổ sung dashboard theo dõi accuracy, latency, usage và chi phí AI.
