# Task 3 Reflection — Infrastructure Monitoring & Automation

## What I Learned About DevOps

This task made clear that DevOps isn't just about writing a Dockerfile or clicking "deploy" once, it's about building a system that stays reliable and observable over time. Setting up CloudWatch logging and a health check taught me that a working deployment isn't the finish line; a deployment that can tell you when something's wrong, and recover from it automatically, is a plus.
Working through repeated YAML syntax errors, IAM permission gaps, DNS issues in WSL, Docker authentication problems reinforced that most of DevOps work in practice is debugging, reading logs carefully, isolating one variable at a time, and not assuming the first error message is the real root cause.

## How Automation Improves Software Delivery

Before this task, my deployment process was entirely manual: build the image, push it. Adding the GitHub Actions deploy job changed that completely, now a push to `main` doesn't alone results in a new image but  built, pushed to ECR, and deployed to ECS automatically, with the pipeline waiting to confirm the new deployment is stable before finishing.

The health check added a tea, even if a bad deploy did get through, ECS can detect an unhealthy container and cycle it out on its own, rather than requiring someone to notice the outage first.

## Improvements I Would Make With More Time

- **Add a load balancer** so the app can has a stable DNS name instead of an IP that changes every time the task restarts this would also unlock proper HTTP-level health checks and possibly HTTPS.
- **Add Container Insights** for real CPU/memory dashboards on Fargate, rather than relying on logs alone for resource visibility.
- **Add automated rollback** — if the new deployment fails its health check, automatically redeploy the previous known-good task definition instead of leaving the service degraded.
- **Add a staging environment** so changes could be tested in a separate ECS service before touching the production-facing one.
- **Tighten IAM permissions** — several times I had to grant broad `FullAccess` policies to get past permission errors quickly
