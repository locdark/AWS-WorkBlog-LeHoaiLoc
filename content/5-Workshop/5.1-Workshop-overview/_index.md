---
title: "Workshop overview"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Introducing FoodieRecipe

FoodieRecipe is a recipe-sharing platform with accounts, search, recipe management, likes, comments, and AI recipe generation from ingredient images.

![FoodieRecipe running on the production domain](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

#### Workshop architecture

![FoodieRecipe architecture on AWS](/images/2-Proposal/foodie-recipe-architecture.png)

| Component | Responsibility |
| --------- | -------------- |
| Next.js `web` | UI, authentication, search, likes, comments, and AI image submission |
| NestJS `api` | Business APIs, image upload, AWS integrations, and persistence |
| EC2, Docker, Nginx | Run both containers and serve the production domain |
| Amazon RDS for PostgreSQL | Store users, recipes, ingredients, interactions, and AI history |
| Amazon S3 | Store images resized and converted to JPEG by the Backend |
| Amazon Rekognition | Detect image labels with `DetectLabels` |
| Amazon Bedrock | Generate recipe JSON from an ingredient prompt |
| Amazon CloudFront | Return expiring signed URLs for private images |
| Secrets Manager, IAM | Protect secrets and supply temporary credentials |
| CloudWatch | Collect application logs and RDS metrics |

#### Implemented end-to-end flow

1. A user signs in and selects an image on the **AI Fridge** page.
2. Next.js sends `multipart/form-data` to `POST /api/ai/analyze-image`.
3. NestJS uses `sharp` to resize the image to at most 1024 px, encodes JPEG at quality 85, and uploads it under `ai-images/` in S3.
4. Rekognition reads the S3 object and returns up to 30 labels with a service threshold of 50%.
5. The Backend keeps ingredient labels at 80% or higher, removes generic labels, normalizes names, and deduplicates them.
6. Bedrock receives a text prompt and returns recipe JSON.
7. NestJS validates the output and persists the recipe, ingredients, steps, and generation history in RDS.
8. On image reads, the Backend returns a CloudFront signed URL; local fallback uses an S3 pre-signed GET URL.

#### Learning objectives

- Understand the responsibility boundaries among Next.js, NestJS, and AWS services.
- Use an IAM role instead of static access keys on EC2.
- Build an S3–Rekognition–Bedrock workflow matching the source.
- Deploy containerized applications behind Nginx and a custom domain.
- Validate image access, logs, and business features.
