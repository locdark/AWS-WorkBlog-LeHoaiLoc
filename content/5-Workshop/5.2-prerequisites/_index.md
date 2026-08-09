---
title: "Prerequisites"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### 1. Account and Region

1. Sign in with a workshop IAM user/role rather than the root user.
2. Select a Region supporting Amazon Rekognition and the intended Bedrock model.
3. Record the **Account ID** and **Region** for resource naming.
4. Create an AWS Budget Alert before starting.

> **Warning:** Amazon Bedrock model availability and access depend on the Region and account. Confirm model access before section 5.4.3.

#### 2. Local tools

Install and verify:

```bash
node --version
npm --version
aws --version
docker --version
git --version
```

Use a current Node.js LTS release, AWS CLI v2, and a stable Docker Desktop/Engine release.

#### 3. Configure the AWS CLI

```bash
aws configure
aws sts get-caller-identity
aws configure get region
```

Never commit credentials, `.env` files, or pre-signed URLs to Git.

#### 4. Prepare the source code

Reference structure:

```text
foodierecipe/
├── web/            # Next.js web application
├── api/            # NestJS API
├── docs/
└── .gitignore
```

Keep `api/.env.example` limited to key names for local setup; never add real values:

```dotenv
# Database
DATABASE_URL=

# App
PORT=
NODE_ENV=
# Set false for HTTP deployments; set true when the API is served over HTTPS.
COOKIE_SECURE=

# JWT / auth (optional placeholders)
JWT_SECRET=
JWT_EXPIRES_IN=

# S3 bucket
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=

# CloudFront private delivery (leave all values empty to use S3 presigned URLs locally)
CLOUDFRONT_DOMAIN=
CLOUDFRONT_KEY_PAIR_ID=
# Base64 of the PEM private key. Never commit the real key.
CLOUDFRONT_PRIVATE_KEY_BASE64=
CLOUDFRONT_URL_EXPIRES_IN=

# Email verification (Resend)
RESEND_API_KEY=
EMAIL_FROM=

BEDROCK_MODEL_ID=
```

> **Warning:** All `.env.example` values are placeholders. Never commit the real `.env`. Production does not create `.env.production`: EC2 uses an IAM role for AWS credentials, Secrets Manager for sensitive values, and Parameter Store for non-sensitive configuration. Set `COOKIE_SECURE=true` when serving the API over HTTPS.

- `DATABASE_URL` points to local PostgreSQL on port `5433`.
- Leave `CLOUDFRONT_*` empty locally; `api` returns an S3 pre-signed GET URL.
- `CLOUDFRONT_PRIVATE_KEY_BASE64` contains the Base64-encoded private key, not raw PEM text.
- `RESEND_API_KEY` and `EMAIL_FROM` are required only when email verification is enabled.

`web/.env.example`:

```dotenv
NEXT_PUBLIC_API_URL=
NEXT_PUBLIC_APP_NAME=
NEXT_PUBLIC_GOOGLE_CLIENT_ID=
NEXT_PUBLIC_CLOUDFRONT_DOMAIN=
```

Supply `NEXT_PUBLIC_*` variables before building Next.js because their values are embedded in the client bundle.

#### 5. Required permissions

The principal running the workshop needs permission to create/manage the used S3, CloudFront, EC2, IAM role, RDS, Secrets Manager, CloudWatch Logs, Rekognition, and Bedrock resources.

> **Note:** Do not grant `AdministratorAccess` to the application. In section 5.3.1, you will create a dedicated least-privilege EC2 role.

#### 6. Resource naming

| Resource | Suggested name |
| -------- | -------------- |
| Image bucket | `my-foodie-ai-images` or an equivalent globally unique name |
| EC2 role | `FoodieRecipeBackendRole` |
| EC2 Security Group | `FoodieRecipeEc2Sg` |
| RDS Security Group | `FoodieRecipeRdsSg` |
| Database secret | `prod/foodie-recipe/db` |
| Application secret | `prod/foodie-recipe/app` |
| Log group | `foodie-recipe-log` |
| Production domain | `myapps.io.vn` |

After completing the checks, continue to [Build the core infrastructure](../5.3-core-infrastructure/).
