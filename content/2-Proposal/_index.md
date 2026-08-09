---
title: "Proposal"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# FoodieRecipe

## An AI-Enhanced Recipe-Sharing Application

## 1. Executive Summary

**FoodieRecipe** is a complete web product for a community of cooking enthusiasts. Users can create accounts, publish and manage recipes, upload food images, search by dish or ingredient, like and comment on recipes, and discover content from other contributors. The system uses **Next.js** for the Frontend, **NestJS** for the Backend, and a relational database for users, recipes, categories, ingredients, likes, comments, and images.

The product's main differentiator is an AI-assisted image pipeline: images are uploaded to Amazon S3, moderated and recognized with Amazon Rekognition, analyzed by a multimodal model through Amazon Bedrock, promoted to ready-image storage, and delivered through Amazon CloudFront for faster access. The Next.js build is stored in a private S3 web bucket and published through CloudFront, while EC2 runs NestJS in Docker behind Nginx. Amazon RDS stores relational data, Secrets Manager protects secrets, IAM controls permissions, and CloudWatch collects logs and metrics.

I implemented the project end to end, from requirements analysis, architecture design, and Next.js/NestJS development to database construction, AI-pipeline integration, and AWS deployment. The work includes IAM, S3, Rekognition, Bedrock, CloudFront, EC2, Docker, Nginx, RDS, Secrets Manager, and CloudWatch configuration; end-to-end testing; security, performance, and cost reviews; and technical documentation.

## 2. Background and Problem Statement

### 2.1. Background

Traditional recipe platforms often require users to manually enter a dish name, description, ingredients, and tags. As content grows, moderating inappropriate images, standardizing metadata, and maintaining fast image delivery become increasingly difficult.

### 2.2. Problems to solve

- Publishing a recipe requires multiple manual input fields.
- User-uploaded images may have invalid formats, excessive size, or inappropriate content.
- The system cannot automatically identify dishes, ingredients, and relevant tags.
- Serving images directly from storage may increase origin latency and requests.
- AI output may be inaccurate, malformed, or unnecessarily expensive when invoked repeatedly.

### 2.3. Proposed solution

FoodieRecipe provides the core capabilities of a recipe-sharing platform: account, recipe, ingredient, category, search, like, comment, and content-administration functions. During recipe publishing, the NestJS Backend issues a pre-signed URL so Next.js can upload directly to Amazon S3. Rekognition performs label detection and moderation; accepted images and high-confidence labels are sent to Bedrock for dish-name, description, ingredient, and tag suggestions. Users review, edit, and confirm suggestions before saving. Images are displayed through CloudFront rather than direct S3 URLs.

## 3. Project Objectives

### 3.1. General objective

Build FoodieRecipe as a complete recipe-sharing product while developing and evaluating an intelligent image pipeline that reduces manual input, improves content quality, and accelerates image access.

### 3.2. Specific objectives

- Build an accessible image-upload experience with Next.js.
- Build image-lifecycle APIs with NestJS.
- Support account, recipe, ingredient, category, search, like, and comment management.
- Store application data in a relational database with clear constraints.
- Store private images in S3 using consistent object keys and metadata.
- Use Rekognition for label detection and content moderation.
- Use Bedrock to suggest structured recipe information.
- Use CloudFront to cache and deliver S3 images.
- Apply least-privilege IAM, validation, fallback, and human review.
- Measure latency and the cost drivers of AI service usage.

## 4. Project Scope

### 4.1. Complete product scope

- User registration, sign-in, profile management, and user/administrator roles.
- Recipe creation, viewing, editing, and deletion.
- Ingredient, instruction, serving, cooking-time, and category management.
- Recipe search, filtering, and pagination.
- Like or unlike recipes and display the total count and current user's like state.
- Create, view, and delete comments according to ownership; administrators can moderate inappropriate comments.
- Management and moderation of user-submitted recipe content.
- Relational data storage and NestJS APIs.
- An operational architecture using CloudFront, one S3 image bucket split into `uploads/` and `delivery/` prefixes, EC2, Docker, Nginx, RDS, Secrets Manager, IAM, and CloudWatch.
- AI-assisted image recognition, moderation, and content suggestions.

