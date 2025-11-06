# Homelab Infrastructure 🏠🖥️

A modern, containerized homelab infrastructure built with Kubernetes, GitOps principles, and Infrastructure as Code (IaC). This repository contains all configurations, applications, and automation scripts for managing a personal home laboratory environment.

## Overview

This homelab setup provides a production-like environment for learning, experimentation, and hosting personal services. Built with modern DevOps practices, it enables rapid deployment and management of containerized applications in a home environment.

## Features

- **GitOps Workflow**: Declarative infrastructure management with version control
- **Kubernetes Orchestration**: Container orchestration for scalable application deployment
- **Multi-Environment Support**: Staging and production cluster configurations
- **Automated Deployments**: Continuous deployment of applications
- **Infrastructure as Code**: All configurations stored as code
- **Dev Container Support**: Consistent development environment
- **Containerized Applications**: Docker-based application packaging
- **Service Discovery**: Internal DNS and service mesh capabilities

## Architecture

```
homelab/
├── .devcontainer/          # Development container configuration
├── apps/                   # Application definitions and configurations
├── clusters/               # Cluster configurations
│   └── staging/           # Staging environment setup
└── scripts/               # Automation and utility scripts
```

## Tech Stack

### Core Infrastructure
- **Kubernetes**: Container orchestration platform
- **Docker**: Containerization technology
- **GitOps Tool**: (Flux CD / ArgoCD) - Continuous deployment
- **Shell Scripts**: Automation and maintenance tasks

### Development Tools
- **Dev Containers**: Reproducible development environment
- **Git**: Version control
- **YAML**: Configuration management

### Networking & Storage
- **Container Networking**: Pod-to-pod communication
- **Persistent Storage**: Volume management
- **Ingress Controller**: External access management

## Prerequisites

### Required Software
- **Kubernetes Cluster**: 
  - K3s (recommended for homelab)
  - K8s (standard Kubernetes)
  - MicroK8s
  - Or any other Kubernetes distribution
- **kubectl**: Kubernetes command-line tool
- **Docker**: Container runtime
- **Git**: Version control system
- **Helm**: (Optional) Package manager for Kubernetes

### Hardware Requirements
- **Minimum**: 
  - 2 CPU cores
  - 4GB RAM
  - 50GB storage
- **Recommended**:
  - 4+ CPU cores
  - 8GB+ RAM
  - 100GB+ storage
  - Multiple nodes for high availability

### Network Requirements
- Static IP address or DHCP reservation
- Port forwarding for external access (optional)
- Internal DNS resolution

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/iminierai-aig/homelab.git
cd homelab
```

### 2. Set Up Kubernetes Cluster

#### Option A: K3s (Recommended for Homelab)

```bash
# Install K3s
curl -sfL https://get.k3s.io | sh -

# Verify installation
sudo k3s kubectl get nodes
```

#### Option B: Standard Kubernetes

```bash
# Follow official Kubernetes installation guide
# https://kubernetes.io/docs/setup/
```

### 3. Configure kubectl

```bash
# Copy K3s config (if using K3s)
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

# Verify connection
kubectl get nodes
```

### 4. Install GitOps Operator

#### For Flux CD:

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap Flux
flux bootstrap github \
  --owner=iminierai-aig \
  --repository=homelab \
  --branch=main \
  --path=clusters/staging \
  --personal
```

#### For ArgoCD:

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 5. Deploy Applications

```bash
# Applications will be automatically deployed via GitOps
# Monitor deployment status
kubectl get pods -A
```

## Configuration

### Cluster Configuration

Edit cluster-specific settings in `clusters/staging/`:

```yaml
# Example: flux-system configuration
apiVersion: v1
kind: Namespace
metadata:
  name: flux-system
```

### Application Deployment

Add new applications in the `apps/` directory:

```yaml
# Example: app deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
```

### Environment Variables

Create a `.env` file or use Kubernetes Secrets:

```bash
# Create a secret
kubectl create secret generic my-secret \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

## Usage

### Managing Applications

```bash
# List all applications
kubectl get deployments -A

# View application logs
kubectl logs -n namespace pod-name

# Scale application
kubectl scale deployment my-app --replicas=3

# Restart application
kubectl rollout restart deployment/my-app
```

### Monitoring Cluster Health

```bash
# Check node status
kubectl get nodes

# View all pods
kubectl get pods -A

# Check resource usage
kubectl top nodes
kubectl top pods -A
```

### Accessing Services

```bash
# Port forward to access services locally
kubectl port-forward svc/service-name 8080:80

