# Employee Platform GitOps

GitOps repository containing Kubernetes deployment configuration and environment-specific application configuration for the Employee Platform.

Argo CD continuously reconciles the configuration in this repository with the K3s cluster.

## Repository Structure

```text
employee-platform-gitops/
├── apps/
│   ├── database/
│   │   └── mysql.yaml
│   ├── staging/
│   │   └── values.yaml
│   └── production/
│       └── values.yaml
├── argocd/
│   ├── staging/
│   │   ├── employee-platform.yaml
│   │   └── mysql.yaml
│   └── production/
│       └── employee-platform.yaml
├── monitoring/
│   ├── kube-prometheus-stack.yaml
│   └── loki-alloy.yaml
└── README.md
```

## GitOps Architecture

```text
Application Repository
        │
        │ GitHub Actions
        ▼
      GHCR
        │
        │ image SHA
        ▼
   GitOps Repository
        │
        ▼
      Argo CD
        │
        ▼
       K3s
        │
        ├── Staging
        ├── Production
        └── Monitoring
```

## Environments

### Staging

```text
Namespace: employee-staging
Host: staging.employee.local
```

### Production

```text
Namespace: employee-production
Host: employee.local
```

## Application Deployment

Argo CD Applications manage the Employee Platform deployments.

Staging:

```text
employee-platform-staging
```

Production:

```text
employee-platform-production
```

Both applications use automated synchronization with:

- Automated sync
- Pruning
- Self-healing
- Automatic namespace creation

## Immutable Image Versions

Application images are deployed using Git commit SHA values.

Example:

```text
ghcr.io/diyanme/employee-platform:<commit-sha>
```

This avoids relying on mutable deployment tags such as `latest`.

GitHub Actions automatically updates the appropriate environment values file after successfully publishing a new image.

Example production update:

```text
Update main image to <commit-sha>
```

## Database

MySQL is deployed in the staging namespace through an Argo CD-managed application.

Resources include:

- MySQL Deployment
- MySQL Service
- Kubernetes Secret
- PersistentVolumeClaim

## Monitoring

The monitoring stack is managed through Argo CD.

Components include:

- Prometheus
- Grafana
- kube-state-metrics
- Prometheus Node Exporter
- Loki
- Grafana Alloy

### Metrics

Prometheus collects Kubernetes and node metrics.

Grafana provides dashboards and visualization.

### Logs

Grafana Alloy discovers Kubernetes pods and forwards container logs to Loki.

```text
Kubernetes Pods
      │
      ▼
Grafana Alloy
      │
      ▼
     Loki
      │
      ▼
   Grafana
```

## Argo CD Applications

The GitOps repository manages:

```text
employee-platform-staging
employee-platform-production
mysql-staging
kube-prometheus-stack
loki
grafana-alloy
```

Check applications:

```bash
kubectl get applications -n argocd
```

## Verification

Check Kubernetes resources:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
```

Check Argo CD:

```bash
kubectl get applications -n argocd
```

Check monitoring:

```bash
kubectl get pods -n monitoring
```

## GitOps Principles Used

- Git is the source of deployment configuration.
- Argo CD continuously reconciles desired and actual state.
- Application images use immutable commit SHA versions.
- Environment configuration is separated into staging and production.
- Deployment changes are recorded through Git history.
- Kubernetes resources are managed declaratively.

## Related Repositories

- Application: `employee-platform`
- Infrastructure: `employee-platform-infrastructure`