### 4.2. Implemented work scope

- Analyze requirements, design the complete architecture, and model FoodieRecipe data.
- Build the Next.js Frontend in `web` and the NestJS Backend in `api`.
- Implement authentication, authorization, users, recipes, ingredients, categories, search, likes, and comments.
- Create Amazon RDS for PostgreSQL, design schemas and migrations, and connect it securely to the Backend.
- Create the S3 image bucket and configure object keys, CORS, lifecycle, encryption, and access control.
- Implement direct uploads with pre-signed URLs and validate format, size, metadata, and ownership.
- Integrate Rekognition for label detection, moderation, and normalized results.
- Integrate Bedrock for structured dish-name, description, ingredient, and tag suggestions.
- Allow users to review, edit, or reject AI suggestions.
- Create EC2, attach an IAM role, package NestJS with Docker, and configure Nginx as a reverse proxy.
- Manage database credentials and sensitive configuration with Secrets Manager.
- Configure CloudFront with a private S3 origin, signed URLs, and cache policies.
- Configure CloudWatch Logs, metrics, alarms, and dashboards for observability.
- Perform end-to-end tests, handle retries/timeouts/failures, review security, optimize cost, and prepare handover documentation.

### 4.3. Product exclusions

- Building payments, delivery, or e-commerce capabilities.
- Training a custom computer-vision model.
- Guaranteeing perfect AI accuracy or replacing human moderation.
- Self-hosting AI models or building dedicated model-training infrastructure.

## 5. Users and Core Features

| User             | Need                                   | Related features                                                               |
| ---------------- | -------------------------------------- | ------------------------------------------------------------------------------ |
| Member           | Discover and interact with recipes     | Browse, search, like, comment, and load images through CloudFront              |
| Contributor      | Publish recipes with less manual input | Upload images, receive AI suggestions, edit, confirm, and monitor interactions |
| Administrator    | Control image quality and safety       | Review flagged images, Rekognition results, and processing states              |
| Development team | Trace and resolve pipeline failures    | Inspect image IDs, states, timing, and normalized errors                       |

## 6. Solution Architecture

![FoodieRecipe Overall Architecture](/images/2-Proposal/foodie-recipe-architecture.png)

### 6.1. Complete product architecture

- **Frontend:** Next.js provides the user interface, calls the NestJS API, and loads images through CloudFront.
- **Backend:** NestJS runs in Docker on EC2, with Nginx acting as the API reverse proxy.
- **Data:** Amazon RDS stores users, recipes, ingredients, categories, likes, comments, and image metadata.
- **Images:** one S3 image bucket uses `uploads/` for originals and `delivery/` for ready images served through CloudFront.
- **Web deployment:** the static Next.js export is stored in a private S3 web bucket and delivered through CloudFront over HTTPS.
- **AI:** NestJS invokes Rekognition for detection/moderation and Bedrock for recipe suggestions.
- **Security:** an IAM role grants AWS permissions to EC2, while Secrets Manager stores sensitive configuration.
- **Monitoring:** CloudWatch and CloudWatch Logs collect system and application metrics and logs.

### 6.2. Numbered flows in the diagram

| Step | Flow                                                                                                                                                                           |
| :--: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|  1   | The user accesses the Next.js website and FoodieRecipe content through CloudFront.                                                                                             |
|  2   | CloudFront retrieves ready images from the S3 image bucket's `delivery/` prefix and caches them at edge locations.                                                             |
|  3   | The Next.js application calls the NestJS API on EC2 through Nginx.                                                                                                             |
|  4   | Next.js uploads an image directly to the image S3 bucket with a Backend-issued pre-signed URL.                                                                                 |
|  5   | NestJS verifies the image under `uploads/`, reads metadata, and manages its lifecycle. After successful processing, the image is moved to `delivery/` for CloudFront delivery. |
|  6   | NestJS reads and writes business data and image states in Amazon RDS.                                                                                                          |
|  7   | NestJS invokes Amazon Bedrock to analyze the image and suggest recipe content.                                                                                                 |
|  8   | NestJS invokes Amazon Rekognition for label detection and image moderation.                                                                                                    |
|  9   | The EC2 application retrieves database credentials and sensitive configuration from Secrets Manager.                                                                           |

