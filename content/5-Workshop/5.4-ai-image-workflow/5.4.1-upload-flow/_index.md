---
title: "Upload and optimize images through NestJS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### 1. Implemented upload flow

The **AI Fridge** page lets an authenticated user select an ingredient image:

![Ingredient-image upload page](/images/1-Worklog/1.4-Week4/ai-image-upload-page.png)

![Selected ingredient image](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

Unlike a pre-signed PUT flow, the current implementation sends the file through NestJS. This lets the Backend control resizing, encoding, and the object key before writing to S3.

#### 2. Submit multipart data from Next.js

In `web/lib/ai-api.ts`:

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

The request uses the application's existing cookie/JWT authentication and is available only after sign-in.

#### 3. Receive the file in NestJS

In `api/src/modules/ai-generation/ai-generation.controller.ts`:

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

As a production hardening step, add `ParseFilePipe`, a size limit, and a `FileTypeValidator` for JPEG/PNG/WebP before the service receives the file.

#### 4. Optimize and upload to S3

`S3Service.uploadImage()` resizes and compresses the image with `sharp`:

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

For the AI workflow, `folder` is `ai-images`, producing keys such as:

```text
ai-images/1785009946812.jpg
```

Keep **Block Public Access** enabled. EC2 calls `s3:PutObject` through its IAM role; never place access keys in the image or source.

#### 5. Verify

```bash
curl -X POST 'https://myapps.io.vn/api/ai/analyze-image' \
  -H 'Authorization: Bearer <JWT>' \
  -F 'image=@ingredients.jpg'

aws s3api head-object \
  --bucket my-foodie-ai-images \
  --key ai-images/<generated-file>.jpg
```

The step succeeds when the object has `ContentType=image/jpeg`, remains private, and the request continues to Rekognition.
