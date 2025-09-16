# Productionizing AWS’s Retail Sample App with GitOps on EKS


*How I transformed AWS’s demo retail microservices application into a production-ready, cost-optimized platform using Terraform, GitHub Actions, ArgoCD, and EKS Auto Mode.*

![Banner](https://github.com/HasanAshab/retail-store-devops/blob/main/docs/images/banner.png?raw=true)



## Background

The [AWS Retail Sample App](https://github.com/aws-containers/retail-store-sample-app) is a microservices-based demo designed to showcase cloud-native workloads. While it demonstrates architecture concepts, it’s not “production-ready” out of the box.

The objective of this project was to:

* Take the AWS Retail Sample App (5 microservices)
* Deploy it on **Amazon EKS**
* Implement **infrastructure as code, CI/CD, and GitOps**
* Ensure the setup was **secure, observable, and cost-efficient**

This case study documents the approach, challenges, and outcomes.

---

## Architecture Overview
![EKS](https://github.com/HasanAshab/retail-store-devops/blob/main/docs/images/EKS.gif?raw=true)

The final platform integrated:

* **Terraform** for infrastructure as code
* **Amazon EKS Auto Mode** to simplify compute management
* **GitHub Actions** for CI/CD with intelligent change detection
* **ArgoCD** for GitOps-driven continuous delivery
* **ECR** as the private container registry
* **Ingress + Cert Manager** for HTTPS traffic
* Optional **Prometheus/Grafana** for observability

High-level workflow:

```
Developer → GitHub → GitHub Actions → ECR → ArgoCD → EKS
```

---

## Implementation

### 1. Infrastructure as Code with Terraform

* Deployed EKS with **Auto Mode**, eliminating the need to manage node groups.
* Added **random suffix strategy** for unique cluster naming.
* Provisioned add-ons (Ingress, Cert Manager, optional monitoring stack).

### 2. CI/CD with GitHub Actions

* Implemented **change detection** to rebuild only modified services.
* Built and pushed Docker images to **private ECR**.
* Updated Helm values with new image tags.

### 3. GitOps with ArgoCD

* Configured **one ArgoCD application per service**, avoiding a single umbrella chart.
* Enabled automatic sync with Git as the source of truth.
* Supported granular rollouts, service-level visibility, and independent rollbacks.

### 4. Security & Cost Controls

* Enforced **private ECR with scanning + encryption**.
* Applied **IAM least-privilege policies** for CI/CD and services.
* Used **single NAT gateway** in dev and spot instance support for cost optimization.

## Key Challenges and Solutions

* **Node Group Complexity** → Solved with **EKS Auto Mode** for hands-free compute scaling.
* **Helm Chart Updates Overwriting Infra Images** → Implemented targeted AWK script to update only app images.
* **GitOps Workflow Discipline** → Enforced branch protections and structured Git flows.

---

## Results

* **Deployment Speed**:

  * Infrastructure: 15–20 mins
  * Application changes: 3–5 mins
  * Single service update: 1–2 mins

* **Cost Efficiency**:

  * Auto Mode prevented over-provisioning
  * Change detection reduced unnecessary builds
  * Shared NAT + optional spot instances lowered non-prod expenses

* **Production Readiness**:

  * HTTPS ingress with auto certificates
  * RBAC/IAM security enforcement
  * Optional monitoring and alerting enabled

## Lessons Learned

1. **EKS Auto Mode** dramatically reduces operational overhead.
2. **Change detection in CI/CD** is a simple but powerful optimization.
3. **GitOps** adds discipline and traceability but requires strict Git hygiene.
4. **Security and observability** should be built in from day one, not after incidents.


## At a Glance
By productionizing the **AWS Retail Sample App**, this project demonstrates how a demo workload can be elevated into a **secure, cost-optimized, and fully automated cloud-native platform**.

The integration of Terraform, GitHub Actions, ArgoCD, and EKS Auto Mode provided:

* Automated infrastructure provisioning
* Intelligent CI/CD pipelines
* GitOps-driven deployments
* Security, monitoring, and cost controls

This approach can be adapted to any microservices-based workload seeking **production readiness with efficient DevOps practices**.

Check out the [source code](https://github.com/HasanAshab/retail-store-devops) at GitHub for more details.

---
## 📬 Contact

If you’d like to connect, collaborate, or discuss DevOps, feel free to reach out:

* **Website**: [hasan-ashab](https://hasan-ashab.vercel.app/)
* **GitHub**: [github.com/HasanAshab](https://github.com/HasanAshab/)
* **LinkedIn**: [linkedin.com/in/hasan-ashab](https://linkedin.com/in/hasan-ashab/)