IAM and CloudWatch are cross-cutting components: IAM determines which services EC2 may call, while CloudWatch receives logs and metrics from EC2, Docker, Nginx, and NestJS.

### 6.3. Image-processing workflow

1. Next.js sends file information to NestJS to create an image record.
2. NestJS checks authorization, generates an object key, and returns an expiring pre-signed URL.
3. The browser uploads directly to S3 and calls the confirmation API.
4. NestJS verifies the object, sets the state to `processing`, and invokes Rekognition.
5. Rekognition returns labels and moderation labels with confidence scores.
6. An inappropriate image moves to `failed` with a moderation reason code.
7. For an accepted image, NestJS sends the image and labels to a multimodal model through Bedrock.
8. The Backend validates the JSON output; the user reviews and confirms the suggestion.
9. NestJS returns a CloudFront URL for Next.js to display the ready image.

### 6.4. Image states

| State        | Meaning                                                                          |
| ------------ | -------------------------------------------------------------------------------- |
| `pending`    | Record created; image is waiting for or currently uploading                      |
| `processing` | Upload completed; Rekognition or Bedrock is processing the image                 |
| `completed`  | Processing completed; the image may be displayed through CloudFront              |
| `failed`     | Upload, AI processing, or moderation failed; details are stored in an error code |

## 7. Component Design

### 7.1. Next.js Frontend

- Upload component supporting file selection and drag-and-drop.
- Image preview, size/type validation, and progress display.
- NestJS integration for pre-signed URLs and upload confirmation.
- Poll or refresh analysis state by image ID.
- Editable AI suggestions before persistence.
- CloudFront URLs, suitable image dimensions, and lazy loading.

### 7.2. NestJS Backend

- `ImageModule` manages controllers, services, DTOs, and image states.
- S3 service creates pre-signed URLs and checks metadata and object deletion.
- Rekognition service performs label detection and moderation.
- Bedrock service builds prompts, invokes the model, and validates output.
- Standardized exceptions, limited retries, and idempotency by image ID.
- AWS credentials and internal configuration never reach the browser.

### 7.3. Amazon S3

- The **`uploads/` prefix** receives originals from Next.js through pre-signed URLs and permits Backend processing only.
- The **`delivery/` prefix** stores approved images for CloudFront delivery.
- The image bucket remains private with Block Public Access enabled; its bucket policy allows CloudFront to read only `delivery/*`.
- The S3 web bucket stores the static Next.js export, remains private, and allows only its web distribution through Origin Access Control.
- Proposed object key: `recipes/{userId}/{recipeId}/{imageId}-{version}.{ext}`.
- Suitable metadata includes `Content-Type`, owner ID, recipe ID, and checksum.
- Encryption is enabled; versioning/lifecycle handles temporary, failed, and old image versions.

### 7.4. Amazon Rekognition

- `DetectLabels` identifies dishes, ingredients, and related objects.
- `DetectModerationLabels` flags potentially inappropriate content.
- Confidence thresholds are configured and evaluated using a test image set.
- Uncertain results use a human-review path.

### 7.5. Amazon Bedrock

- Uses a multimodal model capable of image understanding.
- Prompts combine the image with top Rekognition labels.
- Expected output includes `dishName`, `description`, `ingredients`, `tags`, and `confidence`.
- The Backend validates the JSON schema and applies fallback handling.
- Results are cached by image hash and prompt version to reduce repeated calls.

### 7.6. Amazon CloudFront

- S3 is a private origin accessed with Origin Access Control.
- The image behavior permits only the required read methods.
- Versioned/hashed object keys support a long cache TTL.
- Updating the key minimizes the need for invalidation.
- The web distribution serves `index.html` and Next.js static assets, while the image distribution serves objects under `delivery/`.

