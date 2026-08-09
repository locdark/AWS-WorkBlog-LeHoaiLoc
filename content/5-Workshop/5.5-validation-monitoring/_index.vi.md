---
title: "Kiểm thử và giám sát"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### 1. Kiểm tra môi trường production

Mở `https://myapps.io.vn` và xác nhận domain dùng HTTPS, Next.js tải được dữ liệu từ `/api`, cookie xác thực hoạt động và ảnh private hiển thị qua URL có thời hạn.

![FoodieRecipe hoạt động trên domain production](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

```bash
curl -I https://myapps.io.vn
curl https://myapps.io.vn/api
docker ps
docker logs --tail 100 foodierecipe-web
docker logs --tail 100 foodierecipe-api
```

#### 2. Ma trận kiểm thử end-to-end

| Kịch bản | Kết quả mong đợi |
| -------- | ---------------- |
| Đăng ký và đăng nhập hợp lệ | Nhận session/token và truy cập trang thành viên |
| Sai mật khẩu hoặc token hết hạn | API trả `401`, không lộ thông tin nhạy cảm |
| Tạo/sửa/xóa công thức | RDS cập nhật đúng người sở hữu và transaction |
| Thích rồi bỏ thích | `likeCount` và trạng thái UI đồng bộ |
| Tạo/sửa/xóa bình luận | Kiểm tra quyền người viết và cây bình luận |
| Upload ảnh nguyên liệu hợp lệ | S3 có JPEG tối ưu dưới `ai-images/` |
| Rekognition không tìm thấy nguyên liệu | Không gọi Bedrock hoặc trả hướng dẫn chọn ảnh khác |
| Bedrock trả JSON hợp lệ | Recipe và AI history được lưu `SUCCESS` |
| Bedrock trả output lỗi | Transaction không tạo recipe dở dang, history `FAILED` |
| CloudFront signed URL hợp lệ/hết hạn | `200` trước hạn và bị từ chối sau hạn |
| Truy cập URL S3 trực tiếp | `403 AccessDenied` |
| Reboot EC2 | Docker và Nginx tự khởi động, health check phục hồi |

Giao diện khám phá công thức cho phép kiểm tra tìm kiếm, filter, lượt thích và danh sách dữ liệu từ API:

![Danh sách công thức và trạng thái lượt thích](/images/1-Worklog/1.4-Week4/recipe-discovery-page.png)

#### 3. Log có cấu trúc

Luồng AI hiện ghi các mốc chính:

```text
[AI] analyze-image started for user <id>
[AI] image uploaded to S3: ai-images/<file>.jpg
[AI] Rekognition returned <count> labels
[AI] extracted <count> ingredients
[AI] calling Bedrock model <model-id>
[AI] recipe transaction committed with id <recipe-id>
```

Nên bổ sung `requestId`, `userId`, `historyId`, `durationMs` và `errorCode` dưới dạng JSON để tìm kiếm bằng CloudWatch Logs Insights. Không log password, JWT, AWS credential, secret, ảnh/base64 hoặc signed URL đầy đủ.

#### 4. CloudWatch Logs

CloudWatch chứa `foodie-recipe-log` cho ứng dụng và `RDSOSMetrics` cho RDS Enhanced Monitoring:

![CloudWatch Log Groups của FoodieRecipe](/images/1-Worklog/1.8-Week8/cloudwatch-log-groups.png)

CloudWatch Agent trên EC2 cần IAM permission `logs:CreateLogStream`, `logs:DescribeLogStreams` và `logs:PutLogEvents`. Đặt retention phù hợp, ví dụ một tuần cho log workshop và một tháng cho RDS metrics.

Ví dụ truy vấn Logs Insights:

```text
fields @timestamp, @message
| filter @message like /\[AI\]/
| sort @timestamp desc
| limit 50
```

#### 5. Metrics và cảnh báo

Theo dõi:

- EC2 CPU, disk và status check.
- RDS CPU, connections, free storage và latency.
- API 4xx/5xx từ Nginx hoặc application log.
- Số lần gọi Rekognition/Bedrock thành công và thất bại.
- CloudFront requests, error rate và cache hit rate.

Tạo alarm cho EC2 status check failed, API 5xx tăng, RDS thiếu storage và Budget vượt ngưỡng. Gửi cảnh báo qua SNS topic có email subscription đã xác nhận.

#### 6. Rà soát bảo mật

1. S3 bật Block Public Access và không dùng ACL public.
2. CloudFront đọc S3 qua OAC; URL người dùng có thời hạn.
3. EC2 dùng IAM Role, không dùng static access key.
4. Secrets Manager/Parameter Store thay cho `.env.production` trên máy chủ.
5. RDS không public và chỉ nhận kết nối từ EC2 Security Group.
6. Port `3000/3001/5432` không mở trực tiếp ra Internet.
7. Domain production dùng HTTPS và `COOKIE_SECURE=true`.

#### 7. Tiêu chí hoàn thành

- Domain production, Frontend, API và RDS hoạt động ổn định.
- Đăng nhập, công thức, thích và bình luận vượt qua test chính.
- Một ảnh nguyên liệu tạo được recipe và AI history `SUCCESS`.
- Trường hợp lỗi tạo history `FAILED` và không để dữ liệu dở dang.
- URL S3 trực tiếp bị chặn; signed URL hoạt động đúng hạn.
- CloudWatch nhận log mới và container phục hồi sau reboot.
