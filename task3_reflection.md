# Task 3 Reflection — Infrastructure Monitoring & Automation

## What I Learned About DevOps

This task made clear that DevOps isn't just about writing a Dockerfile or clicking "deploy" once — it's about building a system that stays reliable and observable over time. Setting up CloudWatch logging and a health check taught me that a working deployment isn't the finish line; a deployment that can tell you when something's wrong, and recover from it automatically, is a different level of maturity. I also learned that ECS's built-in scheduler already handles a lot of "automation" implicitly (restarting failed tasks to maintain a desired count) — part of the skill is knowing what your platform already gives you versus what you need to configure explicitly.

Working through repeated failures — YAML syntax errors, IAM permission gaps, DNS issues in WSL, Docker authentication problems — also reinforced that most of DevOps work in practice is methodical debugging: reading logs carefully, isolating one variable at a time, and not assuming the first error message is the real root cause.

## How Automation Improves Software Delivery

Before this task, my deployment process was entirely manual: build the image, push it, and manually update the ECS service every time. Adding the GitHub Actions deploy job changed that completely — now a push to `main` alone results in a new image, built, pushed to ECR, and deployed to ECS automatically, with the pipeline waiting to confirm the new deployment is stable before finishing.

This matters beyond convenience. Manual deployment steps are exactly where mistakes creep in — forgetting to rebuild, pushing the wrong tag, or updating the wrong service. Automating that path removes an entire category of human error, and it means deployments happen consistently every time, not just when I remember every step correctly.

The health check adds a second layer: even if a bad deploy did get through, ECS can detect an unhealthy container and cycle it out on its own, rather than requiring someone to notice the outage first.

## Improvements I Would Make With More Time

- **Add a load balancer** so the app has a stable DNS name instead of an IP that changes every time the task restarts — this would also unlock proper HTTP-level health checks and possibly HTTPS.
- **Add Container Insights** for real CPU/memory dashboards on Fargate, rather than relying on logs alone for resource visibility.
- **Add automated rollback** — if the new deployment fails its health check, automatically redeploy the previous known-good task definition instead of leaving the service degraded.
- **Add a staging environment** so changes could be tested in a separate ECS service before touching the production-facing one.
- **Tighten IAM permissions** — several times I had to grant broad `FullAccess` policies to get past permission errors quickly; with more time I'd scope these down to only the specific actions needed.
- **Add automated backups** — this app has no persistent data, but if it did, a scheduled backup step (e.g. to S3) would be a natural next automation improvement to add.
