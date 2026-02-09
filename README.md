# Kubernetes GitOps Blueprint

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Kubernetes](https://img.shields.io/badge/kubernetes-v1.28+-blue.svg)](https://kubernetes.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.8+-blue.svg)](https://argoproj.github.io/cd/)

A production-ready GitOps blueprint for Kubernetes using ArgoCD, Helm, Kustomize, and GitHub Actions. Includes a sample microservices application with CI/CD, monitoring, and multi-environment deployments.

## 🏗️ Architecture

```
┌─────────────────┐
│   Developer     │
│   Git Push      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  GitHub Actions │
│  CI Pipeline    │
│  - Build        │
│  - Test         │
│  - Scan         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Container       │
│ Registry        │
│ (Docker Hub/    │
│  GHCR)          │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   ArgoCD        │
│   - Sync        │
│   - Monitor     │
│   - Self-Heal   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Kubernetes     │
│  Cluster        │
│  - API Service  │
│  - Worker       │
│  - Redis        │
│  - Monitoring   │
└─────────────────┘
```

## ✨ Features

- **GitOps Workflow**: Declarative infrastructure and application management with ArgoCD
- **Multi-Environment Support**: Dev, Staging, and Production environments using Kustomize overlays
- **Helm Charts**: Production-ready Helm charts for microservices (API, Worker, Redis)
- **CI/CD Pipeline**: Automated CI/CD with GitHub Actions for building, testing, and deploying
- **Monitoring Stack**: Prometheus and Grafana for observability
- **Auto-Scaling**: Horizontal Pod Autoscaling (HPA) configured for services
- **Security**: RBAC, network policies, and security scanning in CI
- **App-of-Apps Pattern**: Scalable ArgoCD application management

## 📋 Prerequisites

- Kubernetes cluster (v1.28+)
- ArgoCD installed and configured
- Helm 3.x installed
- kubectl configured with cluster access
- GitHub repository with Actions enabled
- Container registry access (Docker Hub, GHCR, etc.)

## 🚀 Quick Start

### 1. Bootstrap ArgoCD

First, ensure ArgoCD is installed in your cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Create ArgoCD Projects

Apply the project definitions:

```bash
kubectl apply -f argocd/projects/microservices.yaml
kubectl apply -f argocd/projects/monitoring.yaml
```

### 3. Deploy App-of-Apps

Deploy the root application that manages all other applications:

```bash
kubectl apply -f argocd/app-of-apps.yaml
```

ArgoCD will automatically discover and sync all applications defined in the `applications/` directory.

### 4. Monitor Deployment

Watch the ArgoCD applications:

```bash
kubectl get applications -n argocd
argocd app list
```

Access ArgoCD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open https://localhost:8080 (admin password from argocd-initial-admin-secret)
```

## 📁 Project Structure

```
kubernetes-gitops-blueprint/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── .gitignore                # Git ignore rules
├── argocd/                   # ArgoCD configuration
│   ├── app-of-apps.yaml     # Root application (app-of-apps pattern)
│   ├── projects/            # ArgoCD project definitions
│   │   ├── microservices.yaml
│   │   └── monitoring.yaml
│   └── applications/        # Individual application manifests
│       ├── api-service.yaml
│       ├── worker-service.yaml
│       ├── redis.yaml
│       └── monitoring-stack.yaml
├── helm-charts/             # Helm charts for services
│   ├── api-service/         # API service chart
│   ├── worker-service/      # Worker service chart
│   └── redis/               # Redis chart
├── kustomize/               # Kustomize configurations
│   ├── base/                # Base configuration
│   └── overlays/            # Environment-specific overlays
│       ├── dev/
│       ├── staging/
│       └── prod/
├── monitoring/              # Monitoring stack configs
│   ├── prometheus/          # Prometheus configuration
│   └── grafana/             # Grafana dashboards and configs
└── .github/
    └── workflows/           # GitHub Actions workflows
        ├── ci.yml           # CI pipeline
        └── release.yml      # Release pipeline
```

## 🌍 Environment Management

This blueprint uses Kustomize overlays to manage different environments:

### Development

```bash
kubectl apply -k kustomize/overlays/dev
```

- Reduced replicas (1-2)
- Lower resource limits
- Development-specific configurations

### Staging

```bash
kubectl apply -k kustomize/overlays/staging
```

- Medium replicas (2-3)
- Production-like resource limits
- Staging-specific configurations

### Production

```bash
kubectl apply -k kustomize/overlays/prod
```

- Higher replicas (3+)
- Production resource limits
- Enhanced monitoring and alerting

## 📊 Monitoring

### Prometheus

Prometheus is configured to scrape metrics from all services. Access the Prometheus UI:

```bash
kubectl port-forward svc/prometheus-server -n monitoring 9090:80
```

### Grafana

Grafana dashboards are automatically provisioned. Access Grafana:

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
# Default credentials: admin/admin
```

Pre-configured dashboards:
- Microservices Overview
- API Service Metrics
- Worker Service Metrics
- Redis Metrics

### Alerting

Prometheus alerting rules are defined in `monitoring/prometheus/rules/alerts.yaml`. Alerts include:
- High CPU/Memory usage
- Pod restart failures
- Service availability issues
- Redis connection failures

## 🔧 Configuration

### Helm Values

Customize deployments by modifying values in `helm-charts/*/values.yaml`:

- Resource requests/limits
- Replica counts
- Environment variables
- Service configurations
- Ingress settings

### ArgoCD Sync Policy

Applications are configured with automated sync policies:
- **Auto-sync**: Enabled for automatic deployment
- **Self-heal**: Automatically corrects drift
- **Prune**: Removes resources not in Git

To disable auto-sync for production, modify the sync policy in the application manifest.

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [ArgoCD](https://argoproj.github.io/cd/) for GitOps capabilities
- [Helm](https://helm.sh/) for package management
- [Kustomize](https://kustomize.io/) for configuration management
- [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) for monitoring

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitOps Principles](https://www.gitops.tech/)
