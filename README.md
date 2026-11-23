# aws-eks-modular-terraform-platform
<div align="center">

# ☁️ AWS EKS Modular Terraform Platform

**Production-Grade, Multi-Environment Infrastructure-as-Code (IaC) with GitOps PR Previews & FinOps Optimization**

[![Terraform](https://img.shields.io/badge/Terraform-v1.17+-623CE4?style=for-the-badge&logo=terraform&logoColor=white)](https://developer.hashicorp.com/terraform)
[![AWS](https://img.shields.io/badge/AWS-EKS%20%7C%20VPC%20%7C%20S3-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.36-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-OIDC%20CI%2FCD-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Security](https://img.shields.io/badge/Security-tfsec%20Scanned-00C853?style=for-the-badge&logo=checkmarx&logoColor=white)](https://github.com/aquasecurity/tfsec)

</div>

---

## 📖 Overview

This repository implements an enterprise-ready, modular **Infrastructure-as-Code (IaC)** platform that provisions isolated multi-AZ AWS VPC networks and managed **Amazon EKS** clusters across `dev` and `prod` environments.

Designed with **DRY software engineering principles**, the architecture separates reusable infrastructure modules from environment-specific configurations, integrates **FinOps cost controls**, enforces **zero-trust keyless authentication** via GitHub OIDC, and automates **Pull Request security checks and `terraform plan` previews**.

---

## 🏗️ System Architecture

```mermaid
flowchart LR
    subgraph GitHub ["GitHub Repository & CI/CD"]
        PR["Pull Request"] --> GHA["GitHub Actions Workflow"]
        GHA -->|"1. Lint & Security"| SEC["terraform fmt & tfsec"]
        GHA -->|"2. Keyless Auth"| OIDC["AWS IAM OIDC Federation"]
        GHA -->|"3. Diff Output"| COMMENT["Automated PR Plan Comment"]
    end

    subgraph Backend ["Remote State & Locking"]
        S3[("S3 State Bucket\n(AES-256 + Versioning)")]
        DDB[("DynamoDB Table\n(State LockID)")]
    end

    subgraph AWS ["AWS Cloud (ap-south-1 / Mumbai)"]
        subgraph VPC ["Multi-AZ VPC (10.0.0.0/16)"]
            PUB["Public Subnets\n(NAT Gateways + ELB Tags)"]
            PRIV["Private Subnets\n(Isolated Worker Nodes)"]
            EKS["Amazon EKS Control Plane (v1.36)"]
            NODES["Managed Node Groups\n(Spot in Dev / On-Demand in Prod)"]
        end
    end

    OIDC --> Backend
    OIDC --> AWS
    PUB --> PRIV
    PRIV --> EKS --> NODES
