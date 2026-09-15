# Incident Response Guide

This guide covers common deployment issues encountered with the `node-express-app` project, their likely causes, and resolution steps. It's based on real issues encountered while building and deploying this application.



## 1. ECS task keeps stopping / restarting in a loop

**Symptoms:** Task status cycles between `PROVISIONING`, `RUNNING`, and `STOPPED` repeatedly

**Possible causes:**
- The container is crashing shortly after startup (application error)
- The health check is failing (misconfigured command, wrong port, or app not ready before the check runs)
- Insufficient CPU/memory causing the task to be killed

**Resolution steps:**
1. Check CloudWatch Logs for the failing task's log stream — this usually shows the actual crash reason
2. Review the health check configuration — confirm the command matches the actual app port and route
3. Increase the health check's "start period" if the app needs more time to boot before checks begin
4. Confirm the task definition's CPU/memory allocation is sufficient for the app


## 2. GitHub Actions workflow fails immediately with no logs / "invalid workflow"

**Symptoms:** Workflow run fails in 0 seconds, no graph or steps shown

**Possible causes:**
- Invalid YAML syntax in the workflow file (most commonly: a duplicated key like `steps:` appearing twice, incorrect indentation, or a stray character)

**Resolution steps:**
1. Open the workflow file and check for duplicate top-level keys within the same job
2. Validate the YAML structure carefully — indentation errors are a common cause
3. If unsure, rewrite the file from a known-good template rather than patching small pieces, to avoid compounding structural errors


## 3. CI/CD deploy job fails with AWS authentication errors

**Symptoms:** Errors says `AWS secret access key must be provided` 

**Possible causes:**
- GitHub repository secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) are missing or notsaved 
- The IAM user associated with the credentials lacks the required permissions (ECR push, ECS deploy, IAM read)

**Resolution steps:**
1. Confirm both secrets exist under repo **Settings → Secrets and variables → Actions**, spelled exactly as referenced in the workflow file
2. Confirm the IAM user has the necessary managed policies attached (e.g. `AmazonEC2ContainerRegistryFullAccess`, `AmazonECS_FullAccess`)
3. Re-run the workflow after correcting secrets — no new commit is required



## 4. Docker/ECR authentication fails locally

**Symptoms:** `docker login` to ECR returns `400 Bad Request` or similar

**Possible causes:**
- Docker daemon isn't running (especially relevant when using Docker Desktop with WSL integration)
- Stale or expired AWS CLI credentials
- Typos in the long ECR registry URL (a very easy mistake given its length)

**Resolution steps:**
1. Confirm Docker is running: `docker ps` should return without error
2. Confirm AWS credentials are valid: `aws sts get-caller-identity`
3. Re-copy the exact ECR URI from `aws ecr describe-repositories` rather than retyping it manually


## 5. Git push fails with DNS or "could not resolve host" errors

**Symptoms:** `fatal: unable to access '...': Could not resolve host: github.com`

**Possible causes:**
- WSL's DNS resolution has reset (common after a WSL restart)

**Resolution steps:**
1. Manually set a DNS nameserver: `sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'`
2. Make it persistent by disabling WSL's auto-generated resolv.conf via `/etc/wsl.conf`
3. Restart WSL (`wsl --shutdown` from PowerShell) and retry
