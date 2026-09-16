# Infrastructure Documentation

This document describes the  infrastructure supporting the `node-express-app` deployment, including AWS resources, configuration, and how they relate to one another.

## Overview

The application runs as a containerized Node.js/Express service on AWS, built and deployed through an automated CI/CD pipeline. See buildlab.io for a visual overview of how these pieces connect.

**Live application:** http://13.51.79.15:3002 *public IP — changes if the task restarts*

## AWS Resources

### Region
All resources are provisioned in **eu-north-1** (Europe, Stockholm).

### Amazon ECR (Elastic Container Registry)
- **Repository:** `node-express-app`
- **URI:** `026296209205.dkr.ecr.eu-north-1.amazonaws.com/node-express-app`
- **Purpose:** Stores the Docker images built from this repository. Both a `latest` tag and a commit-SHA-tagged image are pushed on every deploy, so specific past versions remain available for rollback if needed.

### Amazon ECS (Elastic Container Service)
- **Cluster:** `node-express-app`
- **Launch type:** Fargate 
- **Task definition family:** `node-express-task`
  - Container: `node-express-app`
  - Port mapping: 3002 (TCP)
  - CPU/Memory: 0.5 vCPU / 1 GB
  - Logging: CloudWatch Logs, log group `/ecs/node-express-task`
  - Health check: `curl -f http://localhost:3002/ || exit 1`, every 30s, 3 retries, 10s start period
- **Service:** `node-express-task-service-l77u36ai`
  - Desired task count: 1
  - Deployment controller: ECS (rolling update)
  - Networking: default VPC, public IP enabled
  - Health check grace period: 30 seconds

### Security Group
- Inbound: TCP port 3002 from `0.0.0.0/0` (allows public access to the app)
- All other inbound traffic blocked by default

### IAM
- **Role:** `ecsTaskExecutionRole`
  - Attached policy: `AmazonECSTaskExecutionRolePolicy`
  - Purpose: allows ECS to pull the container image from ECR and write logs to CloudWatch on the task's behalf
- **CI/CD credentials:** a dedicated IAM user's access key and secret are stored as GitHub Actions secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`), scoped with ECR, ECS, and IAM read permissions needed for the deploy pipeline

### CloudWatch
- **Log group:** `/ecs/node-express-task` — captures container stdout/stderr, used to confirm the app boots correctly and to debug crashes
- **Metrics:** CPU and memory utilization graphs available under the service's Health and metrics tab

## CI/CD Pipeline

Defined in `.github/workflows/ci.yml`, triggered on every push to `main`:

1. **Build job** — installs dependencies, runs a syntax check, and smoke-tests the app locally in the CI runner
2. **Deploy job** (runs only after build succeeds, only on `main`) — authenticates to AWS, builds and pushes a new Docker image to ECR, fetches the current ECS task definition, swaps in the new image, registers it as a new revision, and updates the ECS service to use it

## Known Limitations

- No load balancer: the app's public IP changes whenever the ECS task restarts. A stable DNS name would require attaching an Application Load Balancer.
- No staging environment: all deploys go directly to the single production-like service.
- No autoscaling configured: desired task count is fixed at 1.
- IAM permissions for the CI/CD user are broader than strictly necessary (full ECR/ECS access) rather than least-privilege scoped, due to time constraints during setup.
