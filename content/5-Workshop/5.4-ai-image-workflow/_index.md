---
title: "AI image workflow"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This section implements the FoodieRecipe workflow as it exists in source: **Next.js → NestJS → S3 → Rekognition → Bedrock → RDS → CloudFront**.

#### Contents

1. [Upload and optimize images through NestJS](5.4.1-upload-flow/)
2. [Detect ingredients with Rekognition](5.4.2-rekognition/)
3. [Generate and persist recipes with Bedrock](5.4.3-bedrock/)
4. [Deliver private images through CloudFront](5.4.4-cloudfront/)

#### Workflow rules

- The Frontend submits a multipart file; the Backend validates, optimizes, and uploads it to S3.
- Rekognition invokes only `DetectLabels`; do not claim moderation until that API is implemented.
- Only labels at 80% confidence or higher and outside the generic-label list become ingredients.
- Bedrock receives a text prompt, and its output must parse as recipe JSON.
- AI history supports `PENDING`, `PROCESSING`, `SUCCESS`, and `FAILED`; do not invent image states absent from the schema.
- The S3 bucket remains private; CloudFront signed URLs or S3 pre-signed GET URLs grant temporary read access.