# List all services
kubectl get svc -A
```

## Common Applications

This homelab can host various applications:

### Media Services
- Plex / Jellyfin - Media streaming
- Sonarr / Radarr - Media management
- Transmission / qBittorrent - Downloads

### Productivity
- Nextcloud - File storage and collaboration
- Home Assistant - Home automation
- Vaultwarden - Password management

### Monitoring
- Prometheus - Metrics collection
- Grafana - Visualization
- Loki - Log aggregation

### Development
- GitLab / Gitea - Git hosting
- Jenkins / Drone - CI/CD
- Harbor - Container registry

### Networking
- Pi-hole - DNS and ad-blocking
- Traefik / Nginx - Reverse proxy
- WireGuard - VPN

## Development

### Using Dev Container

The repository includes a dev container configuration for consistent development:

```bash
# Open in VS Code with Dev Containers extension
code .
# Click "Reopen in Container"
```

### Running Scripts

Execute automation scripts from the repository:

```bash
# Make scripts executable
chmod +x scripts/*.sh

# Run a script
./scripts/deploy.sh
```

### Testing Changes

```bash
# Validate Kubernetes manifests
kubectl apply --dry-run=client -f apps/

# Test in staging environment
kubectl apply -f apps/ -n staging
```

## Maintenance

### Backup Strategy

```bash
# Backup cluster configuration
kubectl get all -A -o yaml > cluster-backup.yaml

# Backup persistent volumes
# Use Velero or similar backup tools
```

### Updates

```bash
# Update K3s
curl -sfL https://get.k3s.io | sh -

# Update GitOps operator
flux install --export > clusters/staging/flux-system/gotk-components.yaml

# Update applications (via GitOps)
git pull origin main
```

### Troubleshooting

```bash
# Check pod status
kubectl describe pod pod-name

# View events
kubectl get events -A --sort-by='.lastTimestamp'

# Check logs
kubectl logs -f pod-name

# Debug networking
kubectl run debug --image=nicolaka/netshoot -it --rm
```

## Security

### Best Practices
- Use namespaces for isolation
- Implement RBAC (Role-Based Access Control)
- Use Network Policies to control traffic
- Encrypt secrets at rest
- Regular security updates
- Use private container registries
- Implement pod security policies

### Secrets Management

```bash
# Create sealed secrets (recommended)
kubectl create secret generic my-secret \
  --from-literal=password=mypassword \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml
```

## Networking

### Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: app.homelab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

### Service Types
- **ClusterIP**: Internal cluster communication
- **NodePort**: Access via node IP and port
- **LoadBalancer**: External load balancer (if available)
- **Ingress**: HTTP/HTTPS routing

## Monitoring & Observability

### Prometheus Setup

```bash
# Install Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

### Grafana Dashboards

- Node metrics
- Pod metrics
- Application metrics
- Network traffic
- Storage usage

## Performance Optimization

- Resource limits and requests
- Horizontal Pod Autoscaling (HPA)
- Persistent volume optimization
- Image optimization and caching
- Node affinity and anti-affinity rules

## Disaster Recovery

### Backup Procedures
1. Regular etcd backups (for K8s)
2. Persistent volume snapshots
3. Configuration backups (Git)
4. Database backups

### Recovery Steps
1. Restore cluster from backup
2. Re-apply configurations via GitOps
3. Restore persistent data
4. Verify application functionality

## Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/NewFeature`)
3. Commit your changes (`git commit -m 'Add NewFeature'`)
4. Push to the branch (`git push origin feature/NewFeature`)
5. Open a Pull Request

## Project Roadmap

- [ ] Add production cluster configuration
- [ ] Implement automated backups
- [ ] Add monitoring stack (Prometheus/Grafana)
- [ ] Set up service mesh (Istio/Linkerd)
- [ ] Implement centralized logging
- [ ] Add certificate management (cert-manager)
- [ ] Set up external DNS
- [ ] Implement disaster recovery procedures
- [ ] Add CI/CD pipelines
- [ ] Documentation for each application

## Useful Commands

### Kubernetes Cheat Sheet

```bash
# Get cluster info
kubectl cluster-info

# List all resources
kubectl get all -A

# Describe resource
kubectl describe <resource-type> <resource-name>

# Execute command in pod
kubectl exec -it pod-name -- /bin/bash

# Copy files to/from pod
kubectl cp local-file pod-name:/remote-path

# Apply configuration
kubectl apply -f config.yaml

# Delete resources
kubectl delete -f config.yaml
```

## Resources

### Documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [K3s Documentation](https://docs.k3s.io/)
- [Flux CD Documentation](https://fluxcd.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)

### Community
- [r/homelab](https://reddit.com/r/homelab)
- [r/kubernetes](https://reddit.com/r/kubernetes)
- [Kubernetes Slack](https://kubernetes.slack.com/)

### Learning Resources
- Kubernetes the Hard Way
- CNCF Landscape
- DevOps Toolkit YouTube Channel

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Kubernetes community
- K3s by Rancher Labs
- Flux CD / ArgoCD teams
- Homelab community

## Support

For issues or questions:
- Open an issue on [GitHub Issues](https://github.com/iminierai-aig/homelab/issues)
- Check the troubleshooting section
- Consult Kubernetes documentation

## Disclaimer

This is a personal homelab setup. While following best practices, it's designed for learning and experimentation. For production workloads, additional security hardening and reliability measures are recommended.

---

**Happy Homelabbing! 🚀**

Built with ❤️ for learning and self-hosting
