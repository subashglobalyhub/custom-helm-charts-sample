# Backend Web Application Deployment Guide

This guide provides comprehensive instructions for deploying backend web applications on Kubernetes using Helm charts. This Helm chart automates the deployment of containerized backend applications with production-ready features including auto-scaling, ingress management, and resource optimization.

## Prerequisites

- Kubernetes cluster (v1.19+)
- Helm 3.x installed
- kubectl configured with cluster access
- (Optional) cert-manager installed for TLS certificate management
- (Optional) NGINX Ingress Controller installed
- (Optional) Image pull secret configured for private container registries

## Components

This Helm chart deploys the following Kubernetes resources:

- **Deployment**: Manages the application pods with configurable resource limits and requests
- **Service**: ClusterIP service for internal communication
- **Ingress**: External access with TLS support and cert-manager integration
- **HorizontalPodAutoscaler (HPA)**: Automatic scaling based on CPU utilization

## Chart Information

- **Chart Name**: `backend-webapp`
- **Chart Version**: `0.1.0`
- **App Version**: `1.16.0`

## Installation Steps

### 1. Basic Installation

Install the chart with default values:

```bash
helm install subash-webapp-release webserver/ \
  --values webserver/values.yaml \
  --namespace custom-backend \
  --create-namespace
```

### 2. Verify Installation

Check that all resources are deployed successfully:

```bash
# Check pods
kubectl get pods -n custom-backend

# Check services
kubectl get svc -n custom-backend

# Check ingress
kubectl get ingress -n custom-backend

# Check HPA
kubectl get hpa -n custom-backend
```

### 3. Environment-Specific Deployments

#### Development Environment

```bash
# Create namespace
kubectl create namespace dev

# Install with dev configuration
helm install mywebapp-release-dev webserver/ \
  --values webserver/values.yaml \
  -f webserver/values-dev.yaml \
  -n dev
```

#### Production Environment

```bash
# Create namespace
kubectl create namespace prod

# Install with prod configuration
helm install mywebapp-release-prod webserver/ \
  --values webserver/values.yaml \
  -f webserver/values-prod.yaml \
  -n prod
```

**Note**: Create `values-dev.yaml` and `values-prod.yaml` files to override default values for each environment.

## Configuration

### Key Configuration Options

The chart can be customized through `values.yaml`. Key parameters include:

#### Application Configuration
- `appLabel`: Application label used for resource naming
- `maintainer`: Maintainer label for resource tracking
- `image.repository`: Container image repository
- `image.tag`: Container image tag
- `imagePullSecretName`: Secret name for private registry authentication

#### Resource Limits
- `resources.requests.cpu`: CPU request (e.g., "250m")
- `resources.requests.memory`: Memory request (e.g., "600Mi")
- `resources.limits.cpu`: CPU limit (e.g., "500m")
- `resources.limits.memory`: Memory limit (e.g., "900Mi")

#### Service Configuration
- `service.type`: Service type (ClusterIP, NodePort, LoadBalancer)
- `service.port`: Service port
- `service.targetPort`: Container target port

#### Horizontal Pod Autoscaling
- `hpa.enabled`: Enable/disable HPA
- `hpa.minReplicas`: Minimum number of replicas
- `hpa.maxReplicas`: Maximum number of replicas
- `hpa.cpu.targetAverageUtilization`: CPU target for scaling (percentage)
- `hpa.scaleUp.stabilizationWindowSeconds`: Scale-up stabilization window
- `hpa.scaleUp.podsStep`: Number of pods to add per scale-up
- `hpa.scaleDown.stabilizationWindowSeconds`: Scale-down stabilization window
- `hpa.scaleDown.podsStep`: Number of pods to remove per scale-down

#### Ingress Configuration
- `ingress.host`: Domain name for the application
- `ingress.tls.secretName`: TLS secret name (managed by cert-manager)

### Customizing Values

Create a custom values file or override values during installation:

```bash
# Override specific values
helm install myapp webserver/ \
  --set image.repository=myregistry/myapp \
  --set image.tag=v1.0.0 \
  --set ingress.host=myapp.example.com \
  --namespace myapp \
  --create-namespace

# Use custom values file
helm install myapp webserver/ \
  --values webserver/values.yaml \
  --values my-custom-values.yaml \
  --namespace myapp \
  --create-namespace
```

## Management Commands

### List All Deployments

```bash
# List all Helm releases
helm list --all-namespaces

# List releases in specific namespace
helm list -n custom-backend
```

### Upgrade Deployment

```bash
# Upgrade with same values
helm upgrade subash-webapp-release webserver/ \
  --values webserver/values.yaml \
  --namespace custom-backend

# Upgrade with new image tag
helm upgrade subash-webapp-release webserver/ \
  --values webserver/values.yaml \
  --set image.tag=latest \
  --namespace custom-backend
```

### Rollback Deployment

```bash
# View release history
helm history subash-webapp-release -n custom-backend

# Rollback to previous version
helm rollback subash-webapp-release -n custom-backend

# Rollback to specific revision
helm rollback subash-webapp-release 2 -n custom-backend
```

### Uninstall Deployment

```bash
# Uninstall release
helm uninstall subash-webapp-release --namespace custom-backend

# Verify uninstallation
kubectl get all -n custom-backend
```

## Accessing the Application

### Port Forwarding (Local Access)

