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

- **Platform**: AWS EKS
- **GitOps Tool**: ArgoCD
- **Package Manager**: Helm
- **CI/CD**: Jenkins
- **Container Registry**: Docker Hub / ECR
- **Monitoring**: Prometheus + Grafana (planned)

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
- `GitOps/Jenkinsfile`: Pipeline that updates image tags from CI builds

## Skills Demonstrated

- **Microservices Architecture**: Designed and implemented 10+ services with gRPC communication
- **Containerization**: Created optimized Dockerfiles for multi-language stack
- **Kubernetes**: Managed deployments, services, and namespaces on EKS
- **GitOps**: Implemented ArgoCD for automated deployments
- **CI/CD**: Built Jenkins pipelines for build, test, and deploy
- **Helm**: Packaged applications for Kubernetes deployment
- **AWS**: Deployed and managed EKS clusters
- **Monitoring**: Set up application observability (in progress)

## Quick Start

### Prerequisites
- AWS CLI configured
- kubectl installed
- Helm installed
- ArgoCD CLI (optional)

### Deploy to AWS EKS

1. **Set up EKS Cluster**:
   ```bash
   eksctl create cluster --name microservices-cluster --region ap-south-1
   ```

2. **Install ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Deploy Application**:
   ```bash
   helm install microservices ./kubernetes
   ```

4. **Access Application**:
   ```bash
   kubectl get svc frontend -n application
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

- **DevSecOps Integration**: Security scanning in CI pipelines
- **RBAC**: Kubernetes role-based access control
- **Network Policies**: Service mesh isolation (planned)
- **Secrets Management**: AWS Secrets Manager integration (planned)

## Monitoring & Observability

- **Health Checks**: Kubernetes liveness/readiness probes
- **Logging**: Centralized logging with Fluentd (planned)
- **Metrics**: Prometheus metrics collection
- **Dashboards**: Grafana visualization (planned)

## Future Enhancements

- Service mesh with Istio
- Advanced monitoring stack
- Auto-scaling policies
- Multi-region deployment
- Blue-green deployments

This setup demonstrates production-ready DevSecOps practices for microservices deployment on AWS, showcasing expertise in cloud-native technologies and GitOps methodologies.
