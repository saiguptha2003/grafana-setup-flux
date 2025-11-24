# Grafana Central Monitoring Setup (MVP)

This repository contains a complete GitOps-based monitoring solution using Flux, Grafana, Prometheus, and Loki for local Kubernetes clusters testing.

## Repository Structure

```
.
├── central-grafana/
│   ├── base/                     # Base configuration for all environments
│   │   ├── configs/             # Grafana dashboards and monitoring configs
│   │   ├── controllers/         # Helm releases for monitoring components
│   │   │   ├── kube-prometheus-stack/
│   │   │   └── loki-stack/
│   │   ├── orgs/               # Organization-specific configurations
│   │   └── kustomization.yaml  # Main base kustomization
│   └── environments/
│       └── prd/                # Production environment (configured for local testing)
│           └── monitoring/     # Local testing overlays and patches
└── clusters/
    └───a1/
        └───app-configuration/  # Cluster-specific Flux manifests
```

## Components

### Monitoring Stack
- **kube-prometheus-stack**: Prometheus, Grafana, AlertManager
- **Loki Stack**: Log aggregation with Loki and Promtail (with MinIO)
- **Custom Dashboards**: Flux-specific monitoring dashboards
- **Organization Config**: MSCQA team and datasource configurations

### GitOps Integration
- **Flux GitOps**: Complete GitOps deployment pipeline
- **Kustomize**: Environment-specific configuration management
- **Helm**: Package management for monitoring components

## Quick Start (Local Testing)

### Prerequisites
1. Local Kubernetes cluster (kind, minikube, k3d, etc.)
2. Flux v2 installed
3. kubectl configured to access your cluster

### Deploy to Local Cluster

1. **Apply Cluster Configuration**:
   ```bash
   kubectl apply -f clusters/a1/app-configuration/monitoring-grafana.yaml
   ```

2. **Monitor Deployment**:
   ```bash
   # Check Flux status
   flux get kustomizations
   
   # Check HelmReleases
   flux get helmreleases -n prd-monitoring
   
   # Check pods
   kubectl get pods -n prd-monitoring
   ```

3. **Access Grafana**:
   ```bash
   # Via NodePort (default)
   # Grafana will be available at: http://localhost:30000
   
   # Or via port-forward
   kubectl port-forward -n prd-monitoring svc/kube-prometheus-stack-grafana 3000:80
   # Then access at: http://localhost:3000
   ```

4. **Login Credentials**:
   - **Username**: admin
   - **Password**: admin

## Testing Locally

### Test Kustomize Builds

```bash
# Test base configuration
kustomize build central-grafana/base

# Test production environment
kustomize build central-grafana/environments/prd/monitoring

# Test specific components
kustomize build central-grafana/base/configs
kustomize build central-grafana/base/controllers/kube-prometheus-stack
```

## MVP Configuration

### Local Testing Features
- **Simple Authentication**: Fixed admin/admin credentials
- **MinIO Storage**: No external dependencies for Loki
- **Standard Storage**: Uses default storage class
- **Minimal Resources**: Optimized for local development
- **NodePort Access**: No ingress controller required
- **Short Retention**: 1-7 days for faster testing

### Resource Requirements
- **CPU**: ~350m total
- **Memory**: ~1.5Gi total
- **Storage**: ~12Gi total (Grafana 2Gi + Prometheus 5Gi + Loki 5Gi)

### Storage Classes
The configuration uses `standard` storage class which should be available in most local clusters.

## Architecture

### Data Flow
1. **Git Repository** → Flux GitRepository Controller
2. **Flux Kustomization** → Kustomize Controller  
3. **HelmRelease** → Helm Controller
4. **Kubernetes Resources** → Monitoring Stack

### Access Points
- **Grafana UI**: http://localhost:30000 (NodePort)
- **Prometheus UI**: Port-forward to access prometheus:9090
- **AlertManager UI**: Port-forward to access alertmanager:9093

## What's Included

### Dashboards
- Flux Control Plane monitoring
- Cluster statistics
- Log aggregation views

### Data Sources
- Prometheus (metrics)
- Loki (logs)
- AlertManager (alerts)

### Monitoring Targets
- Flux GitOps controllers
- Kubernetes cluster metrics
- Application logs via Promtail

## Customization

### Adding New Dashboards
1. Place JSON dashboard files in `central-grafana/base/configs/dashboards/`
2. Update `central-grafana/base/configs/kustomization.yaml`

### Modifying Resources
Edit the patch files in `central-grafana/environments/prd/monitoring/`:
- `production-grafana-patch.yaml`
- `production-loki-patch.yaml`

### Changing Access Method
To use LoadBalancer instead of NodePort:
```yaml
# In production-grafana-patch.yaml
service:
  type: LoadBalancer
```

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check storage class availability
   ```bash
   kubectl get storageclass
   ```

2. **Flux reconciliation issues**:
   ```bash
   flux get all
   flux reconcile kustomization central-grafana-prd
   ```

3. **Resource constraints**: Reduce resource limits in patch files

### Useful Commands
```bash
# Check all monitoring pods
kubectl get pods -n prd-monitoring

# Check Grafana logs
kubectl logs -n prd-monitoring -l app.kubernetes.io/name=grafana

# Check Loki logs
kubectl logs -n prd-monitoring -l app.kubernetes.io/name=loki

# Get Grafana service details
kubectl get svc -n prd-monitoring kube-prometheus-stack-grafana
```

## Next Steps

This MVP setup provides:
- ✅ Complete monitoring stack
- ✅ GitOps deployment
- ✅ Local testing capability
- ✅ Flux monitoring
- ✅ Log aggregation

For production deployment, consider:
- External authentication (LDAP, OAuth)
- Ingress with TLS
- External storage (AWS S3, GCS)
- Backup strategies
- High availability setup
- Resource scaling
- Security policies

## Contributing

1. Test changes locally with `kustomize build`
2. Verify deployment in local cluster
3. Submit PR with clear description
4. Monitor Flux reconciliation