```bash
# Port forward to service
kubectl port-forward -n custom-backend svc/dotcom-backend-svc 8080:80

# Access at http://localhost:8080
```

### Ingress Access

If using Minikube or local cluster:

```bash
# Enable ingress addon (Minikube)
minikube addons enable ingress

# Start tunnel (Minikube)
minikube tunnel
```

Access the application via the configured ingress host: `https://backend.domain.com`

### Get Service Endpoint

```bash
# Get service details
kubectl get svc dotcom-backend-svc -n custom-backend

# Get ingress details
kubectl get ingress dotcom-backend-ingress -n custom-backend
```

## Troubleshooting

### Check Pod Status

```bash
# List all pods
kubectl get pods -n custom-backend

# Get detailed pod information
kubectl describe pod <pod-name> -n custom-backend

# Check pod events
kubectl get events -n custom-backend --sort-by='.lastTimestamp'
```

### View Pod Logs

```bash
# View logs for a specific pod
kubectl logs <pod-name> -n custom-backend

# Follow logs in real-time
kubectl logs -f <pod-name> -n custom-backend

# View logs for all pods with label
kubectl logs -l app=dotcom-backend -n custom-backend

# View previous container logs (if pod restarted)
kubectl logs <pod-name> -n custom-backend --previous
```

### Check Deployment Status

```bash
# Get deployment status
kubectl get deployment -n custom-backend

# Describe deployment
kubectl describe deployment dotcom-backend-deployment -n custom-backend

# Check deployment rollout status
kubectl rollout status deployment/dotcom-backend-deployment -n custom-backend

# View deployment history
kubectl rollout history deployment/dotcom-backend-deployment -n custom-backend
```

### Check Service and Ingress

```bash
# Check service endpoints
kubectl get endpoints dotcom-backend-svc -n custom-backend

# Describe service
kubectl describe svc dotcom-backend-svc -n custom-backend

# Check ingress
kubectl describe ingress dotcom-backend-ingress -n custom-backend

# Test service connectivity from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -n custom-backend -- wget -O- http://dotcom-backend-svc:80
```

### Check HPA Status

```bash
# Get HPA status
kubectl get hpa -n custom-backend

# Describe HPA
kubectl describe hpa dotcom-backend-hpa -n custom-backend

# Check HPA metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/custom-backend/pods
```

### Check Resource Usage

```bash
# Check pod resource usage
kubectl top pods -n custom-backend

# Check node resource usage
kubectl top nodes

# Check resource quotas (if any)
kubectl describe quota -n custom-backend
```

### Image Pull Issues

```bash
# Check image pull secrets
kubectl get secrets -n custom-backend

# Describe secret
kubectl describe secret ecr-registry-secret -n custom-backend

# Verify image pull secret is configured
kubectl get deployment dotcom-backend-deployment -n custom-backend -o yaml | grep imagePullSecrets
```

### Common Issues and Solutions

#### Pods in CrashLoopBackOff

```bash
# Check pod status
kubectl get pods -n custom-backend

# View logs
kubectl logs <pod-name> -n custom-backend

# Check events
kubectl describe pod <pod-name> -n custom-backend

# Check if resource limits are too restrictive
kubectl top pod <pod-name> -n custom-backend
```

#### Pods Not Starting

```bash
# Check if image exists and is accessible
docker pull <image-repository>:<image-tag>

# Verify image pull secret
kubectl get secret ecr-registry-secret -n custom-backend

# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"
```

#### Ingress Not Working

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress status
kubectl describe ingress dotcom-backend-ingress -n custom-backend

# Verify TLS certificate
kubectl get certificate -n custom-backend

# Check cert-manager logs
kubectl logs -n cert-manager -l app=cert-manager
```

#### HPA Not Scaling

```bash
# Check if metrics server is installed
kubectl get deployment metrics-server -n kube-system

# Check HPA status
kubectl describe hpa dotcom-backend-hpa -n custom-backend

# Verify pod metrics are available
kubectl top pods -n custom-backend
```

### Debugging Commands

```bash
# Execute command in pod
kubectl exec -it <pod-name> -n custom-backend -- /bin/sh

# Get shell access to pod
kubectl exec -it <pod-name> -n custom-backend -- /bin/bash

# Check environment variables
kubectl exec <pod-name> -n custom-backend -- env

# Test connectivity from pod
kubectl exec <pod-name> -n custom-backend -- wget -O- http://dotcom-backend-svc:80
```

### Helm Release Troubleshooting

```bash
# Check Helm release status
helm status subash-webapp-release -n custom-backend

# Get rendered templates (dry-run)
helm template subash-webapp-release webserver/ \
  --values webserver/values.yaml \
  --namespace custom-backend

# Validate chart
helm lint webserver/

# Check for issues
helm install subash-webapp-release webserver/ \
  --values webserver/values.yaml \
  --namespace custom-backend \
  --dry-run --debug
```

## Official Documentation

For detailed configuration options and advanced usage:

### Kubernetes Resources
- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

### Helm Resources
- [Helm Documentation](https://helm.sh/docs/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Helm Chart Template Guide](https://helm.sh/docs/chart_template_guide/)

### Ingress and TLS
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)

## Support

For additional help and community support:
- [Kubernetes Slack](https://kubernetes.slack.com/)
- [Helm GitHub Discussions](https://github.com/helm/helm/discussions)
- [Kubernetes Stack Overflow](https://stackoverflow.com/questions/tagged/kubernetes)
