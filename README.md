# Google Microservices DevSecOps CD

This repository handles the Continuous Deployment (CD) pipeline for the Google Microservices DevSecOps project, a cloud-native e-commerce application built with microservices architecture. It focuses on deploying the application to AWS using GitOps principles with ArgoCD and Helm charts.

## Overview

The project implements a two-repository setup:
- **CI Repository**: `google-microservices-DevSecOps` - Contains source code, Dockerfiles, and Jenkins pipelines for building and testing microservices.
- **CD Repository**: `google-microservices-DevSecOps-CD` - Contains Kubernetes manifests, Helm charts, and ArgoCD configurations for deployment.

## Architecture

The application consists of 10 microservices (excluding loadgenerator) written in multiple languages:

| Service | Language | Description |
|---------|----------|-------------|
| frontend | Go | Web frontend serving the e-commerce site |
| cartservice | C# | Manages shopping cart using Redis |
| productcatalogservice | Go | Provides product catalog and search |
| currencyservice | Node.js | Handles currency conversion |
| paymentservice | Node.js | Processes payments (mock) |
| shippingservice | Go | Calculates shipping costs |
| emailservice | Python | Sends order confirmations (mock) |
| checkoutservice | Go | Orchestrates order processing |
| recommendationservice | Python | Provides product recommendations |
| adservice | Java | Serves contextual ads |

### Deployment Architecture

- **Platform**: AWS EKS (ap-south-1 region) with Terraform/OpenTofu infrastructure provisioning (in development)
- **GitOps Tool**: ArgoCD with declarative application manifests
- **Package Manager**: Helm with templating and values management
- **CI/CD**: Jenkins with shared libraries and GitOps pipelines
- **Container Registry**: ECR (planned migration from Docker Hub for enhanced security)
- **Monitoring**: Kube-prometheus-stack (Prometheus + Grafana) with custom dashboards
- **Alerting**: Prometheus Alertmanager with Slack/email notifications

## Workflow

1. **Development**: Code changes pushed to CI repo
2. **CI Pipeline**: Jenkins builds Docker images, runs tests, pushes to registry
3. **CD Pipeline**: Jenkins updates Helm values, commits to CD repo
4. **GitOps Deployment**: ArgoCD detects changes, deploys to EKS
5. **Monitoring**: Application health monitored via Kubernetes

## Key Components

### ArgoCD Configuration
- `argocd/main.yml`: Application manifest for ArgoCD
- `argocd/online-app.yml`: Project configuration with permissions

### Helm Chart
- `kubernetes/`: Helm chart for deploying all microservices
- `kubernetes/templates/`: Kubernetes manifests for each service
- `kubernetes/values.yaml`: Default configuration values

### Jenkins CD Pipeline
- **GitOps Pipeline**: `GitOps/Jenkinsfile` orchestrates CD process using declarative Jenkins syntax
- **YAML Processing**: Utilizes `yq` for efficient Helm values.yaml updates during image tag promotion
- **Shared Libraries**: Leverages custom Jenkins shared library for reusable CD functions
- **Automated Commits**: Updates Kubernetes manifests and triggers ArgoCD synchronization
- **Notification Integration**: Slack notifications for deployment status and failures

## Skills Demonstrated

- **Microservices Architecture**: Designed and implemented 10+ services with gRPC communication and service mesh principles
- **Containerization**: Created optimized, multi-stage Dockerfiles for multi-language stack with security best practices
- **Kubernetes**: Managed deployments, services, ConfigMaps, Secrets, and RBAC on EKS with principle of least privilege
- **GitOps**: Implemented ArgoCD with declarative application manifests and automated synchronization
- **CI/CD**: Built comprehensive Jenkins pipelines using shared libraries, declarative syntax, and yq for YAML processing
- **Helm**: Packaged applications with templating, values management, and chart reusability
- **AWS**: Deployed and managed EKS clusters with proper networking, security groups, and IAM roles
- **DevSecOps**: Integrated security scanning, compliance checks, and vulnerability management
- **Infrastructure as Code**: Terraform/CloudFormation experience for infrastructure provisioning
- **Monitoring & Observability**: Implemented health checks, liveness/readiness probes, and logging strategies

## Quick Start

### Prerequisites
- AWS CLI configured
- kubectl installed
- Helm installed
- ArgoCD CLI (optional)

### Deploy to AWS EKS

1. **Set up EKS Cluster** (choose EC2 instance type wisely for ~10GB project with monitoring):
   ```bash
   eksctl create cluster --name microservices-cluster --region ap-south-1 --node-type t3.large --nodes 3
   ```