### 7.7. Product runtime infrastructure

- The S3 image bucket stores originals under `uploads/`; `delivery/` is the private origin path from which CloudFront distributes ready images.
- Amazon EC2 runs NestJS in Docker, with Nginx serving as the reverse proxy and API entry point.
- Amazon RDS stores users, recipes, ingredients, categories, likes, comments, and image metadata.
- AWS Secrets Manager supplies secrets without embedding them in source code or the Docker image.
- The EC2 IAM role provides temporary access to S3, RDS, Bedrock, Rekognition, Secrets Manager, and CloudWatch.
- CloudWatch collects metrics, while CloudWatch Logs centralizes NestJS, Nginx, and container logs.
- The complete infrastructure is deployed, configured, and tested as part of the FoodieRecipe implementation.

## 8. Non-Functional Requirements

### 8.1. Security

- IAM policies follow least privilege.
- AWS access keys are never stored in the repository or Frontend.
- Pre-signed URLs are short-lived and bound to specific object keys.
- The S3 bucket is private; images are read through CloudFront.
- The S3 web bucket is private; the website is accessed through CloudFront over HTTPS.
- EC2 exposes only required ports, and Nginx controls requests to NestJS. RDS is private and accepts only Backend connections.
- The application retrieves credentials from Secrets Manager through an IAM role instead of using static access keys on EC2.
- Data is transmitted over HTTPS, and secrets are managed outside source code.
- Files are checked by MIME type, extension, size, and signature.
- Logs exclude credentials, tokens, complete pre-signed URLs, and sensitive data.

### 8.2. Performance

- The browser uploads directly to S3.
- CloudFront caches images at edge locations to reduce latency and origin load.
- Next.js uses lazy loading and appropriate image dimensions.
- Rekognition and Bedrock timing is measured separately to identify bottlenecks.

### 8.3. Reliability

- Every stage uses one image ID and an explicit state machine.
- Retries are limited, use backoff, and do not create duplicate objects.
- A cleanup task handles `pending` or `failed` records and objects that remain too long.
- AI output is always validated and requires user confirmation.

## 9. Implementation Plan

Planned duration: **eight weeks, from June 22, 2026 to August 14, 2026**.

The following table covers the complete eight-week FoodieRecipe implementation, from application development to AWS deployment and operation.

| Week | Work                                                                             | Milestone                                                           |
| :--: | -------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
|  1   | Study AWS, IAM, Budget Alerts, and image-pipeline design                         | Initial requirements, scope, and diagram                            |
|  2   | Design S3, RDS, Secrets Manager, and the data model                              | Defined data, storage, and access infrastructure                    |
|  3   | Build NestJS APIs, authentication, recipes, likes, comments, and pre-signed URLs | Working Backend and data lifecycle                                  |
|  4   | Build Next.js and integrate it with NestJS                                       | Working account, recipe, interaction, search, and upload interfaces |
|  5   | Integrate Rekognition                                                            | Working label detection and moderation                              |
|  6   | Integrate Bedrock                                                                | Structured, validated recipe suggestions                            |
|  7   | Create EC2/RDS, deploy NestJS with Docker/Nginx, and configure CloudFront        | API and images served from AWS infrastructure                       |
|  8   | Deploy the Frontend, configure CloudWatch, test, and document                    | Observable end-to-end product                                       |

## 10. Acceptance Criteria

### 10.1. Product-level criteria

- Users can register, sign in, and manage their profiles.
- Authorized users can create, view, search, edit, and delete recipes.
- Signed-in users can like or unlike a recipe without creating duplicate records.
- Users can view and create comments; only the owner or an administrator can delete them according to authorization rules.
- Recipes support ingredients, instructions, categories, cooking time, servings, and images.
- Administrators can review and manage inappropriate recipes or images.
- The Next.js Frontend communicates reliably with NestJS, and relational data remains consistent.
- The Next.js website is built, synchronized to the S3 web bucket, and successfully accessed through CloudFront.
- Next.js and NestJS are deployed, connected through Nginx, and operate reliably on AWS infrastructure.
- EC2, Docker, Nginx, S3, RDS, Bedrock, Rekognition, CloudFront, Secrets Manager, IAM, and CloudWatch are configured and tested according to the architecture.

