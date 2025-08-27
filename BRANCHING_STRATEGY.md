# Branching Strategy & GitOps Workflow

This document explains the advanced branching strategy and GitOps workflow implemented in this retail store sample application.

## 📋 Table of Contents

- [Branch Architecture](#branch-architecture)
- [Public Application (Main Branch)](#public-application-main-branch)
- [Production (GitOps Branch)](#production-gitops-branch)
- [GitHub Actions Setup](#github-actions-setup)
- [Deployment Workflows](#deployment-workflows)
- [Infrastructure Components](#infrastructure-components)
- [Troubleshooting](#troubleshooting)
- [Development Workflow](#development-workflow)

## 🏗️ Branch Architecture

This repository implements a **dual-branch GitOps strategy** designed for different deployment scenarios with clear separation between simple public deployments and production-ready automated workflows.

```mermaid
graph LR
    A[Developer] --> B[Code Changes]
    B --> C{Target Branch?}
    C -->|Simple Deployment| D[main branch]
    C -->|Production| E[gitops branch]
    
    D --> F[Manual Deployment]
    D --> G[Public ECR Images]
    D --> H[Umbrella Chart]
    
    E --> I[GitHub Actions]
    E --> J[Private ECR Images]
    E --> K[Individual Apps]
    
    I --> L[Build & Push]
    L --> M[Update Helm Charts]
    M --> N[ArgoCD Sync]
```

## 🏭 Production (GitOps Branch)

### **Purpose**
Full production workflow with automated CI/CD pipeline and private container registry integration.

### **Characteristics**
```yaml
✅ Branch: gitops
✅ Images: Private ECR (auto-updated)
✅ Deployment: Automated via GitHub Actions
✅ ArgoCD: Individual service applications
✅ Workflows: Full CI/CD pipeline (.github/workflows/)
✅ Updates: Automatic on code changes
✅ Target: Production environments, enterprise deployments
```

### **Image Configuration**
```yaml
# Services use private ECR (updated by workflow)
ui: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/retail-store-ui:{COMMIT_HASH}
catalog: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/retail-store-catalog:{COMMIT_HASH}
cart: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/retail-store-cart:{COMMIT_HASH}
checkout: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/retail-store-checkout:{COMMIT_HASH}
orders: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/retail-store-orders:{COMMIT_HASH}

# Infrastructure components (same as main - preserved by workflow)
mysql: public.ecr.aws/docker/library/mysql:8.0
redis: public.ecr.aws/docker/library/redis:6.0-alpine
postgresql: public.ecr.aws/docker/library/postgres:13
rabbitmq: public.ecr.aws/docker/library/rabbitmq:3.8-management
dynamodb-local: public.ecr.aws/aws-dynamodb-local/aws-dynamodb-local:1.25.1
```

### **ArgoCD Configuration**
```yaml
# Uses individual applications for granular control
- retail-store-ui
- retail-store-catalog  
- retail-store-cart
- retail-store-checkout
- retail-store-orders

# Each points to gitops branch
spec:
  source:
    targetRevision: gitops
```

## 🔧 GitHub Actions Setup

### **Required Secrets**
Configure these secrets in your GitHub repository settings:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key for ECR/EKS access | `AKIA...` |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Key | `wJalrXUt...` |
| `AWS_REGION` | AWS Region for resources | `us-west-2` |
| `AWS_ACCOUNT_ID` | AWS Account ID for ECR URLs | `123456789012` |

### **IAM Permissions Required**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:PutImage",
        "ecr:CreateRepository",
        "ecr:DescribeRepositories"
      ],
      "Resource": "*"
    }
  ]
}
```

## 🔄 Deployment Workflows

### **Production Workflow**
```bash
1. Developer commits to gitops branch (src/ directory)
2. GitHub Actions detects changes
3. Builds only changed services
4. Pushes images to private ECR
5. Updates Helm chart values with new image tags
6. Commits changes back to gitops branch
7. ArgoCD syncs individual applications
8. Only changed services are redeployed
```

### **Change Detection Logic**
```yaml
# Only builds services with actual code changes
Changed files in src/ui/ → Build ui service
Changed files in src/catalog/ → Build catalog service
Changed files in src/cart/ → Build cart service
Changed files in src/checkout/ → Build checkout service
Changed files in src/orders/ → Build orders service

# Manual trigger builds all services
workflow_dispatch → Build all services
```

## 🏗️ Infrastructure Components

### **Service Images vs Infrastructure Images**

#### **Service Images (Updated by Workflow)**
- **ui, catalog, cart, checkout, orders** - Application services
- **Source**: Private ECR repositories
- **Updates**: Automated via GitHub Actions
- **Versioning**: Git commit hash (7 characters)

#### **Infrastructure Images (Preserved by Workflow)**
- **mysql, redis, postgresql, rabbitmq, dynamodb-local** - Database/messaging
- **Source**: Public ECR/Docker Hub
- **Updates**: Manual only (stable versions)
- **Versioning**: Semantic versioning

### **Workflow Protection Logic**
```bash
# AWK script ensures only main service image is updated
/^image:/ { in_main_image = 1 }  # Target first image: section only
in_main_image && /repository:/ && !updated_repo { 
  # Update only if we haven't updated repository yet
}
/^[a-zA-Z]/ && !/^image:/ { in_main_image = 0 }  # Exit image section
```

## 🔧 Troubleshooting

### **SharedResourceWarning in ArgoCD**
```yaml
# Problem: Same resources deployed by multiple applications
Error: ClusterIssuer/letsencrypt-prod is part of applications argocd/retail-store-app and retail-store-ui

# Solution: Use only one deployment method per branch
Public Application branch: Use umbrella chart only
Production branch: Use individual applications only
```

### **Image Pull Errors**
```yaml
# Problem: Wrong ECR repository or missing images
Error: Failed to pull image "123456789012.dkr.ecr.us-west-2.amazonaws.com/retail-store-ui:abc1234"

# Solutions:
1. Check ECR repository exists (workflow creates automatically)
2. Verify AWS credentials have ECR permissions
3. Ensure image was built and pushed successfully
4. Check GitHub Actions logs for build failures
```

### **Workflow Not Triggering**
```yaml
# Problem: GitHub Actions not running on commits
# Solutions:
1. Ensure changes are in src/ directory
2. Check branch is 'gitops'
3. Verify GitHub Actions is enabled
4. Check workflow file syntax (.github/workflows/deploy.yml)
```

### **Infrastructure Images Being Overwritten**
```yaml
# Problem: MySQL/Redis images pointing to private ECR
# Solution: Workflow should preserve infrastructure images
# Check: AWK script in deploy.yml targets only main service image
```

## 👨‍💻 Development Workflow

### **For Changes (GitOps Branch)**
```bash
1. git checkout gitops
2. Make changes to application code in src/ directory
3. git commit && git push origin gitops
4. GitHub Actions automatically:
   - Builds changed services
   - Updates Helm charts
   - Commits changes back
5. ArgoCD automatically syncs changes
```

## 🎯 Best Practices

### **When to Use This Branching Strategy**
- ✅ Production deployments
- ✅ Enterprise environments
- ✅ Automated testing pipelines
- ✅ Continuous deployment workflows

### **Security Considerations**
- 🔒 Use IAM roles with minimal permissions
- 🔒 Rotate AWS access keys regularly
- 🔒 Enable ECR image scanning
- 🔒 Use branch protection rules
- 🔒 Review automated commits

---

This branching strategy provides a robust foundation for both stable production deployments and rapid development cycles while maintaining clear separation of concerns and automated quality gates.
