# StreamingApp — DevOps Project 5: Container Orchestration and Scaling

## Project Overview

StreamingApp is a MERN-based video streaming platform composed of multiple microservices. For DevOps Project 5, the application has been containerized and deployed to Amazon EKS using Kubernetes and Helm.

The project demonstrates:

- Docker containerization of application services
- Docker Hub image publishing
- Amazon ECR image repositories
- Amazon EKS cluster deployment
- Kubernetes Deployments and Services
- MongoDB StatefulSet with persistent storage
- Kubernetes ConfigMap and Secret
- Readiness and liveness probes
- AWS Load Balancer Controller
- Application Load Balancer (ALB) ingress
- Helm-based application deployment
- Horizontal service scaling
- Rolling updates
- Kubernetes self-healing
- Amazon S3 integration for video and thumbnail storage
- Amazon CloudWatch monitoring and centralized logging
- Jenkins CI/CD pipeline configuration

---

## Architecture

```mermaid
flowchart TB
    User[User / Browser]

    ALB[AWS Application Load Balancer]

    Frontend[Frontend<br/>React + Nginx]
    Auth[Auth Service<br/>Port 3001]
    Streaming[Streaming Service<br/>Port 3002]
    Admin[Admin Service<br/>Port 3003]
    Chat[Chat Service<br/>Port 3004]
    Mongo[(MongoDB<br/>StatefulSet + PVC)]
    S3[(Amazon S3<br/>Videos + Thumbnails)]
    CW[Amazon CloudWatch]
    ECR[Amazon ECR]

    User --> ALB

    ALB --> Frontend
    ALB --> Auth
    ALB --> Streaming
    ALB --> Admin
    ALB --> Chat

    Auth --> Mongo
    Streaming --> Mongo
    Admin --> Mongo
    Chat --> Mongo

    Streaming --> S3
    Admin --> S3

    Frontend -.-> Auth
    Frontend -.-> Streaming
    Frontend -.-> Admin
    Frontend -.-> Chat

    EKS[Amazon EKS Cluster]
    EKS --> Frontend
    EKS --> Auth
    EKS --> Streaming
    EKS --> Admin
    EKS --> Chat
    EKS --> Mongo

    EKS --> CW
    ECR --> EKS
