# Deployment Documentation — Task 2 (CI/CD)

## Overview

This document describes how the `node-express-app` is continuously integrated and deployed using GitHub Actions (CI) and AWS ECS with Fargate (CD).

**Live application:** http://13.51.79.15:3002

## Continuous Integration (GitHub Actions)

The CI workflow is defined in `.github/workflows/ci.yml` and runs automatically on every push or pull request to the `main` branch.

The workflow:

1. Checks out the repository code
1. Sets up Node.js 18
1. Installs dependencies with `npm install`
1. Runs a build/syntax check using `node --check app.js`
1. Starts the app and runs a smoke test (`curl` against `http://localhost:3002`) to confirm it boots and responds correctly

## Continuous Deployment (AWS ECS + Fargate)

### Architecture

- **Container registry:** Amazon ECR (`026296209205.dkr.ecr.eu-north-1.amazonaws.com/node-express-app`)
- **Compute:** AWS ECS running on Fargate (serverless containers, no EC2 instances to manage)
- **Region:** eu-north-1

### Deployment steps taken

1. **Built and pushed the Docker image to ECR:**
   
   ```bash
   aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 026296209205.dkr.ecr.eu-north-1.amazonaws.com
   docker tag node-express-app:latest 026296209205.dkr.ecr.eu-north-1.amazonaws.com/node-express-app:latest
   docker push 026296209205.dkr.ecr.eu-north-1.amazonaws.com/node-express-app:latest
   ```
1. **Created an ECS cluster** (`node-express-cluster`) to host the service.
1. **Created an IAM task execution role** (`ecsTaskExecutionRole`) with the `AmazonECSTaskExecutionRolePolicy` attached, allowing ECS to pull images from ECR and write logs to CloudWatch.
1. **Registered a task definition** (`node-express-task`) specifying:
- Fargate launch type
- 0.5 vCPU / 1 GB memory
- Container image: the ECR URI above
- Container port: 3002
1. **Created an ECS service** (`node-express-service`) in the cluster:
- Desired task count: 1
- Public IP enabled
- A security group allowing inbound TCP traffic on port 3002 from anywhere (0.0.0.0/0)
1. **Retrieved the running task’s public IP** and confirmed the app is reachable at `http://13.51.79.15:3002`.

### Environment Configuration

- The application listens on port 3002 by default, matching the container port mapping configured in the task definition.
- No sensitive secrets were required for this application; if they were, they would be managed via AWS Secrets Manager or ECS task definition secrets rather than hardcoded.

## Notes

- No load balancer was used for this deployment, so the public IP is tied to the running task and will change if the task restarts. A future improvement would be to add an Application Load Balancer for a stable DNS endpoint.