2. **Install ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Install Kube-Prometheus-Stack for Monitoring**:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
   ```

4. **Deploy Application**:
   ```bash
   helm install microservices ./kubernetes
   ```

5. **Access Application** (NodePort: 31328):
   ```bash
   kubectl get svc frontend -n application
   # Access via: http://<EC2-Public-IP>:31328
   ```

6. **Access Grafana** (Port 80 via LoadBalancer - planned):
   ```bash
   kubectl get svc monitoring-grafana -n monitoring
   # Login: admin / prom-operator
   # Dashboards: Kubernetes (12740), Node Exporter (1860), Namespace (15758)
   ```

7. **Cleanup EKS Cluster**:
   ```bash
   eksctl delete cluster --name microservices-cluster --region ap-south-1
   ```

## Development

### Local Development
Use Docker Compose for local testing:
```bash
cd ../google-microservices-DevSecOps
docker-compose up
```

### Adding New Services
1. Add service to CI repo with Dockerfile
2. Create Kubernetes manifests in CD repo
3. Update Helm chart and values
4. Configure ArgoCD application

## Security Features

- **DevSecOps Integration**: Comprehensive security scanning in CI pipelines with Snyk integration planned
- **RBAC**: Kubernetes role-based access control with principle of least privilege
- **Network Policies**: Service mesh isolation and traffic encryption (planned with Istio)
- **Secrets Management**: AWS Secrets Manager integration for secure credential handling
- **Container Security**: Multi-stage Docker builds with minimal attack surface
- **Compliance**: Audit trails and compliance checks integrated into pipelines

## Monitoring & Observability

- **Kube-Prometheus-Stack**: Manually deployed Helm chart with Prometheus, Grafana, and Alertmanager
- **Health Checks**: Kubernetes liveness/readiness probes implemented across all services
- **Logging**: Centralized logging strategy with structured logging and correlation IDs
- **Metrics**: Prometheus metrics collection with custom application metrics and service monitoring
- **Dashboards**: Grafana with pre-configured dashboards:
  - Kubernetes Monitoring (ID: 12740)
  - Node Exporter (ID: 1860)
  - Kubernetes / Views / Namespace (ID: 15758)
- **Grafana Access**: Login with admin/prom-operator
- **Alerting**: Prometheus Alertmanager with Slack/email notifications for service health
- **Tracing**: Distributed tracing for request flow analysis (planned)

## Future Enhancements

- **Infrastructure as Code**: Complete Terraform/OpenTofu setup for EKS provisioning and infrastructure management
- **Container Registry Migration**: Transition from Docker Hub to ECR for enhanced security and compliance
- **Load Balancer Setup**: AWS Load Balancer for frontend service (replacing NodePort 31328)
- **Service Mesh**: Istio integration for advanced traffic management and security
- **Advanced Monitoring**: ELK stack integration with distributed tracing
- **Auto-scaling**: HPA/VPA policies with custom metrics and KEDA integration
- **Multi-region Deployment**: Cross-region failover and global load balancing
- **Blue-Green Deployments**: Zero-downtime deployment strategies with Argo Rollouts
- **Email Integration**: Complete email notification system for Jenkins pipelines (currently in development)
- **Snyk Integration**: Advanced vulnerability scanning and dependency management

## Infrastructure Requirements

### EC2 Instance Selection
For ~10GB project with monitoring stack, recommended instance types:
- **t3.large** (8GB RAM, 2 vCPU) - Minimum for basic functionality
- **t3.xlarge** (16GB RAM, 4 vCPU) - Recommended for full monitoring stack
- **t3.2xlarge** (32GB RAM, 8 vCPU) - For production workloads with heavy monitoring

### Security Group Ports
| Service | Port | Protocol | Source |
|---------|------|----------|--------|
| HTTP | 80 | TCP | 0.0.0.0/0 |
| HTTPS | 443 | TCP | 0.0.0.0/0 |
| SSH | 22 | TCP | Your IP |
| Jenkins | 8080 | TCP | Your IP |
| SonarQube | 9000 | TCP | Your IP |
| Frontend (NodePort) | 31328 | TCP | 0.0.0.0/0 |
| Grafana (LoadBalancer) | 80 | TCP | 0.0.0.0/0 |

### Screenshots (Planned)
- [ ] EKS cluster creation confirmation
- [ ] ArgoCD dashboard with application sync status
- [ ] Grafana dashboards showing application metrics
- [ ] Jenkins pipeline execution with build artifacts
- [ ] Application frontend running on NodePort 31328
- [ ] Prometheus targets and alerting rules

This setup demonstrates production-ready DevSecOps practices for microservices deployment on AWS, showcasing expertise in cloud-native technologies and GitOps methodologies.
