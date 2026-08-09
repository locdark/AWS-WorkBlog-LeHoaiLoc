---
title: "Deliver private images through Amazon CloudFront"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### 1. Image-delivery principles

- Keep **Block Public Access** enabled on the S3 bucket.
- Use an S3 REST origin with CloudFront Origin Access Control (OAC).
- Allow object reads only from the designated CloudFront distribution.
- Generate expiring signed URLs in the Backend; do not persist signed URLs.
- When CloudFront is not configured locally, return an S3 pre-signed GET URL for the same object key.

Direct S3 access must be denied:

![S3 denies direct access to the private object](/images/1-Worklog/1.2-Week2/s3-direct-url-access-denied.png)

#### 2. Create an OAC with AWS CLI

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

Create the distribution with the S3 REST endpoint, attach the OAC, select **Redirect HTTP to HTTPS**, and allow only `GET` and `HEAD`.

#### 3. Grant CloudFront bucket access

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

Do not add a public principal or disable Block Public Access.

#### 4. Create a trusted key group

1. Generate an RSA key pair and keep the private key out of Git.
2. Upload the public key to CloudFront.
3. Create a key group containing the public key.
4. Attach the key group to the cache behavior under **Trusted key groups**.
5. Store the Base64 private key in Secrets Manager.

```bash
openssl genrsa -out cloudfront-private-key.pem 2048
openssl rsa -pubout \
  -in cloudfront-private-key.pem \
  -out cloudfront-public-key.pem

base64 -w 0 cloudfront-private-key.pem
```

#### 5. Generate a signed URL in NestJS

`S3Service.getDeliveryUrl()` uses:

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

Partially supplied CloudFront configuration causes an explicit error instead of producing a broken URL. When all CloudFront values are empty, local fallback is:

```ts
return getSignedUrl(
  s3Client,
  new GetObjectCommand({ Bucket: bucket, Key: key }),
  { expiresIn },
);
```

The local S3 pre-signed GET URL grants temporary object access:

![Image loaded through an S3 pre-signed GET URL](/images/1-Worklog/1.2-Week2/s3-presigned-url-success.png)

#### 6. Allow CloudFront images in Next.js

In `web/next.config.ts`, normalize the domain and add it to `remotePatterns`:

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

The public value must be present before `pnpm build`. In the Dockerfile:

```dockerfile
ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}
RUN pnpm build
```

#### 7. Deploy Next.js to production

Next.js uses `output: "standalone"`, so run it as a container on EC2 behind Nginx instead of synchronizing `.next` to S3:

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

#### 8. Verify access and caching

```bash
# Valid signed URL
curl -I '<SIGNED_CLOUDFRONT_URL>'

# Direct S3 URL must return 403
curl -I 'https://my-foodie-ai-images.s3.<region>.amazonaws.com/ai-images/<key>.jpg'

# Inspect the distribution
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query 'Distribution.{Status:Status,Domain:DomainName}'
```

Request the signed URL repeatedly and inspect `X-Cache`. Prefer versioned or timestamped object keys to reduce invalidations.
