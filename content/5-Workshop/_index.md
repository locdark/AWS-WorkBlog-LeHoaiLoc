---
title: "Workshop"
date: 2026-06-22
weight: 5
chapter: true
pre: " <b> 5. </b> "
---

# Building and deploying FoodieRecipe on AWS

#### Overview

This workshop builds **FoodieRecipe** with Next.js under `web` and NestJS under `api`. Users can register, sign in, discover, create, like, and comment on recipes. The **AI Fridge** feature accepts an ingredient image and generates a recipe automatically.

The implemented AI flow sends a multipart file to NestJS, optimizes and uploads it to Amazon S3, invokes Amazon Rekognition `DetectLabels`, normalizes ingredient labels, and sends a text prompt to Amazon Bedrock. Recipes and generation history are persisted in PostgreSQL on Amazon RDS. Private images are returned through CloudFront signed URLs, with S3 pre-signed GET URLs as the local fallback.

The production architecture runs Next.js and NestJS Docker containers on EC2 behind Nginx at `myapps.io.vn`. AWS Secrets Manager protects secrets, an IAM role supplies temporary credentials, and Amazon CloudWatch collects logs.

> **Note:** This workshop follows the current implementation: Rekognition uses only `DetectLabels`; `DetectModerationLabels` and direct pre-signed PUT uploads are not implemented.

#### Contents

1. [Workshop overview](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequisites/)
3. [Core infrastructure](5.3-core-infrastructure/)
4. [AI image workflow](5.4-ai-image-workflow/)
5. [Validation and monitoring](5.5-validation-monitoring/)
6. [Resource cleanup](5.6-cleanup/)

#### Workshop outcomes

- Prepare a private S3 bucket, IAM role, Secrets Manager, VPC, EC2, and RDS.
- Deploy Next.js and NestJS with Docker/Nginx on EC2.
- Send an image from Next.js to NestJS as multipart form data and store an optimized copy in S3.
- Detect ingredients with Rekognition and generate a recipe with Bedrock.
- Deliver private images through CloudFront signed URLs.
- Test authentication, recipes, likes, comments, the AI workflow, and CloudWatch Logs.
