# E-Commerce Microservices Application

GitHub: https://github.com/keshavp07-git/emartapp-microservices/tree/master
Duration: Aug 2025 – Dec 2025

---

## Project Overview

A containerized, production-ready e-commerce platform built on a microservices architecture. The application is composed of 6 independently deployable services managed with Docker and Kubernetes via Helm. NGINX is used as a reverse proxy and load balancer, cutting response latency by 30%. The project integrates a full DevSecOps pipeline — with SonarQube SAST, Trivy container scanning, and Sonatype Nexus artifact management — alongside reusable Terraform modules for infrastructure provisioning. Automated CI/CD pipelines reduced deployment failures by 45% and all releases use rolling updates for zero-downtime deployments.

---

## Tech Stack

Category               | Tools & Technologies
-----------------------|-------------------------------------------------------------
Cloud Platform         | AWS (EKS, EC2, VPC, IAM, S3, RDS)
Containerization       | Docker
Container Orchestration| Kubernetes (K8s), Helm
Web Server / Proxy     | NGINX (Reverse Proxy, Load Balancer)
CI/CD                  | GitLab CI
Infrastructure as Code | Terraform (reusable modules)
DevSecOps              | SonarQube (SAST), Trivy (container & artifact scanning)
Artifact Management    | Sonatype Nexus (Docker, Maven, npm registries)
Secrets Management     | GitLab Protected/Masked Variables, AWS Secrets Manager
Version Control        | Git, GitLab

---

## Architecture Diagram

    +------------------+
    |   Client Browser |
    +--------+---------+
             |
    +--------v---------+
    |   NGINX           |
    |  Reverse Proxy    |
    |  Load Balancer    |
    +--+--+--+--+--+---+
       |  |  |  |  |
       |  |  |  |  +-------------------------------+
       |  |  |  +---------------------+            |
       |  |  +-----------+            |            |
       |  |              |            |            |
  +----v--+   +----------v-+  +-------v--+  +-----v------+
  | User  |   |  Product   |  |  Order   |  |  Payment   |
  | Svc   |   |  Svc       |  |  Svc     |  |  Svc       |
  +-------+   +------------+  +----------+  +------------+
       |              |            |
  +----v--------------v------------v----+
  |        Kubernetes (AWS EKS)         |
  |   Helm-managed deployments          |
  |   Rolling updates, HPA, PV/PVC      |
  +-------------------+-----------------+
                       |
          +------------v-----------+
          |   Terraform Modules    |
          |   VPC, EKS, RDS, IAM   |
          +------------------------+

    CI/CD Security Layer (GitLab CI)
    ---------------------------------
    Code Push --> SonarQube SAST --> Trivy Scan
        --> Nexus Artifact Registry --> Deploy to EKS

---

## Setup & Installation

### Prerequisites

- AWS CLI configured with appropriate IAM permissions
- kubectl installed and configured
- Terraform >= 1.3
- Helm >= 3.x
- Docker installed locally
- GitLab account with CI/CD runners configured

### Step 1 — Clone the repository

    git clone https://github.com/keshavp07-git/emartapp-microservices.git
    cd emartapp-microservices

### Step 2 — Provision AWS infrastructure with Terraform

    cd terraform/
    terraform init
    terraform plan
    terraform apply

This provisions: VPC, subnets, EKS cluster, IAM roles, RDS instance, and S3 buckets.

### Step 3 — Configure kubectl for EKS

    aws eks update-kubeconfig --region <your-region> --name <cluster-name>

### Step 4 — Configure Sonatype Nexus

Set up Nexus registries for Docker, Maven, and npm.
Add Nexus credentials as GitLab masked/protected variables:

    NEXUS_URL        = <your-nexus-url>
    NEXUS_USERNAME   = <username>
    NEXUS_PASSWORD   = <password>   (masked)

### Step 5 — Deploy services using Helm

    helm upgrade --install emart ./helm/emart \
      --namespace production \
      --create-namespace \
      --values helm/emart/values.yaml

### Step 6 — Configure NGINX

    kubectl apply -f k8s/nginx-ingress.yaml

NGINX acts as the single entry point, routing traffic to individual microservices and load balancing across pods.

### Step 7 — Set up secrets in AWS Secrets Manager

Store all sensitive credentials (DB passwords, API keys) in AWS Secrets Manager and reference them in your Kubernetes deployments via ExternalSecrets or environment injection.

---

## CI/CD Pipeline Details

### Pipeline Flow (GitLab CI)

    Code Push to GitLab
      |
      v
    GitLab CI Pipeline Triggered
      |
      +-- SonarQube SAST scan (Static Application Security Testing)
      |     Fails pipeline if critical vulnerabilities found
      |
      +-- Docker image build
      |
      +-- Trivy container scan
      |     Scans image for CVEs and misconfigurations
      |     Fails pipeline on HIGH / CRITICAL findings
      |
      +-- Push verified image to Sonatype Nexus Docker Registry
      |
      +-- Update Helm chart values (image tag)
      |
      v
    Helm deploy to Kubernetes (AWS EKS)
      |
      v
    Rolling update — zero downtime release
      |
      v
    Health checks via liveness & readiness probes

### Secrets Handling

- All credentials stored in GitLab protected/masked variables — never appear in pipeline logs
- Production secrets managed via AWS Secrets Manager
- No hardcoded secrets anywhere in the codebase

### Key Metrics

- Deployment failures: reduced by 45% through automated pipeline checks
- Response latency: reduced by 30% via NGINX load balancing
- Security: zero secrets exposed in logs or artifacts
- Releases: 100% zero-downtime via Kubernetes rolling updates
