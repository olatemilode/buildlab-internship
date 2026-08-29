##My containerization approach:

I chose an open-source Node.js/Express application with no existing Docker setup, so I could design the containerization from scratch. I based the image on node:18-alpine to keep it lightweight, and structured the Dockerfile to copy package.json and install dependencies before copying the rest of the source code,this lets Docker cache the install layer so rebuilds are faster when only app code changes. The app’s port (3002) is exposed and mapped at runtime, and I used an environment variable (PORT) to make the port configurable without editing the Dockerfile.


##Challenges encountered:

I ran into a DNS resolution failure during the build. WSL couldn’t resolve registry-1.docker.io, which I fixed by manually setting a DNS nameserver and disabling WSL’s auto-generated resolv.conf. I also had a container that appeared to run successfully but didn’t show up in docker ps, it had actually exited right after starting, and docker ps only shows running containers by default. Running in the foreground (without -d) surfaced the real error, and removing the stale container cleared a naming conflict before I could re-run it successfully.


##How Docker improves application deployment:

Docker made it clear how much the app runs the same way regardless of the host machine, since the image bundles the runtime, dependencies, and code together. It removes “works on my machine” issues, makes environments reproducible from a single Dockerfile, and simplifies onboarding and future CI/CD work since anyone can rebuild the exact same setup.