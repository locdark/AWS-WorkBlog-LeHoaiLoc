---
title: "Validation and monitoring"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### 1. Validate production

Open `https://myapps.io.vn` and confirm HTTPS, Next.js data loading from `/api`, authentication cookies, and private images rendered through expiring URLs.

![FoodieRecipe running on the production domain](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

```bash
curl -I https://myapps.io.vn
curl https://myapps.io.vn/api
docker ps
docker logs --tail 100 foodierecipe-web
docker logs --tail 100 foodierecipe-api
```

#### 2. End-to-end test matrix

| Scenario | Expected result |
| -------- | --------------- |
| Valid registration and sign-in | Receive a session/token and enter member pages |
| Wrong password or expired token | API returns `401` without exposing sensitive data |
| Create/update/delete a recipe | RDS updates the correct owner and transaction |
| Like then unlike | `likeCount` and UI state remain synchronized |
| Create/update/delete a comment | Author permissions and comment tree remain correct |
| Upload a valid ingredient image | S3 contains an optimized JPEG under `ai-images/` |
| Rekognition finds no ingredients | Skip Bedrock or ask for a clearer image |
| Bedrock returns valid JSON | Persist recipe and a `SUCCESS` AI history record |
| Bedrock returns invalid output | Avoid a partial recipe and record `FAILED` history |
| Valid/expired CloudFront signed URL | Return `200` before expiry and deny access afterward |
| Direct S3 URL | Return `403 AccessDenied` |
| Reboot EC2 | Docker and Nginx restart and health checks recover |

The recipe discovery UI validates search, filters, like state, and API data:

![Recipe list and like state](/images/1-Worklog/1.4-Week4/recipe-discovery-page.png)

#### 3. Structured logging

The AI flow currently records major milestones:

```text
[AI] analyze-image started for user <id>
[AI] image uploaded to S3: ai-images/<file>.jpg
[AI] Rekognition returned <count> labels
[AI] extracted <count> ingredients
[AI] calling Bedrock model <model-id>
[AI] recipe transaction committed with id <recipe-id>
```

As an improvement, add `requestId`, `userId`, `historyId`, `durationMs`, and `errorCode` as JSON fields for CloudWatch Logs Insights. Never log passwords, JWTs, AWS credentials, secrets, image/base64 data, or complete signed URLs.

#### 4. CloudWatch Logs

CloudWatch contains `foodie-recipe-log` for the application and `RDSOSMetrics` for RDS Enhanced Monitoring:

![FoodieRecipe CloudWatch Log Groups](/images/1-Worklog/1.8-Week8/cloudwatch-log-groups.png)

The EC2 CloudWatch Agent needs `logs:CreateLogStream`, `logs:DescribeLogStreams`, and `logs:PutLogEvents`. Use appropriate retention, for example one week for workshop logs and one month for RDS metrics.

Example Logs Insights query:

```text
fields @timestamp, @message
| filter @message like /\[AI\]/
| sort @timestamp desc
| limit 50
```

#### 5. Metrics and alarms

Monitor:

- EC2 CPU, disk, and status checks.
- RDS CPU, connections, free storage, and latency.
- API 4xx/5xx from Nginx or application logs.
- Successful and failed Rekognition/Bedrock calls.
- CloudFront requests, error rate, and cache hit rate.

Create alarms for failed EC2 status checks, elevated API 5xx, low RDS storage, and budget thresholds. Notify a confirmed email subscription through SNS.

#### 6. Security review

1. S3 has Block Public Access enabled and no public ACL.
2. CloudFront reads S3 through OAC; user-facing URLs expire.
3. EC2 uses an IAM role without static access keys.
4. Secrets Manager/Parameter Store replace `.env.production` on the server.
5. RDS is private and accepts only the EC2 Security Group.
6. Ports `3000/3001/5432` are not exposed directly to the Internet.
7. The production domain uses HTTPS and `COOKIE_SECURE=true`.

#### 7. Completion criteria

- The production domain, Frontend, API, and RDS are stable.
- Authentication, recipes, likes, and comments pass core tests.
- An ingredient image produces a recipe and `SUCCESS` AI history.
- Failure produces `FAILED` history without partial data.
- Direct S3 access is denied and signed URLs respect expiry.
- CloudWatch receives new logs and containers recover after reboot.
