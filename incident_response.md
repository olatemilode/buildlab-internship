# Incident Response Guide

This guide covers deployment issues encountered with the `node-express-app` project, their causes, and resolution steps. 

## 1. ECS task keeps stopping / restarting in a loop

**Symptoms:** Task status cycles between `PROVISIONING`, `RUNNING`, and `STOPPED` repeatedly

**Possible causes:**
- The container is crashing shortly after startup (application error)
- The health check is failing (app not ready before the check runs)
- Insufficient CPU/memory causing the task to be killed

**Resolution steps:**
1. Check CloudWatch Logs for the failing task's log stream 
2. Review the health check configuration 
3. Increase the health check's "start period" if the app needs more time to boot before checks begin
4. Confirm the task definition's CPU/memory allocation is sufficient for the app


## 2. GitHub Actions workflow fails immediately with no logs / "invalid workflow"

**Symptoms:** Workflow run fails in 0 seconds, no graph or steps shown

**Possible causes:**
- Invalid YAML syntax in the workflow file
- 
**Resolution steps:**
1. Open the workflow file and check for duplicate top-level keys within the same job
2. Validate the YAML structure carefully

## 3. CI/CD deploy job fails with AWS authentication errors

**Symptoms:** Errors says `AWS secret access key must be provided` 

**Possible causes:**
- GitHub repository secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) are missing or notsaved 
- The IAM user associated with the credentials lacks the required permissions (ECR push, ECS deploy, IAM read)

**Resolution steps:**
1. Confirm both secrets exist under repo **Settings → Secrets and variables → Actions**, spelled exactly as referenced in the workflow file
2. Confirm the IAM user has the necessary managed policies attached (e.g. `AmazonECS_FullAccess`)
3. Re-run the workflow after correcting secrets 


## 4. Docker/ECR authentication fails locally

**Symptoms:** `docker login` to ECR returns `400 Bad Request` or similar

**Possible causes:**
- Docker daemon isn't running 
- Stale or expired AWS CLI credentials
- Typos in the long ECR registry URL 

**Resolution steps:**
1. Confirm Docker is running: `docker ps` should return without error
2. Confirm AWS credentials are valid: `aws sts get-caller-identity`
3. Re-copy the exact ECR URI from `aws ecr describe-repositories` rather than retyping it manually


## 5. Git push fails with DNS or "could not resolve host" errors

**Symptoms:** `fatal: unable to access '...': Could not resolve host: github.com`

**Possible causes:**
- WSL's DNS resolution has reset
- Internet connection failure

**Resolution steps:**
1. Check the internet connection and reconnect
2. Restart WSL and retry
