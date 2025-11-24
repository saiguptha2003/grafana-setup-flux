# Local Testing Grafana Deployment (MVP)

This directory contains the local testing configuration for the central Grafana setup using Flux GitOps.

## Components Included

- **Kube-Prometheus-Stack**: Complete monitoring stack with Prometheus, Grafana, and AlertManager
- **Loki Stack**: Log aggregation with Loki and Promtail (using internal MinIO)
- **Grafana Dashboards**: Flux monitoring dashboards
- **MSCQA Org Configuration**: Organization-specific datasources and teams

## Prerequisites

1. Local Kubernetes cluster (kind, minikube, k3d, etc.)
2. Flux v2 installed
3. kubectl configured to access your cluster

## Deployment

### 1. Environment Setup

The configuration uses simple defaults suitable for local testing:
- **Grafana Admin**: admin/admin
- **Storage**: Uses standard storage class
- **Resources**: Minimal resource requirements

### 2. Deploy via Flux

Apply the cluster configuration to your Flux-managed cluster:
```bash
kubectl apply -f ../../../clusters/a1/app-configuration/monitoring-grafana.yaml
```

### 3. Verify Deployment

Check Flux Kustomizations:
```bash
flux get kustomizations
```

Check HelmReleases:
```bash
flux get helmreleases -n prd-monitoring
```

Check pods:
```bash
kubectl get pods -n prd-monitoring
```

## Configuration

### Local Testing Settings

- **Grafana Admin Password**: `admin`
- **Persistence**: Enabled with standard storage class
- **Retention**: 
  - Prometheus: 7 days
  - Loki: 1 day
- **Access**: NodePort on port 30000
- **Resources**: Minimal for local testing
- **Storage**: MinIO for Loki (no external dependencies)

### Storage

#### Grafana
- Persistent volume: 2Gi
- Storage class: `standard`

#### Prometheus
- Persistent volume: 5Gi
- Storage class: `standard`

#### Loki
- Uses internal MinIO for storage
- Persistent volume: 5Gi

### Security

- Simple admin credentials for testing
- No TLS (local access only)
- No external authentication

## Access

Once deployed, Grafana will be available at:
- **URL**: http://localhost:30000 (via NodePort)
- **Username**: admin
- **Password**: admin

To access via port-forward:
```bash
kubectl port-forward -n prd-monitoring svc/kube-prometheus-stack-grafana 3000:80
```
Then access at: http://localhost:3000

## Monitoring

The setup includes monitoring for:
- Flux GitOps controllers
- Kubernetes cluster metrics
- Application logs via Loki
- Custom dashboards for Flux operations

## Troubleshooting

### Check Flux Status
```bash
flux get all
```

### Check Pod Status
```bash
kubectl get pods -n prd-monitoring
```

### Check Logs
```bash
kubectl logs -n prd-monitoring -l app.kubernetes.io/name=grafana
kubectl logs -n prd-monitoring -l app.kubernetes.io/name=loki
```

### Test Kustomize Build Locally
```bash
# From repository root
kustomize build central-grafana/environments/prd/monitoring
```

## Local Development

This is a simplified MVP setup for local testing:

1. **No external dependencies**: Uses MinIO instead of AWS S3
2. **Simple authentication**: Fixed admin credentials
3. **Minimal resources**: Suitable for local development
4. **Short retention**: 1-7 days for faster testing cycles
5. **NodePort access**: No ingress controller required

## Next Steps

For production deployment, you would:
1. Configure proper storage classes
2. Set up ingress with TLS
3. Configure external authentication
4. Set up proper backup strategies
5. Configure monitoring and alerting
6. Scale resources appropriately