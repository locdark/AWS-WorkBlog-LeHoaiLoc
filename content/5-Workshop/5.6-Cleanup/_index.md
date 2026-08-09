---
title: "Resource cleanup"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Summary

The completed workshop runs Next.js and NestJS Docker containers on EC2/Nginx and uses RDS PostgreSQL, a private S3 bucket, Rekognition, Bedrock, CloudFront signed URLs, Secrets Manager, IAM, and CloudWatch. If this is a practice environment, remove resources to prevent additional charges.

> **Danger:** The following steps delete data. Confirm the AWS account, Region, and exact resource names first, and create required snapshots or backups.

#### 1. Stop new requests

1. Stop demos and jobs that invoke Bedrock or Rekognition.
2. Stop the `foodierecipe-web` and `foodierecipe-api` containers.
3. Remove or replace the `myapps.io.vn` DNS record if the environment is retired.

#### 2. Delete CloudFront resources

1. Retrieve the distribution configuration and ETag.
2. Set `Enabled=false`, update it, and wait for `Deployed`.
3. Delete the distribution with the latest ETag.
4. Delete the trusted key group and public key after no behavior references them.
5. Delete the OAC after removing the distribution.

```bash
aws cloudfront get-distribution-config --id <distribution-id>
aws cloudfront delete-distribution \
  --id <distribution-id> \
  --if-match <etag-after-disable>
```

#### 3. Empty and delete the S3 bucket

```bash
aws s3 rm s3://my-foodie-ai-images --recursive
aws s3api delete-bucket --bucket my-foodie-ai-images
```

For a versioned bucket, delete every version and delete marker before deleting the bucket.

#### 4. Delete Backend and database resources

1. Stop and terminate EC2.
2. Release the Elastic IP if it is no longer needed.
3. Delete RDS and create a final snapshot only when data must be retained.
4. Delete the DB subnet group and Security Groups after dependencies are gone.
5. Delete the VPC, subnets, and route tables only when they were created exclusively for the workshop.

#### 5. Delete secrets, IAM resources, and logs

1. Delete `prod/foodie-recipe/db` and `prod/foodie-recipe/app` with an appropriate recovery window.
2. Remove the `/foodierecipe/prod/` Parameter Store path if present.
3. Detach policies and delete the IAM role after deleting EC2.
4. Delete `foodie-recipe-log` and unused alarms/SNS topics.
5. Do not delete `RDSOSMetrics` while another resource still uses that shared log group.

#### 6. Confirm cost status

Open **Billing and Cost Management** and review EC2, RDS, S3, CloudFront, Secrets Manager, CloudWatch, Rekognition, and Bedrock. Budget alerts can continue after cleanup because billing data is delayed.

#### Result

- No test EC2/RDS, Elastic IP, image objects, CloudFront distribution, or secrets remain.
- DNS no longer points to a deleted server.
- No workload invokes Rekognition/Bedrock or writes CloudWatch Logs.
- Required snapshots and shared resources remain intentionally.
