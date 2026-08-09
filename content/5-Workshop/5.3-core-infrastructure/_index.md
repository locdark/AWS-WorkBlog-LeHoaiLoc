---
title: "Build the Core Infrastructure"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This section creates and deploys the foundation for NestJS to store images, invoke AI services, and persist FoodieRecipe data.

#### Contents

1. [Create S3, IAM, and Secrets Manager](5.3.1-storage-security/)
2. [Prepare NestJS, EC2, and Amazon RDS](5.3.2-backend-data/)
3. [Deploy FoodieRecipe to production](5.3.3-production-deployment/)

#### Architecture after this section

- One private S3 image bucket with an `ai-images/` prefix is ready.
- EC2 has a least-privilege IAM role.
- Database credentials are stored in Secrets Manager.
- NestJS runs in Docker behind Nginx and can connect to RDS.
- Next.js runs as a standalone container, and `myapps.io.vn` is served over HTTPS.
- Migrations have created the database schema, and a health check verifies the deployed API.

> **Note:** Complete S3, IAM, Secrets Manager, EC2, RDS, Docker, and Nginx before implementing the AI workflow. Configure and verify CloudWatch in section 5.5.
