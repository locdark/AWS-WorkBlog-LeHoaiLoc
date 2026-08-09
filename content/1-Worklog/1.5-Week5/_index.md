---
title: "Week 5 Worklog"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Research and Configure Amazon S3 as the object storage for food images and AI-generated outputs.
* Integrate UI/UX for Social Features: News Feed, real-time comments, and notification system.
* Integrate Amazon Rekognition SDK into the NestJS backend to automatically detect ingredients from uploaded photos.
* Successfully integrated Amazon Bedrock (using the Amazon Nova Lite model) to automatically generate intuitive, detailed recipes from the recognized ingredient lists.
* Completed the UI integration for Social interactions, delivering a seamless user experience via real-time comment and news feed components.
* Integrate AWS Secrets Manager to manage and secure all sensitive environment variables (database credentials, API keys) centrally.
* Research and write Blog 1: *AWS Fargate vs Lambda – When to choose Container Serverless over Function Serverless*.
* Sync all source codes and progress reports to GitHub.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Research Amazon S3 and configure AWS SDK on the NestJS backend to upload food and AI-generated images directly to S3. <br>- Set up CORS and Bucket access policies for secure delivery. | 20/07/2026 | 20/07/2026 | [Amazon S3 SDK Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |
| 3 | - Explore Amazon Rekognition SDK. <br>- Develop backend API endpoints to analyze food items from uploaded pictures and return list arrays of identified ingredients in JSON format. | 20/07/2026 | 20/07/2026 | [Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html) |
| 4 | - Research Amazon Bedrock and the Amazon Nova Lite model. <br>- Integrate the UI display for Social features (News Feed, Comment, Like/Save) on the Next.js Frontend. <br>- Design prompt engineering flows and integrate Bedrock API calls to automatically generate recipes. | 22/07/2026 | 22/07/2026 | [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html) |
| 5 | - Research AWS Secrets Manager. <br>- Configure NestJS ConfigService to dynamically load database credentials, API keys, and AWS access credentials from Secrets Manager instead of a local `.env` file. | 20/07/2026 | 20/07/2026 | [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) |
| 6 | - Conduct E2E testing of the AI integration workflow (Image upload -> Rekognition analysis -> Bedrock recipe generation -> S3 storage). <br>- **Finalize and publish Blog 1: AWS Fargate vs Lambda – When to choose Container Serverless over Function Serverless.** | 20/07/2026 | 20/07/2026 | [AWS Lambda vs Fargate Blog](https://aws.amazon.com/) <br><br> [GitHub Workspace](https://github.) |

### Week 5 Achievements:
* Successfully deployed Amazon S3 integration for food image storage, configuring secure access policies.
* Developed backend API endpoints for ingredient detection utilizing Amazon Rekognition SDK with high reliability.
* Integrated Amazon Bedrock (Nova Lite model) to auto-generate structured, readable cooking instructions and recipes based on image detection outputs.
* Secured system configurations by migrating sensitive environment variables and credentials to AWS Secrets Manager.
* Finished and published Blog 1 comparing AWS Fargate and Lambda serverless architectures.
* Synced all AI integration modules and work logs to GitHub.
