---
title: "Week 6 Worklog"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Design a secure network infrastructure with Amazon VPC (Public/Private Subnets, Internet Gateway, Security Groups).
* Deploy Amazon RDS Instance as the primary project database, design and optimize the database schema using Prisma for Social features (User Relations, Likes, Comments).
* Containerize application services (Next.js Frontend & NestJS Backend) using Docker and deploy them on Amazon EC2 via Docker Compose.
* Integrate Amazon CloudFront CDN in front of S3 to optimize static image delivery.
* Configure Amazon CloudWatch Logs to collect and aggregate runtime logs from Docker containers on EC2.
* Write Blog 2: *AWS Technical Update: Control and Commercialize AI Bot Traffic with AWS WAF* and Blog 3: *AWS Lambda – Cold Start and Optimization (Provisioned Concurrency, SnapStart)*.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Design Amazon VPC architecture and configure Security Groups. <br>- Provision RDS Database Instance, design and run Prisma Migration for the database schema integrating Social entities (Followers, Comments, Likes). <br>- Verify database connections from local/VPC. | 27/07/2026 | 27/07/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/) <br><br> [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/) |
| 3 | - Write optimized Dockerfiles for both Next.js and NestJS. <br>- Provision an Amazon EC2 Instance running Ubuntu in the VPC's Public Subnet, and install Docker Engine along with Docker Compose. | 27/07/2026 | 27/07/2026 | [Docker Documentation](https://docs.docker.com/) <br><br> [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/) |
| 4 | - Write the `docker-compose.yml` orchestrator file for the project and configure Nginx as a Reverse Proxy. <br>- Deploy the multi-container stack to EC2 and point database hosts to RDS. | 27/07/2026 | 27/07/2026 | [Docker Compose Docs](https://docs.docker.com/compose/) <br><br> [Nginx Reverse Proxy Guide](https://nginx.org/en/docs/) |
| 5 | - Configure Amazon CloudFront CDN pointing to Amazon S3 to accelerate static image delivery. <br>- Install and set up the CloudWatch Logs agent on EC2 to automatically ship logs from Docker containers. <br>- **Finalize and publish Blog 2: AWS Technical Update: Control and Commercialize AI Bot Traffic with AWS WAF.** | 27/07/2026 | 27/07/2026 | [Amazon CloudFront Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/) <br><br> [AWS WAF Bot Control News](https://aws.amazon.com/blogs/aws/) |
| 6 | - Perform load and performance tests, optimize Nginx configurations, and monitor computing metrics via CloudWatch dashboard graphs. <br>- **Finalize and publish Blog 3: AWS LAMBDA – COLD START AND OPTIMIZATION (PROVISIONED CONCURRENCY, SNAPSTART).** | 27/07/2026 | 27/07/2026 | [Amazon CloudWatch Logs Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/) <br><br> [AWS Lambda SnapStart Docs](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html) |

### Week 6 Achievements:
* Successfully designed and built a secure VPC network architecture, placing the RDS database in a Private Subnet to enhance security.
* Successfully designed and optimized the Database Schema for complex Social features using Prisma.
* Completed Docker containerization and successfully deployed Next.js & NestJS applications on Amazon EC2 using Docker Compose.
* Accelerated static image asset retrieval speeds using Amazon CloudFront distribution caching.
* Centralized runtime log aggregation on Amazon CloudWatch Logs for real-time monitoring and debugging.
* Completed and published two high-quality technical blogs (Blog 2 on WAF Bot Control and Blog 3 on Lambda Cold Start).
* Synced all environment and infrastructure configurations to GitHub.
