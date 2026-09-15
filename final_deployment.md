# Final Deployment Documentation

## Project Summary

This document describes the complete deployment journey of `node-express-app`, a simple Node.js/Express application, from local containerization through to a fully automated CI/CD pipeline running on AWS. It consolidates the work from all three project tasks.

**Live application:** http://13.51.79.15:3002
**Repository:** https://github.com/olatemilode/buildlab-internship


## 1. Application

**Base project:** [denisecase/node-express-app](https://github.com/denisecase/node-express-app) — a minimal, open-source Node.js/Express application with no existing Docker or deployment setup, chosen specifically to give full ownership over every containerization and deployment decision.

The app listens on port 3002 and exposes a handful of demo routes (`/hello`, `/big`, `/fortune`, `/greeting/:name`, etc.).

## 2. Version Control & Containerization (Task 1)

- Forked the base application into a personal GitHub repository, with commit history reflecting incremental, meaningful changes.
- Wrote a `Dockerfile` based on `node:18-alpine`:
  - Copies `package.json` and installs dependencies before copying the rest of the source, so Docker can cache the install layer between builds
  - Exposes port 3002, configurable via the `PORT` environment variable
  - Runs the app with `CMD ["node", "app.js"]`
- Built and ran the image locally, verifying the app responded correctly on `localhost:3002` before moving on.
- Documented setup, build, and run instructions in the project README.

## 3. CI/CD Pipeline (Task 2)

### Continuous Integration
A GitHub Actions workflow (`.github/workflows/ci.yml`) runs on every push or pull request to `main`:
1. Checks out the code
2. Installs dependencies
3. Runs a syntax/build check (`node --check app.js`)
4. Starts the app and smoke-tests it with `curl` to confirm it boots and responds correctly

### Continuous Deployment
The same workflow file extends into a `deploy` job (runs only on pushes to `main`, only after the build job passes):
1. Authenticates to AWS using credentials stored as GitHub Actions secrets
2. Logs in to Amazon ECR
3. Builds a fresh Docker image and pushes it to ECR, tagged both `latest` and with the Git commit SHA
4. Fetches the current ECS task definition, swaps in the new image, and registers it as a new revision
5. Updates the ECS service to roll out the new revision, waiting for the deployment to stabilize before completing

### Cloud Platform
- **Amazon ECR** stores the built Docker images
- **AWS ECS on Fargate** runs the container as a managed, serverless task — no EC2 instances to patch or maintain
- **IAM role (`ecsTaskExecutionRole`)** grants ECS permission to pull images and write logs on the task's behalf


## 4. Monitoring & Automation (Task 3)

### Monitoring
- **Logs:** CloudWatch Logs (`/ecs/node-express-task`) capture the application's stdout, confirming successful startup and surfacing any runtime errors
- **Resource usage:** CPU and memory utilization are visible via the ECS service's Health and metrics tab
- **Availability:** ECS's Tasks tab shows real-time task status (running/stopped) and health

### Automation improvements implemented
1. **Automatic deployment after successful builds** — the CI/CD pipeline above deploys automatically on every successful push to `main`, with no manual intervention
2. **Health checks and restart policy** — the task definition includes a container health check (`curl -f http://localhost:3002/ || exit 1`, every 30 seconds, 3 retries), and ECS's service scheduler automatically replaces any task that fails its health check or stops unexpectedly, maintaining the desired count of 1 running task at all times

### Incident response
A dedicated incident response guide (`INCIDENT_RESPONSE.md`) documents common issues encountered during this project, along with their causes and resolution steps.


## 6. Environment Configuration

- **Environment variables:** `PORT` controls which port the app listens on inside the container; ECS's port mapping is set to 3002 to match
- **Secrets:** AWS credentials for the CI/CD pipeline are stored as encrypted GitHub Actions repository secrets, never committed to source control
- No application-level secrets (database credentials, API keys) are required for this project, as the app has no external dependencies


## 7. Known Limitations & Future Improvements

- No load balancer — the public IP changes whenever the ECS task restarts; adding an Application Load Balancer would provide a stable DNS endpoint
- No staging environment, all changes deploy directly to the single running service
- No autoscaling, desired task count is fixed at 1 regardless of load
 automated rollback on failed health checks beyond ECS's default task replacement behavior; a more advanced setup could automatically redeploy the last known-good task definition
