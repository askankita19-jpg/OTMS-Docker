# OTMS-Docker

**OTMS-Docker** provides a containerized deployment model for the OTMS application using **Docker, Amazon ECR, Jenkins, and automated CI/CD**.

The repository is designed to build, scan, version, publish, deploy, validate, and roll back OTMS container images through an automated Jenkins workflow.

The deployment model separates application containerization from infrastructure and provides a repeatable Docker-based runtime.

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
              +--------------+--------------+
              |              |              |
              v              v              v
             CI         Docker Build      Security
              |              |              |
              |              v              v
              |         Docker Images     Trivy
              |              |              |
              +--------------+--------------+
                             |
                             v
                         Amazon ECR
                             |
                             v
                       Docker Host
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
          Frontend        Backend APIs    Supporting
          Container       Containers     Services
```

---

## Application Components

The containerized application consists of:

| Component        | Technology         | Container Port |
| ---------------- | ------------------ | -------------: |
| Frontend         | React              |           3000 |
| Employee API     | Go                 |           8080 |
| Attendance API   | Python             |           8081 |
| Salary API       | Java / Spring Boot |           8082 |
| Notification API | Python             |           8085 |

Supporting databases:

| Database   | Port |
| ---------- | ---: |
| PostgreSQL | 5432 |
| Redis      | 6379 |
| ScyllaDB   | 9042 |

---

## Container Strategy

Each application component is packaged as an independent container image.

```text
OTMS-Docker
│
├── Frontend
│   └── frontend image
│
├── Employee API
│   └── employee-api image
│
├── Attendance API
│   └── attendance-api image
│
├── Salary API
│   └── salary-api image
│
└── Notification API
    └── notification-api image
```

Each image should be independently versioned and deployable.

---

## Docker Build Principles

Docker images should follow these practices:

* Minimal base images where practical
* Non-root execution where supported
* No credentials embedded in images
* `.dockerignore`
* Explicit dependency versions
* Application health checks
* Reproducible builds
* Versioned image tags
* Immutable image digests
* Vulnerability scanning before publication

---

## CI/CD Workflow

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
Tag Image
   |
   v
Push to Amazon ECR
   |
   v
Deploy
   |
   v
Smoke Tests
```

---

## Image Versioning

Images must not depend on a mutable `latest` tag for controlled deployments.

Example:

```text
frontend:1.0.0
employee-api:1.0.0
attendance-api:1.0.0
salary-api:1.0.0
notification-api:1.0.0
```

A deployment should retain enough information to identify the exact image version or digest running in the environment.

---

## Amazon ECR

Amazon Elastic Container Registry is used as the container image registry.

The workflow is:

```text
Jenkins
   |
   v
Docker Build
   |
   v
Security Scan
   |
   v
Amazon ECR
   |
   v
Docker Host
```

ECR repositories should use appropriate access control and image lifecycle policies.

---

## Docker Deployment

The official deployment path is through Jenkins.

```text
GitHub
   |
   v
Jenkins
   |
   v
Build Images
   |
   v
Scan Images
   |
   v
Push to ECR
   |
   v
Deploy Containers
   |
   v
Validate
```

Docker commands may be used during development and troubleshooting, but the controlled deployment workflow is Jenkins-driven.

---

## Docker Compose

Docker Compose may be provided for local development and integration testing.

Example:

```text
docker-compose.yml
```

can define the application services and their local dependencies.

Compose is intended to simplify development and testing; controlled environment deployments remain Jenkins-driven.

---

## Configuration

Application configuration must be externalized from container images.

Environment-specific configuration should be supplied at deployment time.

Sensitive values must never be baked into:

* Dockerfiles
* Docker images
* Compose files committed to source control
* Application source code

---

## Security

Security is integrated into the container lifecycle.

### Source Security

Gitleaks scans source code for accidentally committed secrets.

### Code Quality

SonarQube provides static analysis.

### Image Security

Trivy scans container images for known vulnerabilities.

### Runtime Security

Containers should:

* Run with the minimum required privileges
* Avoid unnecessary Linux capabilities
* Avoid running as root where practical
* Expose only required ports
* Receive secrets through secure configuration mechanisms

---

## Health Checks

Each application container should provide an appropriate health mechanism.

The deployment pipeline should verify that containers become healthy after deployment.

Conceptually:

```text
Deploy
  |
  v
Container Started
  |
  v
Health Check
  |
  +---- FAIL ----> Deployment Failed
  |
  +---- PASS ----> Smoke Tests
```

---

## Rollback

Container deployments use previously validated image versions for rollback.

Example:

```text
Current
employee-api:1.0.2

       |
       | failure
       v

Rollback

employee-api:1.0.1
```

The rollback should be performed through the Jenkins deployment workflow.

---

## Repository Structure

```text
OTMS-Docker/
│
├── applications/
│   ├── frontend/
│   ├── employee-api/
│   ├── attendance-api/
│   ├── salary-api/
│   └── notification-api/
│
├── docker/
│   ├── frontend/
│   ├── employee-api/
│   ├── attendance-api/
│   ├── salary-api/
│   └── notification-api/
│
├── compose/
│   └── docker-compose.yml
│
├── Jenkinsfile
├── scripts/
│
└── README.md
```

---

## Prerequisites

Typical requirements include:

* Git
* Jenkins
* Docker
* Docker Compose
* AWS CLI
* AWS IAM permissions
* Amazon ECR
* Trivy
* Gitleaks
* SonarQube
* Required application language runtimes for CI

---

## Deployment Validation

A deployment is successful only after:

* Images are successfully built
* Security scans pass
* Images are available in ECR
* Containers start successfully
* Health checks pass
* Required application endpoints respond
* Application connectivity is validated
* Smoke tests pass

---

## Deployment Principles

1. Build once and deploy the validated image.
2. Never store secrets inside images.
3. Use versioned images.
4. Prefer immutable image digests.
5. Scan images before deployment.
6. Deploy through Jenkins.
7. Validate application health after deployment.
8. Keep rollback versions available.
9. Keep infrastructure and configuration reproducible.
10. Avoid unnecessary container privileges.

---

## Project Goal

The goal of this repository is to provide a reliable container-based deployment model for OTMS.

The complete lifecycle is:

```text
CODE
 ↓
CI
 ↓
SECURITY SCANNING
 ↓
DOCKER BUILD
 ↓
IMAGE SCAN
 ↓
ECR
 ↓
DEPLOY
 ↓
HEALTH CHECK
 ↓
SMOKE TEST
 ↓
ROLLBACK / DESTROY
```
