<div align="center">

<br />

# 💼 Pulse HRMS — Human Resource Management System

### *End-to-End DevSecOps · CI/CD · Cloud-Native Kubernetes Deployment*

<br/>

[![GitHub Stars](https://img.shields.io/github/stars/syedmehfooz47/hrms-devops?style=for-the-badge&logo=github&color=FF6B35)](https://github.com/syedmehfooz47/hrms-devops)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![CI Pipeline](https://img.shields.io/badge/CI-Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-K8s-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![SonarQube](https://img.shields.io/badge/Quality-SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)](https://www.sonarqube.org/)
[![Trivy](https://img.shields.io/badge/Security-Trivy-1904DA?style=for-the-badge&logo=aqua&logoColor=white)](https://aquasecurity.github.io/trivy/)

<br/>

**Pulse HRMS** is a production-grade PERN stack application built to demonstrate real-world DevSecOps, full CI/CD automation, container orchestration, security scanning, and cloud-native Kubernetes deployment strategies.

</div>

---

## 📋 Table of Contents

> [!Important]
> Navigate through the infrastructure setup, CI/CD pipeline configuration, and Kubernetes orchestration steps.

| Component | Jump To |
|---|---|
| 🏗️ Architecture | [DevOps Architecture](#architecture) |
| 🚀 CI/CD Pipeline | [Jenkins CI/CD Workflows](#cicd) |
| ☁️ Kubernetes Setup | [K8s Manifests & Deployment](#k8s) |
| 🛡️ Security | [SonarQube & Trivy Integration](#security) |
| 🐳 Docker | [Containerization](#docker) |

---

## 🎯 DevOps & Cloud Overview

While Pulse HRMS is a fully functional AI-powered HR platform (React, Node.js, Express, PostgreSQL), this repository focuses strictly on the **DevSecOps** and **Cloud Infrastructure** lifecycle:

- **Continuous Integration (CI)** — Automated builds, tests, and Docker image creation via Jenkins.
- **Continuous Delivery (CD) & GitOps** — Automated manifest updates triggering Kubernetes deployments.
- **DevSecOps** — Filesystem scanning with Trivy and static code analysis with SonarQube quality gates.
- **Container Orchestration** — Multi-tier microservices architecture on Kubernetes with Nginx reverse proxy.
- **High Availability** — Liveness/Readiness probes and `initContainers` for strict startup sequencing.
- **Alerting** — Automated Jenkins email notifications for pipeline status monitoring.

---

## 🚀 DevOps Architecture & Flow

<div align="center">

*Full end-to-end DevSecOps & GitOps deployment pipeline*

</div>

1. **Developer Commits Code** to the GitHub repository.
2. **Jenkins CI Pipeline** triggers automatically.
   - Cleans workspace and checks out code.
   - Runs Trivy filesystem scan for vulnerabilities.
   - Executes SonarQube code quality analysis.
   - Builds multi-stage Docker images for both Frontend and Backend.
   - Pushes built images to Docker Hub.
3. **Jenkins CD Pipeline (GitOps)** is triggered downstream.
   - Pulls the Kubernetes manifests.
   - Updates `backend.yml` and `frontend.yml` with the newly built Docker image tags.
   - Commits and pushes the updated manifests back to GitHub (`[skip ci]`).
4. **Kubernetes Cluster** synchronizes the updated state and rolls out the new deployments.

---

## 🛠️ DevOps Tech Stack

| Technology | Purpose |
|---|---|
| ![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white) | Source Code & Version Control |
| ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) | Application Containerization |
| ![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white) | Master CI/CD Orchestration |
| ![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=flat-square&logo=sonarqube&logoColor=white) | Code Quality & Quality Gates |
| ![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=flat-square&logo=aqua&logoColor=white) | Vulnerability & Filesystem Scanning |
| ![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white) | Container Orchestration |
| ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white) | Stateful Database Workload |

---

## ⚙️ Jenkins CI/CD Configuration

<h3 id="cicd">Pipeline Architecture</h3>

The project utilizes a dual-pipeline architecture for separation of concerns between integration and deployment.

#### 1. Continuous Integration (`Jenkinsfile`)
- **Agent**: Runs on any available Jenkins worker.
- **Stages**:
  - `Clean Workspace` & `Checkout Code`
  - `Check Skip CI`: Prevents recursive loops from GitOps commits.
  - `Trivy File System Scan`: Early security vulnerability detection.
  - `SonarQube Analysis` & `Quality Gate`: Enforces code standards.
  - `Docker: Build Images`: Compiles React app and Express server into optimized images.
  - `Docker: Push to DockerHub`: Stores artifacts in a central registry.
- **Post Actions**: Generates reports, triggers CD pipeline, and dispatches dynamic HTML email alerts.

#### 2. GitOps Continuous Delivery (`GitOps/Jenkinsfile`)
- **Stages**:
  - `Code Checkout`: Fetches the latest repository state.
  - `Update: Kubernetes manifests`: Utilizes `sed` to dynamically inject the new `$DOCKER_TAG` into `kubernetes/frontend.yml` and `kubernetes/backend.yml`.
  - `Git: Code update and push`: Authenticates with GitHub and commits the updated infrastructure as code back to the `main` branch.

---

## ☁️ Kubernetes Orchestration

<h3 id="k8s">Cluster Design & Manifests</h3>

The infrastructure is defined entirely as code inside the `kubernetes/` directory, implementing a robust microservices layout.

- **Frontend Deployment (`frontend.yml`)**:
  - Dual-container Pod: React application and an Nginx reverse proxy.
  - `initContainers`: Runs a `wget` loop to ensure the backend service is fully responsive before the frontend starts.
  - Exposed externally via a `NodePort` service.
- **Backend Deployment (`backend.yml`)**:
  - Express.js API server handling business logic and AI integration.
  - `initContainers`: Validates PostgreSQL readiness via `pg_isready` before initiating DB connections.
- **Stateful Workloads**:
  - `postgres.yml` along with `persistentVolume.yaml` and `persistentVolumeClaim.yaml` to ensure database persistence and data durability across pod restarts.

### Resiliency & Health Checks
All deployments implement strict Kubernetes health checks to ensure zero-downtime rollouts and self-healing:
```yaml
livenessProbe:
  httpGet:
    path: /
    port: 3000
  initialDelaySeconds: 10
readinessProbe: ...
```

---

## 🛡️ DevSecOps Integration

<h3 id="security">Security & Code Quality Analysis</h3>

Security is shifted left into the Jenkins pipeline:

- **Aqua Trivy**: Scans the raw filesystem and dependencies before the Docker build phase, catching known CVEs early in the lifecycle.
- **SonarQube**: Node.js and React codebases are analyzed for bugs, vulnerabilities, and code smells. The pipeline blocks deployment if the defined Quality Gate fails.

---

## 🐳 Docker Containerization

<h3 id="docker">Multi-Stage Builds & Local Compose</h3>

- **Frontend Image**: Built using a multi-stage `Dockerfile` (`node:20-alpine`) to keep the final image footprint minimal.
- **Local Dev Environments**: A comprehensive `docker-compose.yml` is provided for local debugging, orchestrating the frontend, backend, and PostgreSQL database within an isolated bridged network (`hrms-network`).

---

<div align="center">

### 🙏 Acknowledgements

Built with a focus on DevSecOps and cloud-native best practices.

Built with ❤️ by [Syed Mehfooz C S](https://github.com/syedmehfooz47)

[![GitHub](https://img.shields.io/badge/GitHub-syedmehfooz47-181717?style=for-the-badge&logo=github)](https://github.com/syedmehfooz47)

</div>
