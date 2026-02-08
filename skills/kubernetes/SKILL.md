---
name: kubernetes
description: "Deploy and manage Kubernetes applications. Use when working with K8s manifests, Helm charts, deployment strategies, or cluster configuration."
---

# Kubernetes Skill

Deploy and manage applications in Kubernetes clusters.

## When to Use

Use this skill when the user wants to:
- Create Kubernetes manifests (YAML)
- Use Helm charts for deployments
- Configure Pods, Services, Deployments
- Set up Helm repositories
- Implement K8s networking
- Configure RBAC and security
- Use ingress controllers

## Kubernetes Resources

### Core Resources
- **Pod**: Single container instance
- **Deployment**: Replica set management
- **Service**: Network access abstraction
- **StatefulSet**: Stateful applications
- **DaemonSet**: System services

### Ingress & Networking
- Ingress resources
- Service types (ClusterIP, NodePort, LoadBalancer)
- NetworkPolicies
- Ingress controllers (NGINX, Traefik)

### Storage
- PersistentVolumes
- PersistentVolumeClaims
- StorageClasses
- VolumeMounts

## Helm Charts

```yaml
# Chart structure
my-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── .helmignore
```

## Deployment Strategies

- **RollingUpdate**: Gradual rollout
- **Recreate**: Stop all pods first
- **Canary**: Incremental rollout
- **Blue/Green**: Parallel deployment

## Best Practices

- Use labels and selectors consistently
- Configure resource limits and requests
- Use environment variables for config
- Implement health checks (liveness, readiness)
- Use secrets for sensitive data
- Add annotations for observability

## Deliverables

- Kubernetes manifests
- Helm chart (if using Helm)
- Deployment documentation
- Testing scripts (kubetest)

## Quality Checklist

- All resources have proper labels
- Resource limits set
- Health checks configured
- Service discovery works
- ConfigMaps/Secrets properly used
- RBAC configured where needed
