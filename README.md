# 🚀 Enterprise CI/CD DevOps Pipeline
> **Automated Code Quality, Security Audit, and Containerized Deployment Lifecycle.**

![Build Status](https://img.shields.io/badge/Build-Success-brightgreen?style=for-the-badge&logo=jenkins)
![SonarQube](https://img.shields.io/badge/Quality_Gate-Passed-blue?style=for-the-badge&logo=sonarqube)
![Docker](https://img.shields.io/badge/Docker-Pushed-blue?style=for-the-badge&logo=docker)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

---

## 📖 Overview
This repository hosts a high-fidelity **DevOps CI/CD pipeline** architected for rapid, reliable, and secure software delivery. It automates the entire software development lifecycle (SDLC)—from the moment code is pushed to GitHub until it is deployed as a resilient Docker container.

### 🛠 Tech Stack
*   **Automation:** [Jenkins](https://www.jenkins.io/) (Declarative Pipeline)
*   **Static Analysis:** [SonarQube](https://www.sonarqube.org/) (Project Audit & Quality Gates)
*   **Containerization:** [Docker](https://www.docker.com/) (Immutable Image Creation)
*   **Registry:** [DockerHub](https://hub.docker.com/) (Centralized Image Distribution)
*   **Runtime:** Node.js (Application Environment)

---

## 🏗 Pipeline Architecture
The pipeline follows a deterministic workflow to ensure that only 100% validated code reaches production:

1.  **Checkout**: Synchronizes source code from the repository.
2.  **Build & Test**: Installs dependencies and executes the unit test suite.
3.  **Quality Audit**: Performs deep static analysis via SonarQube to detect bugs, vulnerabilities, and code smells.
4.  **Quality Gate**: Enforces a strict "Stop-Build" policy if security thresholds are not met.
5.  **Containerization**: Packages the validated app into a versioned Docker image.
6.  **Registry Sync**: Pushes the image to DockerHub for global availability.
7.  **Auto-Deploy**: Automatically updates the local production container on port 3000.

---

## 🛡 Security & Best Practices
*   **RBAC Implementation**: Jenkins is secured using Role-Based Access Control to prevent unauthorized configuration changes.
*   **Credential Masking**: All sensitive tokens (Sonar, DockerHub) are securely managed and masked in logs.
*   **Synchronous Gating**: Utilizing `-Dsonar.qualitygate.wait=true` ensures the pipeline never proceeds with compromised code.

---

## 🚀 Deployment Guide

### Prerequisites
*   Docker & Docker Compose installed.
*   Jenkins server with SonarQube Scanner plugin.
*   SonarQube server operational.

### Local Execution
To build and run the application manually:
```bash
# Clone the repository
git clone https://github.com/guru22cseb37/-week12-assignment.git

# Build the Docker image
docker build -t guru784/cicd-pipeline-app:latest .

# Run the container
docker run -d -p 3000:3000 --name my-app guru784/cicd-pipeline-app:latest
```

---

## 🧠 Technical Victories
During development, we conquered several high-level integration hurdles:
*   **Token Integrity**: Resolved environment-level token corruption by implementing direct string-injection in the shell context.
*   **Network Deadlock Elimination**: Bypassed asynchronous webhook requirements by switching to synchronous scanner polling.
*   **Registry Synchronization**: Unified naming conventions across local builds and global registries for zero-friction distribution.


