---
title: "Week 7 Worklog"
date: 2026-08-06
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Write the participation report for the AWS FCAJ Agent Forge - Deepdive event.
* Configure custom domain `myapps.io.vn` pointing to the EC2 Elastic IP, set up Nginx Reverse Proxy, and configure SSL/HTTPS security certificates (Let's Encrypt / Certbot).
* Conduct comprehensive End-to-End (E2E) testing of the entire FoodieRecipe system on the live HTTPS production domain.
* Monitor system performance and debug container logs via Amazon CloudWatch Logs to identify exceptions and optimize code.
* Complete recording and editing the demo video showcasing website interfaces and features.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Write the participation report for the AWS FCAJ Agent Forge - Deepdive event. <br>- Configure DNS record (A record) pointing `myapps.io.vn` to the EC2 Elastic IP. <br>- Configure Nginx as a Reverse Proxy to route port 80/443 traffic to Next.js and NestJS backend. | 03/08/2026 | 03/08/2026 | [Nginx Configuration Guide](https://nginx.org/en/docs/) <br><br> [Event 2 Report](../../4-eventparticipated/4.2-event2/) |
| 3 | - Install Certbot and request a free SSL certificate from Let's Encrypt. <br>- Configure automatic SSL renewal, enable HTTPS, and set secure cookies for authorization. | 04/08/2026 | 04/08/2026 | [Certbot Let's Encrypt Guide](https://certbot.eff.org/) |
| 4 | - Execute comprehensive End-to-End (E2E) system integration testing: User login, S3 upload, Rekognition ingredient extraction, Bedrock Nova Lite recipe creation, and CloudWatch log captures. | 05/08/2026 | 05/08/2026 | [AWS End-to-End Testing Guides](https://aws.amazon.com/) |
| 5 | - Evaluate web application performance on the production domain, inspect and debug exceptions recorded in container logs on CloudWatch, and refactor backend APIs. | 06/08/2026 | 06/08/2026 | [Amazon CloudWatch Logs Guide](https://docs.aws.amazon.com/AmazonWatch/latest/logs/) |
| 6 | - Record the detailed website demo video showcasing interfaces and features on the production domain. | 07/08/2026 | 07/08/2026 | [AWS Presentation Best Practices](https://aws.amazon.com/) |

### Week 7 Achievements:
* Completed the participation report for the AWS FCAJ Agent Forge - Deepdive event.
* Successfully configured custom domain `myapps.io.vn` with secure HTTPS connections (Let's Encrypt SSL).
* Verified stable and optimized application operations across the AWS EC2 multi-container environment on the official domain.
* Centralized monitoring and debugging via Amazon CloudWatch Logs to ensure system reliability.
* Completed recording and editing the demo video showcasing website interfaces and features on the production environment.
