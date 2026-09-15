# Task 2 Reflection — CI/CD

## My Deployment Process

For Continuous Integration, I built a GitHub Actions workflow that runs automatically on every push to `main`. It installs dependencies, runs a syntax check on the app, and then starts the app and hits it with a `curl` request as a basic automated quality check, confirming the app actually boots and responds before considering the build successful.

For Continuous Deployment, I chose AWS, specifically ECS with Fargate, since I already had a working Docker image from Task 1. I pushed that image to Amazon ECR, then set up an ECS cluster, a task execution role with permissions to pull from ECR and write logs, a task definition describing the container and its resource needs, and finally a service that runs the task with a public IP and an open security group on port 3002.

## Challenges Encountered

- **Docker Desktop instability on WSL**: Docker Desktop kept failing to authenticate with ECR (400 Bad Request errors) and was generally unreliable. I switched to running Docker Engine natively inside WSL/Ubuntu instead, which resolved the issue and turned out to be a cleaner setup overall.
- **IAM permissions**: My AWS IAM user initially lacked permissions for ECR and IAM role creation, resulting in AccessDenied errors. I resolved this by attaching the necessary managed policies (ECR full access, ECS full access, IAM full access) directly to my user.
- **CLI syntax precision**: Several commands failed due to small typos (missing hyphens, missing spaces between flags, incorrect domain names like `amazon.aws` instead of `amazonaws.com`). Given how unforgiving CLI syntax is, I eventually switched to the AWS Console for the ECS setup steps, which was more reliable since it uses forms and dropdowns instead of typed commands.
- **CI workflow issues**: My initial GitHub Actions workflow had a duplicated `steps:` key (invalid YAML), outdated action version tags, a typo in a curl URL, and a hanging background process — each of which I diagnosed by reading the Actions logs step by step and fixed incrementally.

## How CI/CD Improves Software Delivery

Setting this up made the value of CI/CD concrete rather than theoretical. CI catches build and runtime issues automatically on every push, rather than relying on manually remembering to test before deploying. CD then takes a validated build and gets it live without manual server setup each time. Together, they shorten the feedback loop between writing code and knowing whether it actually works in a production-like environment, and they reduce the risk of human error in repetitive deployment steps.