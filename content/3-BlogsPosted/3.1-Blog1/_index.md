---
title: "AWS Fargate vs Lambda"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AWS FARGATE VS LAMBDA – WHEN TO CHOOSE CONTAINER SERVERLESS OVER FUNCTION SERVERLESS

In the process of exploring serverless compute options on AWS, I noticed that many newcomers often default to "serverless = Lambda". However, in reality, AWS has another serverless option at the container level: AWS Fargate. While both services share the "no server management" philosophy, they differ in execution units, resource limits, and suitable use cases.

### Core Differences

* **Lambda**: The execution unit is a function — a short block of code, running in a fully managed AWS environment, triggered by events (event-driven).
* **Fargate**: The execution unit is a container — running on ECS or EKS, allowing you to package the entire application (including dependencies, custom runtime) without having to manage the underlying EC2 instances or clusters.

In other words, Lambda is suitable for small, short-lived tasks, while Fargate is ideal for workloads that need to run as a proper "service", with a longer lifecycle and more flexible resource requirements.

### Important Technical Limits of Lambda

* The maximum execution duration for each invocation is 15 minutes (900 seconds) — this is a hard limit and cannot be adjusted.
* Memory can be configured from 128 MB to 10,240 MB, and CPU is allocated proportionally to the chosen memory level.
* For tasks that need to run longer than 15 minutes while still maintaining a serverless architecture, a common solution is to split the logic into multiple functions and use Step Functions or Lambda Destinations to orchestrate the steps — instead of trying to stretch a single function.
* Notably, AWS recently introduced Lambda durable functions, which allow a logical process to pause and resume across multiple invocations, lasting up to a year. This is suitable for workflows that require approval or multi-day processing — though each individual invocation is still limited to 15 minutes.

### Fargate Has No Such Limits

Since it runs containers instead of short-lived functions, Fargate is not subject to the 15-minute limit — containers can run continuously (long-running service), making it suitable for web servers, traditional containerized API backends, or jobs that run for hours. Fargate also allows much more flexible CPU/RAM resource configuration compared to Lambda's 10GB ceiling, making it ideal for workloads requiring high and sustained computing power.

### When to Choose Lambda?

* **Discrete event processing**: file uploads, DynamoDB data changes, SQS messages.
* **Lightweight APIs**: short execution times (seconds to minutes).
* **Highly irregular traffic**: traffic that can drop to zero — Lambda only charges when invocations occur, preventing waste on idle resources.
* **Teams wanting minimal infrastructure management**: accepting constraints on runtimes and execution times.

### When to Choose Fargate?

* **Pre-containerized applications**: existing applications packaged into containers (Docker) that you want to reuse without rewriting into the function model.
* **Workloads running continuously or longer than 15 minutes**: heavy batch processing, workers listening to a queue over an extended period.
* **Need for detailed control over the runtime environment**: multiple concurrent processes within the same container, custom OS-level dependencies.
* **Stable traffic**: large enough that Fargate's provisioned resource pricing model is more cost-effective than Lambda's per-invocation pricing.

### Best Practices for Decision Making

* Do not view this as a binary, system-wide choice — many real-world architectures combine both: Lambda for lightweight event-driven tasks, and Fargate for core long-running services.
* If a Lambda function frequently needs to be split due to the 15-minute limit, that is a clear indicator to consider migrating that part of the workload to Fargate rather than trying to bypass the limit.
* Analyze actual costs based on traffic patterns: sporadic/irregular traffic is usually cheaper with Lambda; stable, continuous traffic is usually cheaper with Fargate.

### Key Takeaways

Through the comparison between Lambda and Fargate, I have gained a better understanding of:
* The distinction between "serverless" at the function level and the container level — both are serverless but address different sets of problems.
* The execution time limit as a critical decision factor when choosing between the two services.
* In actual enterprise architectures, combining multiple compute services according to the characteristics of each workload is more important than finding a single "best" service.

### Conclusion

AWS Lambda and Fargate do not directly compete, but rather complement each other in the AWS serverless landscape. Lambda is perfect for short, event-driven tasks, while Fargate is built for long-running or containerized services. Choosing the right service for the right workload optimizes both cost and operational complexity.

### References

* AWS Lambda FAQs: [https://aws.amazon.com/lambda/faqs/](https://aws.amazon.com/lambda/faqs/)
* AWS Lambda Developer Guide – Configuring function timeout: [https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html](https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html)
* Blog post link: [Facebook](https://web.facebook.com/groups/awsstudygroupfcj/posts/2234471933984433/?_rdc=1&_rdr#)