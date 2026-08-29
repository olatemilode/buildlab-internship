# Node Express App — Dockerized

A simple Node.js/Express web application, containerized with Docker as part of the BuildLabs DevOps Internship (Task 1: Version Control & Containerization).

Original application source: [denisecase/node-express-app](https://github.com/denisecase/node-express-app)

## Project Setup

1. Clone this repository:
   
   ```bash
   git clone git clone https://github.com/denisecase/node-express-app.git buildlab-internship
   cd buildlab-internship
   ```
1. Install dependencies (for local, non-Docker use):
   
   ```bash
   npm install
   ```

## Installation Steps

### Run locally (without Docker)

```bash
npm install
node app.js
```

The app will be available at `http://localhost:3002`.

### Run with Docker

**1. Build the Docker image:**

```bash
docker build -t node-express-app .
```

**2. Run the container:**

```bash
docker run -d -p 3002:3002 --name node-express-app node-express-app
```

**3. Verify it’s running:**

```bash
docker ps
```

**4. Access the app:**
Open your browser to `http://localhost:3002`

Additional routes to try:

- `http://localhost:3002/hello`
- `http://localhost:3002/big`
- `http://localhost:3002/greeting/42`
- `http://localhost:3002/yo/YourName`
- `http://localhost:3002/fortune`

### Environment Variables

The app’s port can be overridden at runtime using the `PORT` environment variable:

```bash
docker run -d -p 4000:4000 -e PORT=4000 --name node-express-app node-express-app
```


## Project Structure

```
├── app.js
├── config
├── Dockerfile
├── LICENSE
├── package.json
└── README.md
