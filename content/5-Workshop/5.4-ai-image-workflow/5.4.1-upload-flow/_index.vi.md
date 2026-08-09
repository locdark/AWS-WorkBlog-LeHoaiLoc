---
title: "Upload và tối ưu ảnh qua NestJS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### 1. Luồng upload đang sử dụng

Trang **Tủ lạnh AI** cho phép người dùng chọn một ảnh nguyên liệu:

![Trang upload ảnh nguyên liệu](/images/1-Worklog/1.4-Week4/ai-image-upload-page.png)

![Ảnh nguyên liệu đã được chọn](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

Khác với pre-signed PUT, implementation hiện tại gửi file qua NestJS. Cách này giúp Backend kiểm soát quá trình resize, định dạng và object key trước khi ghi S3.

#### 2. Gửi multipart từ Next.js

Trong `web/lib/ai-api.ts`:

```ts
export async function analyzeImage(file: File) {
  const formData = new FormData();
  formData.append("image", file);

  const { data } = await apiClient.post(
    "/ai/analyze-image",
    formData,
    { headers: { "Content-Type": "multipart/form-data" } },
  );

  return data;
}
```

Request sử dụng cookie/JWT hiện có của ứng dụng và chỉ được gọi sau khi đăng nhập.

#### 3. Nhận file trong NestJS

Trong `api/src/modules/ai-generation/ai-generation.controller.ts`:

```ts
@Controller('ai')
@UseGuards(AuthGuard)
export class AIGenerationController {
  @Post('analyze-image')
  @UseInterceptors(FileInterceptor('image'))
  analyzeImage(
    @UploadedFile() file: Express.Multer.File,
    @CurrentUser() user: { id: bigint },
  ) {
    return this.aiService.analyzeImage(file, user.id);
  }
}
```

Nên bổ sung `ParseFilePipe`, giới hạn dung lượng và `FileTypeValidator` cho JPEG/PNG/WebP trước khi đưa file tới service.

#### 4. Tối ưu và upload S3

`S3Service.uploadImage()` dùng `sharp` để resize và nén ảnh:

```ts
const optimizedImage = await sharp(file.buffer)
  .resize({ width: 1024, withoutEnlargement: true })
  .jpeg({ quality: 85 })
  .toBuffer();

const key = `${folder}/${Date.now()}.jpg`;

await s3Client.send(new PutObjectCommand({
  Bucket: bucket,
  Key: key,
  Body: optimizedImage,
  ContentType: 'image/jpeg',
}));
```

Với workflow AI, `folder` là `ai-images`, vì vậy object key có dạng:

```text
ai-images/1785009946812.jpg
```

Bucket phải giữ **Block Public Access**. EC2 dùng IAM Role để gọi `s3:PutObject`; không khai báo access key trong image hoặc source.

#### 5. Kiểm tra

```bash
curl -X POST 'https://myapps.io.vn/api/ai/analyze-image' \
  -H 'Authorization: Bearer <JWT>' \
  -F 'image=@ingredients.jpg'

aws s3api head-object \
  --bucket my-foodie-ai-images \
  --key ai-images/<generated-file>.jpg
```

Kết quả đạt yêu cầu khi object có `ContentType=image/jpeg`, không public, và request tiếp tục sang bước Rekognition.
