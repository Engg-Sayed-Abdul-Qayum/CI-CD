# CI-CD: Jenkins + Docker + Nginx Demo

A small, hands-on project that demonstrates a **Continuous Integration / Continuous Deployment (CI/CD)** workflow using **Jenkins** and **Docker**. A static website is packaged into an **Nginx** container image by a CI pipeline, then deployed as a running container by a separate CD pipeline.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Setting Up the Jenkins Pipelines](#setting-up-the-jenkins-pipelines)
- [Verifying the Deployment](#verifying-the-deployment)
- [Running Locally Without Jenkins](#running-locally-without-jenkins)
- [Troubleshooting](#troubleshooting)
- [Possible Improvements](#possible-improvements)
- [Author](#author)

## Overview

| Item | Detail |
|------|--------|
| Application | Static HTML page (`index.html`) |
| Web server | Nginx (official Docker image) |
| Containerization | Docker |
| Automation | Jenkins declarative pipelines |
| Image name / tag | `my-image:latest` |
| Container name | `ci-cd-demo` |
| Exposed port | Host `8082` → Container `80` |

## Architecture

```
 GitHub repo ──► Jenkins CI pipeline ──► Docker image (my-image:latest)
                                                  │
                                                  ▼
                                  Jenkins CD pipeline ──► Container "ci-cd-demo"
                                                          http://<host>:8082
```

1. **CI** checks out the code and builds the Docker image.
2. **CD** stops and removes any previous container, then runs a fresh one from the newly built image.

## Repository Structure

```
CI-CD/
├── dockerfile            # Builds an Nginx image serving index.html
├── index.html            # The static website
└── Jenkins/
    ├── CI/
    │   └── Jenkinsfile   # Pipeline: checkout + docker build
    └── CD/
        └── Jenkinsfile   # Pipeline: replace container + docker run
```

## Prerequisites

- A Linux machine or VM (for example, an AWS EC2 instance or a local VM)
- **Jenkins** installed and running
- **Docker** installed, with the `jenkins` user added to the `docker` group so pipelines can run Docker commands:
  ```bash
  sudo usermod -aG docker jenkins
  sudo systemctl restart jenkins
  ```
- **Git** installed on the Jenkins agent
- Port **8082** open on the host firewall or security group

## How It Works

### Dockerfile

```dockerfile
FROM nginx:latest
WORKDIR /usr/share/nginx/html
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Uses the official Nginx image, copies `index.html` into Nginx's web root, exposes port 80, and runs Nginx in the foreground so the container stays alive.

### CI Pipeline (`Jenkins/CI/Jenkinsfile`)

| Stage | What it does |
|-------|--------------|
| **Git Checkout** | Clones the `main` branch into the workspace, or skips the clone if a `.git` directory already exists |
| **Docker Image Build** | Runs `docker build -t my-image:latest .` |

### CD Pipeline (`Jenkins/CD/Jenkinsfile`)

| Stage | What it does |
|-------|--------------|
| **Stop Existing Container** | If a container named `ci-cd-demo` exists, stops and removes it; otherwise prints a message and continues |
| **Start Container on Port 8082** | Runs `docker run -d --name ci-cd-demo -p 8082:80 my-image:latest` |

## Getting Started

```bash
git clone https://github.com/Engg-Sayed-Abdul-Qayum/CI-CD.git
cd CI-CD
```

## Setting Up the Jenkins Pipelines

Create two pipeline jobs in Jenkins:

1. **CI job**
   - New Item → **Pipeline** → name it, for example, `ci-pipeline`
   - Under *Pipeline*, choose **Pipeline script from SCM**
   - SCM: Git, Repository URL: `https://github.com/Engg-Sayed-Abdul-Qayum/CI-CD.git`
   - Branch: `*/main`
   - Script Path: `Jenkins/CI/Jenkinsfile`

2. **CD job**
   - Same steps, named for example `cd-pipeline`
   - Script Path: `Jenkins/CD/Jenkinsfile`

Optionally, configure the CI job to trigger the CD job after a successful build (via *Post-build Actions* or a GitHub webhook) for a fully automated flow.

**Run order:** run the CI job first so the image exists, then run the CD job.

## Verifying the Deployment

After the CD job succeeds:

```bash
docker ps --filter "name=ci-cd-demo"
curl http://localhost:8082
```

Or open `http://<server-ip>:8082` in a browser. You should see:

> **Hello from Docker!**
> This website is running inside an Nginx Docker container.

## Running Locally Without Jenkins

```bash
docker build -t my-image:latest .
docker run -d --name ci-cd-demo -p 8082:80 my-image:latest
```

Stop and clean up:

```bash
docker stop ci-cd-demo && docker rm ci-cd-demo
```

## Troubleshooting

| Problem | Likely cause / fix |
|---------|--------------------|
| `permission denied` on `/var/run/docker.sock` | Add the `jenkins` user to the `docker` group and restart Jenkins |
| `port is already allocated` | Another process uses 8082; stop it or change the port mapping in the CD Jenkinsfile |
| CD job fails with `Unable to find image 'my-image:latest'` | Run the CI job first (and on the same Docker host) |
| Page not reachable from browser | Open port 8082 in the firewall or cloud security group |

## Possible Improvements

- Pin the base image (for example `nginx:1.27-alpine`) instead of `latest` for reproducible, smaller builds
- Tag images with the build number or commit SHA (`my-image:${BUILD_NUMBER}`) to enable rollbacks
- Push images to a registry (Docker Hub, ECR) and have CD pull from it
- Add a smoke-test stage (for example `curl` the container) after deployment
- Add a GitHub webhook so each push triggers CI automatically, and chain CD after CI
- Add `post` blocks for notifications and cleanup (`docker image prune`)
- Add a `.dockerignore` and rename `dockerfile` to `Dockerfile` (the conventional name)

## Author

**Sayed Abdul Qayum**
- GitHub: [@Engg-Sayed-Abdul-Qayum](https://github.com/Engg-Sayed-Abdul-Qayum)
- LinkedIn: [@Sayed-Abdul-Qayum](https://# CI-CD: Jenkins + Docker + Nginx Demo

A small, hands-on project that demonstrates a **Continuous Integration / Continuous Deployment (CI/CD)** workflow using **Jenkins** and **Docker**. A static website is packaged into an **Nginx** container image by a CI pipeline, then deployed as a running container by a separate CD pipeline.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Setting Up the Jenkins Pipelines](#setting-up-the-jenkins-pipelines)
- [Verifying the Deployment](#verifying-the-deployment)
- [Running Locally Without Jenkins](#running-locally-without-jenkins)
- [Troubleshooting](#troubleshooting)
- [Possible Improvements](#possible-improvements)
- [Author](#author)

## Overview

| Item | Detail |
|------|--------|
| Application | Static HTML page (`index.html`) |
| Web server | Nginx (official Docker image) |
| Containerization | Docker |
| Automation | Jenkins declarative pipelines |
| Image name / tag | `my-image:latest` |
| Container name | `ci-cd-demo` |
| Exposed port | Host `8082` → Container `80` |

## Architecture

```
 GitHub repo ──► Jenkins CI pipeline ──► Docker image (my-image:latest)
                                                  │
                                                  ▼
                                  Jenkins CD pipeline ──► Container "ci-cd-demo"
                                                          http://<host>:8082
```

1. **CI** checks out the code and builds the Docker image.
2. **CD** stops and removes any previous container, then runs a fresh one from the newly built image.

## Repository Structure

```
CI-CD/
├── dockerfile            # Builds an Nginx image serving index.html
├── index.html            # The static website
└── Jenkins/
    ├── CI/
    │   └── Jenkinsfile   # Pipeline: checkout + docker build
    └── CD/
        └── Jenkinsfile   # Pipeline: replace container + docker run
```

## Prerequisites

- A Linux machine or VM (for example, an AWS EC2 instance or a local VM)
- **Jenkins** installed and running
- **Docker** installed, with the `jenkins` user added to the `docker` group so pipelines can run Docker commands:
  ```bash
  sudo usermod -aG docker jenkins
  sudo systemctl restart jenkins
  ```
- **Git** installed on the Jenkins agent
- Port **8082** open on the host firewall or security group

## How It Works

### Dockerfile

```dockerfile
FROM nginx:latest
WORKDIR /usr/share/nginx/html
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Uses the official Nginx image, copies `index.html` into Nginx's web root, exposes port 80, and runs Nginx in the foreground so the container stays alive.

### CI Pipeline (`Jenkins/CI/Jenkinsfile`)

| Stage | What it does |
|-------|--------------|
| **Git Checkout** | Clones the `main` branch into the workspace, or skips the clone if a `.git` directory already exists |
| **Docker Image Build** | Runs `docker build -t my-image:latest .` |

### CD Pipeline (`Jenkins/CD/Jenkinsfile`)

| Stage | What it does |
|-------|--------------|
| **Stop Existing Container** | If a container named `ci-cd-demo` exists, stops and removes it; otherwise prints a message and continues |
| **Start Container on Port 8082** | Runs `docker run -d --name ci-cd-demo -p 8082:80 my-image:latest` |

## Getting Started

```bash
git clone https://github.com/Engg-Sayed-Abdul-Qayum/CI-CD.git
cd CI-CD
```

## Setting Up the Jenkins Pipelines

Create two pipeline jobs in Jenkins:

1. **CI job**
   - New Item → **Pipeline** → name it, for example, `ci-pipeline`
   - Under *Pipeline*, choose **Pipeline script from SCM**
   - SCM: Git, Repository URL: `https://github.com/Engg-Sayed-Abdul-Qayum/CI-CD.git`
   - Branch: `*/main`
   - Script Path: `Jenkins/CI/Jenkinsfile`

2. **CD job**
   - Same steps, named for example `cd-pipeline`
   - Script Path: `Jenkins/CD/Jenkinsfile`

Optionally, configure the CI job to trigger the CD job after a successful build (via *Post-build Actions* or a GitHub webhook) for a fully automated flow.

**Run order:** run the CI job first so the image exists, then run the CD job.

## Verifying the Deployment

After the CD job succeeds:

```bash
docker ps --filter "name=ci-cd-demo"
curl http://localhost:8082
```

Or open `http://<server-ip>:8082` in a browser. You should see:

> **Hello from Docker!**
> This website is running inside an Nginx Docker container.

## Running Locally Without Jenkins

```bash
docker build -t my-image:latest .
docker run -d --name ci-cd-demo -p 8082:80 my-image:latest
```

Stop and clean up:

```bash
docker stop ci-cd-demo && docker rm ci-cd-demo
```

## Troubleshooting

| Problem | Likely cause / fix |
|---------|--------------------|
| `permission denied` on `/var/run/docker.sock` | Add the `jenkins` user to the `docker` group and restart Jenkins |
| `port is already allocated` | Another process uses 8082; stop it or change the port mapping in the CD Jenkinsfile |
| CD job fails with `Unable to find image 'my-image:latest'` | Run the CI job first (and on the same Docker host) |
| Page not reachable from browser | Open port 8082 in the firewall or cloud security group |

## Possible Improvements

- Pin the base image (for example `nginx:1.27-alpine`) instead of `latest` for reproducible, smaller builds
- Tag images with the build number or commit SHA (`my-image:${BUILD_NUMBER}`) to enable rollbacks
- Push images to a registry (Docker Hub, ECR) and have CD pull from it
- Add a smoke-test stage (for example `curl` the container) after deployment
- Add a GitHub webhook so each push triggers CI automatically, and chain CD after CI
- Add `post` blocks for notifications and cleanup (`docker image prune`)
- Add a `.dockerignore` and rename `dockerfile` to `Dockerfile` (the conventional name)

## Author

**Engg. Sayed Abdul Qayum**
- GitHub: [@Engg-Sayed-Abdul-Qayum](https://github.com/Engg-Sayed-Abdul-Qayum)
- LinkedIn: [@Sayed-Abdul-Qayum](https://www.linkedin.com/in/sayed-abdul-qayum/?isSelfProfile=true))
