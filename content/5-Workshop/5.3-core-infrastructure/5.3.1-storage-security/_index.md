---
title: "Prepare S3, IAM, and Secrets Manager"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### 1. Create the S3 image bucket

1. Open **Amazon S3 → Create bucket**.
2. Choose a globally unique name such as `my-foodie-ai-images`.
3. Select the same Region as EC2, Rekognition, and Bedrock when the model supports it.
4. Keep **Object Ownership: Bucket owner enforced** and disable ACLs.
5. Enable every **Block Public Access** setting and default encryption.

![The my-foodie-ai-images S3 bucket](/images/1-Worklog/1.2-Week2/s3-bucket-created.png)

![Block Public Access enabled on the bucket](/images/1-Worklog/1.2-Week2/s3-block-public-access.png)

The Backend stores AI images under `ai-images/`; recipe images and avatars may use separate folders in the same private bucket.

> **Note:** The implemented workflow uploads through NestJS, not directly from the browser to S3. The image bucket therefore does not require CORS for pre-signed PUT. Add CORS only if a direct-upload flow is implemented later.

#### 2. Lifecycle and access

Optionally add a lifecycle rule for test images or unreferenced objects. Do not use a broad deletion rule for active recipe images.

Minimum Backend policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "FoodieRecipeImageObjects",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::my-foodie-ai-images/*"
    },
    {
      "Sid": "DetectIngredients",
      "Effect": "Allow",
      "Action": "rekognition:DetectLabels",
      "Resource": "*"
    },
    {
      "Sid": "GenerateRecipe",
      "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "arn:aws:bedrock:<region>::foundation-model/<model-id>"
    },
    {
      "Sid": "ReadApplicationSecrets",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": [
        "arn:aws:secretsmanager:<region>:<account-id>:secret:prod/foodie-recipe/db-*",
        "arn:aws:secretsmanager:<region>:<account-id>:secret:prod/foodie-recipe/app-*"
      ]
    }
  ]
}
```

Attach the policy to the EC2 IAM role. Do not create access keys for the role or grant `AdministratorAccess` to the application.

#### 3. Create secrets

Create two secrets:

- `prod/foodie-recipe/db`: `DATABASE_URL` or RDS connection fields.
- `prod/foodie-recipe/app`: `JWT_SECRET` and `CLOUDFRONT_PRIVATE_KEY_BASE64`.

![FoodieRecipe secrets in Secrets Manager](/images/1-Worklog/1.2-Week2/secrets-manager-created.png)

Non-sensitive values such as bucket name, model ID, CloudFront domain, and TTL may be stored in Systems Manager Parameter Store.

#### 4. Verify

```bash
aws sts get-caller-identity
aws s3api get-public-access-block --bucket my-foodie-ai-images
aws secretsmanager describe-secret --secret-id prod/foodie-recipe/db
```

The step succeeds when the bucket is private, the EC2 role has only required permissions, and no real secret exists in Git or a Docker image.
