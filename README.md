# OTMS-Docker

**OTMS-Docker** is the containerized deployment version of the OTMS application.

It takes the same five OTMS application services and packages them as Docker images, with **Jenkins as the official CI/CD deployment mechanism**.

This repository represents the second deployment generation of the OTMS project.

---

## Purpose

The purpose of this repository is to demonstrate the evolution from traditional VM-based deployment to containerized application deployment.

```text
OTMS
 |
 | EC2 / AMI
 |
 v
OTMS-Docker
 |
 | Docker containers
 |
 v
OTMS-EKS
 |
 | Kubernetes
 |
 v
OTMS-Monitoring
```

---

## Application Components

| Component        | Technology         | Port |
| ---------------- | ------------------ | ---: |
| Frontend         | React              | 3000 |
| Employee API     | Go                 | 8080 |
| Attendance API   | Python             | 8081 |
| Salary API       | Java / Spring Boot | 8082 |
| Notification API | Python             | 8085 |

Databases:

| Database   | Port |
| ---------- | ---: |
| PostgreSQL | 5432 |
| Redis      | 6379 |
| ScyllaDB   | 9042 |

---

## Architecture

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    +--> Source Checks
    |
    +--> Tests
    |
    +--> SonarQube
    |
    +--> Gitleaks
    |
    +--> Trivy
    |
    v
Docker Build
    |
    v
Docker Images
    |
    v
Amazon ECR
    |
    v
Docker Host
    |
    +--> Frontend
    +--> Employee API
    +--> Attendance API
    +--> Salary API
    +--> Notification API
```

---

## Docker Images

Each application service is independently containerized.

```text
otms/frontend
otms/employee-api
otms/attendance-api
otms/salary-api
otms/notification-api
```

Images are versioned rather than relying on `latest`.

Example:

```text
employee-api:1.0.0
employee-api:1.0.1
employee-api:1.0.2
```

Where appropriate, immutable image digests are used for deployment.

---

## CI/CD Pipeline

```text
Checkout
   |
   v
Gitleaks
   |
   v
Formatting / Validation
   |
   v
Unit Tests
   |
   v
SonarQube
   |
   v
Docker Build
   |
   v
Trivy Image Scan
   |
   v
Push to ECR
   |
   v
Deploy
   |
   v
Smoke Tests
```

---

## Security

The container pipeline includes:

* Gitleaks
* SonarQube
* Trivy
* Dependency scanning
* Docker image vulnerability scanning
* Secret separation from images
* Non-root containers where practical
* Minimal base images

Secrets must never be hardcoded inside:

* Dockerfiles
* source code
* Compose files
* Jenkinsfiles
* Kubernetes manifests

---

## Docker Compose

A Docker Compose configuration may be provided for local development and testing.

Example:

```text
docker compose
     |
     +--> frontend
     +--> employee-api
     +--> attendance-api
     +--> salary-api
     +--> notification-api
```

However:

> **Docker Compose is not the official deployment mechanism for this project.**

The official deployment path is Jenkins.

---

## Deployment Principle

This repository is independently deployable.

```text
Jenkins
   |
   v
Build
   |
   v
Scan
   |
   v
ECR
   |
   v
Docker Host
```

It does not require the traditional `OTMS` EC2 deployment to be running.

It also does not require `OTMS-EKS`.

---

## Rollback

Rollback is performed by deploying the previous known-good image version or immutable image digest.

```text
Current
1.0.2
  |
  | failure
  v
Previous
1.0.1
```

The rollback operation is controlled through Jenkins.

---

## Configuration

Application configuration is provided at runtime rather than baked into Docker images.

Configuration may include:

* Database endpoints
* Redis endpoint
* ScyllaDB endpoint
* Application environment
* API configuration
* Secrets

Sensitive values are managed externally.

---

## Project Goals

This repository demonstrates:

* Docker containerization
* Multi-service container deployment
* Docker image versioning
* Amazon ECR
* Jenkins-driven container CI/CD
* Container security scanning
* Immutable deployment artifacts
* Rollback
* Runtime configuration management

---

## Relationship with OTMS

`OTMS-Docker` uses the same OTMS application concept but represents a separate deployment architecture.

```text
OTMS
Traditional VM
    |
    v
OTMS-Docker
Containers
```

The two deployments are independent.

---

## Related Repositories

* **OTMS** — Traditional EC2/AMI deployment
* **OTMS-Docker** — Docker deployment
* **OTMS-EKS** — Kubernetes/EKS deployment
* **OTMS-Monitoring** — Monitoring and orchestration

---

## Project Status

🚧 **Under Development**

---

## Author

**Ankita**

DevOps / Cloud Engineering Project
