---
title: "Phân phối ảnh private qua Amazon CloudFront"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### 1. Nguyên tắc phân phối ảnh

- S3 bucket giữ **Block Public Access**.
- CloudFront sử dụng S3 REST origin và Origin Access Control (OAC).
- Bucket policy chỉ cho CloudFront distribution được chỉ định đọc object.
- Backend tạo signed URL có thời hạn; không lưu signed URL trong database.
- Khi chưa cấu hình CloudFront ở local, Backend trả S3 pre-signed GET URL cho cùng object key.

Truy cập URL S3 trực tiếp phải bị từ chối:

![S3 từ chối truy cập object private](/images/1-Worklog/1.2-Week2/s3-direct-url-access-denied.png)

#### 2. Tạo OAC bằng AWS CLI

```json
{
  "Name": "foodierecipe-images-oac",
  "Description": "OAC for the private FoodieRecipe image bucket",
  "SigningProtocol": "sigv4",
  "SigningBehavior": "always",
  "OriginAccessControlOriginType": "s3"
}
```

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config file://oac-config.json
```

Tạo distribution với S3 REST endpoint, gắn OAC, đặt **Viewer protocol policy: Redirect HTTP to HTTPS** và chỉ cho phép `GET`, `HEAD`.

#### 3. Bucket policy cho CloudFront

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowFoodieRecipeCloudFrontRead",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-foodie-ai-images/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
        }
      }
    }
  ]
}
```

Không thêm public principal và không tắt Block Public Access.

#### 4. Tạo trusted key group

1. Tạo RSA key pair; giữ private key ngoài Git.
2. Upload public key vào CloudFront.
3. Tạo key group chứa public key.
4. Gắn key group vào cache behavior dưới **Trusted key groups**.
5. Lưu private key dạng Base64 trong Secrets Manager.

```bash
openssl genrsa -out cloudfront-private-key.pem 2048
openssl rsa -pubout \
  -in cloudfront-private-key.pem \
  -out cloudfront-public-key.pem

base64 -w 0 cloudfront-private-key.pem
```

#### 5. Tạo signed URL trong NestJS

`S3Service.getDeliveryUrl()` dùng cấu hình:

```text
CLOUDFRONT_DOMAIN
CLOUDFRONT_KEY_PAIR_ID
CLOUDFRONT_PRIVATE_KEY_BASE64
CLOUDFRONT_URL_EXPIRES_IN
```

```ts
return getCloudFrontSignedUrl({
  url: `https://${domain}/${encodedKey}`,
  keyPairId,
  privateKey,
  dateLessThan: new Date(Date.now() + expiresIn * 1000).toISOString(),
});
```

Nếu chỉ một phần cấu hình CloudFront được cung cấp, service trả lỗi cấu hình thay vì âm thầm tạo URL sai. Nếu tất cả biến đều trống, local fallback là:

```ts
return getSignedUrl(
  s3Client,
  new GetObjectCommand({ Bucket: bucket, Key: key }),
  { expiresIn },
);
```

S3 pre-signed GET URL local có thể đọc object trong thời hạn cho phép:

![Ảnh được đọc bằng S3 pre-signed GET URL](/images/1-Worklog/1.2-Week2/s3-presigned-url-success.png)

#### 6. Cho phép ảnh CloudFront trong Next.js

Trong `web/next.config.ts`, chuẩn hóa domain và thêm vào `remotePatterns`:

```ts
const cloudFrontDomain = process.env.NEXT_PUBLIC_CLOUDFRONT_DOMAIN
  ?.trim()
  .replace(/^https?:\/\//, '')
  .replace(/\/$/, '');

images: {
  remotePatterns: cloudFrontDomain
    ? [{ protocol: 'https', hostname: cloudFrontDomain, pathname: '/**' }]
    : [],
}
```

Biến public phải được truyền trước `pnpm build`. Với Dockerfile:

```dockerfile
ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}
RUN pnpm build
```

#### 7. Deploy Next.js production

Next.js dùng `output: "standalone"`, vì vậy deploy bằng container trên EC2 sau Nginx, không sync `.next` lên S3:

```bash
docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://myapps.io.vn/api \
  --build-arg NEXT_PUBLIC_CLOUDFRONT_DOMAIN=<cloudfront-domain> \
  -t foodierecipe-web:production web

docker run -d \
  --name foodierecipe-web \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  foodierecipe-web:production
```

#### 8. Kiểm tra quyền và cache

```bash
# Signed URL hợp lệ
curl -I '<SIGNED_CLOUDFRONT_URL>'

# URL S3 trực tiếp phải trả 403
curl -I 'https://my-foodie-ai-images.s3.<region>.amazonaws.com/ai-images/<key>.jpg'

# Kiểm tra distribution
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query 'Distribution.{Status:Status,Domain:DomainName}'
```

Gọi signed URL nhiều lần và kiểm tra `X-Cache`. Dùng object key có phiên bản hoặc timestamp để tránh phải invalidation thường xuyên.
