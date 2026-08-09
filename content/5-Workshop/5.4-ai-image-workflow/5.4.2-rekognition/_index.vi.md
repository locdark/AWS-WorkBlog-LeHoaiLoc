---
title: "Nhận diện nguyên liệu với Amazon Rekognition"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### 1. Cấp quyền Rekognition

IAM Role của EC2 cần quyền tối thiểu:

```json
{
  "Effect": "Allow",
  "Action": "rekognition:DetectLabels",
  "Resource": "*"
}
```

Role đồng thời cần `s3:GetObject` cho `arn:aws:s3:::my-foodie-ai-images/ai-images/*` vì Rekognition đọc object bằng bucket/key.

#### 2. Gọi `DetectLabels`

Trong `api/src/common/aws/rekognition.service.ts`:

```ts
async detectLabels(imageKey: string) {
  return this.client.send(new DetectLabelsCommand({
    Image: {
      S3Object: {
        Bucket: this.configService.getOrThrow('AWS_BUCKET_NAME'),
        Name: imageKey,
      },
    },
    MaxLabels: 30,
    MinConfidence: 50,
  }));
}
```

`MinConfidence: 50` giới hạn kết quả do Rekognition trả về; Backend tiếp tục áp dụng ngưỡng chặt hơn khi chọn nguyên liệu.

#### 3. Chuẩn hóa nhãn nguyên liệu

`IngredientService` thực hiện bốn bước:

1. Chỉ giữ label có confidence từ `80%`.
2. Loại nhãn chung như `Food`, `Meal`, `Dish`, `Plate`, `Fruit` và `Vegetable`.
3. Chuẩn hóa tên tương đương, ví dụ `Scallion` thành `Green Onion`.
4. Loại trùng và sắp xếp confidence giảm dần.

```ts
labels
  .filter((label) => (label.Confidence ?? 0) >= 80)
  .filter((label) => !ignoreLabels.has(label.Name!))
  .map((label) => ({
    name: normalize(label.Name!),
    confidence: Number((label.Confidence ?? 0).toFixed(2)),
  }))
  .filter((item, index, self) =>
    self.findIndex((value) => value.name === item.name) === index,
  )
  .sort((a, b) => b.confidence - a.confidence);
```

#### 4. Kết quả thực tế

Giao diện hiển thị các label và confidence Rekognition trả về:

![Kết quả nhận diện label từ Rekognition](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

Các nhãn được lưu cùng lịch sử tạo công thức để truy vết input AI:

![Nhãn Rekognition được lưu trong lịch sử](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.5-Week5/rekognition-labels-history.png)

#### 5. Xử lý lỗi và giới hạn hiện tại

| Trường hợp | Cách xử lý |
| ---------- | ---------- |
| Object S3 không tồn tại | Ghi log có context và trả lỗi chuẩn hóa |
| Region/bucket sai | Kiểm tra `AWS_REGION`, `AWS_BUCKET_NAME` và IAM Role |
| Không có label phù hợp | Không gọi Bedrock hoặc yêu cầu người dùng chọn ảnh rõ hơn |
| Rekognition timeout/throttling | Retry có giới hạn và backoff |

> **Cảnh báo:** Source hiện tại không gọi `DetectModerationLabels`. Không mô tả ảnh đã được kiểm duyệt hoặc bị Rekognition từ chối nếu chưa bổ sung service, policy và test tương ứng.

#### 6. Kiểm tra

1. Dùng ảnh có nguyên liệu rõ ràng và xác nhận có label confidence.
2. Dùng ảnh không liên quan thực phẩm và xác nhận hệ thống không tạo danh sách nguyên liệu sai.
3. Dùng object key không tồn tại để kiểm tra log lỗi.
4. Kiểm tra lịch sử AI lưu model, labels, trạng thái và recipe ID.