### 10.2. AI image-processing criteria

- A user can select and upload a valid image from Next.js.
- The image uploads directly to the correct S3 object key with a pre-signed URL.
- Invalid files are rejected with clear messages.
- Rekognition labels and moderation results are stored in normalized form.
- An inappropriate image never moves to `completed`.
- Bedrock returns the expected schema or the Backend applies a safe fallback.
- A user can edit AI-generated content before confirmation.
- A ready image is accessed through CloudFront without exposing its direct S3 URL.
- Retries do not create duplicate records or objects, and errors are traceable by image ID.
- Documentation covers architecture, APIs, states, security, and testing.

## 11. Cost and Budget Control

The complete product has two main cost groups:

- **Application-platform cost:** EC2, Amazon RDS, the S3 image bucket, CloudFront, Secrets Manager, CloudWatch, network transfer, logs, and backups.
- **AI image-feature cost:** S3 storage/requests, CloudFront, Rekognition analysis volume, the selected Bedrock model, input/output size, and data transfer.

Before operating the product, I enter the expected user count, recipe count, image volume, and AI invocation frequency into AWS Pricing Calculator. Project tags separate platform and AI costs for independent evaluation.

Cost controls include:

- Configure AWS Budget Alerts and tag costs by project.
- Limit image size and per-user image count.
- Invoke AI only after upload confirmation and validation.
- Cache results by checksum/image hash and prompt version.
- Limit retries, output tokens, and concurrent requests.
- Use lifecycle rules for failed, temporary, or old images.
- Monitor CloudFront cache hits to reduce S3 requests.
- Right-size EC2/RDS and select an operating model appropriate for actual traffic.
- Set log/backup retention and remove unused test resources.

## 12. Risks and Mitigations

| Risk                                           | Impact                                         | Mitigation                                                                     |
| ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------ |
| EC2, Nginx, or RDS becomes unavailable         | The overall application is interrupted         | Tested health checks, backups, CloudWatch, and restart or recovery procedures  |
| AI identifies the wrong dish or ingredients    | Incorrect suggestions                          | Label results as suggestions, expose confidence, and require user confirmation |
| Rekognition moderation false positive/negative | Valid images blocked or unsafe images accepted | Use conservative thresholds and store `failed` with a reason for manual review |
| Bedrock returns malformed output               | Backend cannot process the result              | JSON schema validation, limited retry, and fallback                            |
| Upload is interrupted                          | Records and objects become inconsistent        | Explicit states, confirmation API, and cleanup task                            |
| CloudFront serves an old image                 | Stale content appears                          | Versioned keys, suitable TTL, and targeted invalidation                        |
| AWS credentials or sensitive URLs leak         | Data-security risk                             | Environment variables, least privilege, short-lived URLs, and log redaction    |
| AI usage exceeds the budget                    | Unexpected cost                                | Budget Alerts, quotas, caching, retry limits, and usage tracking               |

## 13. Expected Outcomes

- A complete FoodieRecipe product supporting account, recipe, ingredient, category, search, like, and comment management.
- A deployed architecture covering the application, data, image storage, AI, and content-delivery layers.
- A complete image pipeline with clear responsibilities across Next.js, NestJS, and AWS.
- Less manual entry for dish names, descriptions, ingredients, and tags.
- Automated first-pass image screening before content is displayed.
- Faster image access through CloudFront caching.
- A foundation for image search, meal suggestions, and personalization.
- A fully deployed end-to-end product with monitoring, operational documentation, and troubleshooting procedures.

## 14. Future Enhancements

- Search for recipes using an image or ingredient list.
- Suggest dishes from ingredients already available to the user.
- Generate cooking instructions for serving size and dietary preferences.
- Support multiple images per recipe and automatic image resizing.
- Evaluate AI suggestion quality from user feedback.
- Add a dashboard for AI accuracy, latency, usage, and cost.
