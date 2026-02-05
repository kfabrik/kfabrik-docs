# Addons

The KFabrik addon suite provides a complete machine learning inference platform for minikube, enabling local development and testing of Large Language Models (LLMs) with GPU support.

## Overview

The suite consists of three addons that work together:

```
kfabrik-bootstrap (required first)
       │
       ├──► kfabrik-model (optional, for LLM deployment)
       │
       └──► kfabrik-monitoring (optional, for observability)
```

The kfabrik-bootstrap addon must be enabled first as it provides the core infrastructure (KServe, Istio) that other addons depend on.

## kfabrik-bootstrap

### Purpose

The foundation addon that installs all core infrastructure components required for ML inference workloads.

### Components

| Component | Version | Purpose |
|-----------|---------|---------|
| Cert Manager | v1.16.1 | TLS certificate lifecycle management |
| Istio | v1.22.0 | Service mesh for traffic and security |
| KServe | v0.15.0 | Kubernetes-native ML model serving |
| NVIDIA Device Plugin | v0.14.1 | GPU resource discovery and allocation |

### Installation

```bash
# Enable the addon
minikube addons enable kfabrik-bootstrap

# Monitor installation progress
kubectl get jobs -n kserve -w

# Verify components are running
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n kserve
kubectl get pods -n kube-system | grep nvidia
```

### Namespaces Created

| Namespace | Contents |
|-----------|----------|
| cert-manager | Certificate management components and webhooks |
| istio-system | Istio control plane (istiod) |
| kserve | KServe controller and installer job |

### Configuration

Configuration is stored in a ConfigMap in the kserve namespace:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kfabrik-bootstrap-config
  namespace: kserve
data:
  CERT_MANAGER_VERSION: "v1.16.1"
  ISTIO_VERSION: "1.22.0"
  KSERVE_VERSION: "v0.15.0"
  INSTALL_CERT_MANAGER: "true"
  INSTALL_ISTIO: "true"
  INSTALL_KSERVE: "true"
```

### Design Decisions

**Helm-Based Installation:** Uses a Job-based installer that runs Helm charts rather than static YAML manifests. This provides version pinning, configuration management, and upgrade capabilities.

**RawDeployment Mode:** KServe is configured for RawDeployment mode (standard Kubernetes Deployments) rather than Serverless mode (Knative). This simplifies setup, debugging, and operation for local development. Knative support is planned for a future release.

**Job-Based Installer:** A Kubernetes Job orchestrates sequential installation of components with proper dependency ordering. The job self-cleans after 5 minutes (TTL).

### Troubleshooting

**Check installer job:**
```bash
kubectl get jobs -n kserve
kubectl logs -n kserve -l job-name=kfabrik-installer
```

**Verify GPU detection:**
```bash
kubectl get nodes -o json | jq '.items[].status.capacity'
kubectl logs -n kube-system -l name=nvidia-device-plugin-ds
```

---

## kfabrik-model

### Purpose

Provides model deployment configurations and the model-serving namespace for running LLM inference workloads.

### Pre-Configured Models

All models are optimized for consumer GPUs with 6GB VRAM or less:

| Name | Parameters | VRAM | RAM | Download |
|------|------------|------|-----|----------|
| qwen-small | 0.5B | ~1GB | ~6GB | ~1GB |
| qwen-medium | 1.5B | ~3GB | ~8GB | ~3GB |
| tinyllama | 1.1B | ~2.5GB | ~6GB | ~2.2GB |
| smollm2 | 1.7B | ~3.5GB | ~8GB | ~3.4GB |
| phi2 | 2.7B | ~5.5GB | ~12GB | ~5.5GB |

RAM requirements are approximately 2-3x model size due to inference server overhead. Models are deployed one at a time.

### Installation

```bash
# Requires kfabrik-bootstrap to be enabled first
minikube addons enable kfabrik-bootstrap

# Wait for bootstrap to complete
kubectl wait --for=condition=Ready pod -l app=istiod \
  -n istio-system --timeout=300s

# Enable kfabrik-model
minikube addons enable kfabrik-model
```

### Namespace Created

| Namespace | Contents |
|-----------|----------|
| model-serving | Deployed InferenceServices and model configuration ConfigMap |

### Model Configuration

Model definitions are stored in a ConfigMap:

```bash
kubectl get configmap model-config -n model-serving -o yaml
```

Example model configuration:

```yaml
models:
  qwen-small:
    name: qwen-small
    displayName: "Qwen 2.5 0.5B Instruct"
    modelFormat: huggingface
    storageUri: "hf://Qwen/Qwen2.5-0.5B-Instruct"
    vram: "1GB"
    ram: "6GB"
    downloadSize: "1GB"
    parameters: "0.5B"
    replicas: 1
    timeout: 300
    resources:
      requests:
        cpu: "1"
        memory: "4Gi"
      limits:
        cpu: "4"
        memory: "8Gi"
    env:
      HF_MODEL_ID: "Qwen/Qwen2.5-0.5B-Instruct"
      MAX_MODEL_LEN: "512"
      GPU_MEMORY_UTILIZATION: "0.8"
```

### Using with kfabrik CLI

```bash
# List available models
kfabrik list

# Deploy a model
kfabrik deploy --models qwen-small --wait

# Query the model
kfabrik query --model qwen-small --prompt "What is AI?"

# Delete the model
kfabrik delete --model qwen-small
```

### Custom Models

To add custom models, create a custom configuration file:

```yaml
# my-models.yaml
namespace: model-serving
models:
  my-model:
    name: my-model
    displayName: "My Custom Model"
    modelFormat: huggingface
    storageUri: "hf://organization/model-name"
    resources:
      requests:
        cpu: "2"
        memory: "8Gi"
      limits:
        cpu: "4"
        memory: "16Gi"
```

Then deploy with:
```bash
kfabrik deploy --config my-models.yaml --models my-model
```

---

## kfabrik-monitoring

### Purpose

Provides a complete observability stack for monitoring ML inference workloads, including GPU metrics.

### Components

| Component | Version | Purpose |
|-----------|---------|---------|
| Prometheus | v2.51.0 | Metrics collection and storage |
| Grafana | 10.4.0 | Visualization and dashboards |
| DCGM Exporter | 4.5.1 | NVIDIA GPU metrics exporter |

### Installation

```bash
# Enable after kfabrik-bootstrap
minikube addons enable kfabrik-monitoring

# Verify components
kubectl get pods -n monitoring
```

### Namespace Created

| Namespace | Contents |
|-----------|----------|
| monitoring | Prometheus, Grafana, and DCGM Exporter |

### Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kfabrik-monitoring-config
  namespace: monitoring
data:
  PROMETHEUS_VERSION: "v2.51.0"
  GRAFANA_VERSION: "10.4.0"
  DCGM_EXPORTER_VERSION: "4.5.1-4.8.0"
  PROMETHEUS_RETENTION: "15d"
  PROMETHEUS_STORAGE_SIZE: "10Gi"
  GRAFANA_ADMIN_USER: "admin"
  GRAFANA_ADMIN_PASSWORD: "admin"
```

### Accessing Dashboards

**Grafana:**
```bash
kubectl port-forward -n monitoring svc/grafana 3000:3000
# Open http://localhost:3000
# Login: admin / admin
```

**Prometheus:**
```bash
kubectl port-forward -n monitoring svc/prometheus 9090:9090
# Open http://localhost:9090
```

### GPU Metrics

The DCGM Exporter provides detailed GPU metrics:

| Metric | Description |
|--------|-------------|
| DCGM_FI_DEV_GPU_UTIL | GPU utilization percentage |
| DCGM_FI_DEV_MEM_COPY_UTIL | Memory copy utilization |
| DCGM_FI_DEV_FB_USED | Framebuffer memory used (bytes) |
| DCGM_FI_DEV_FB_FREE | Framebuffer memory free (bytes) |
| DCGM_FI_DEV_GPU_TEMP | GPU temperature (Celsius) |
| DCGM_FI_DEV_POWER_USAGE | Power consumption (Watts) |

---

## Resource Requirements

### Minimum Requirements

| Resource | Minimum |
|----------|---------|
| CPU | 4 cores |
| Memory | 8GB RAM |
| Disk | 40GB |

### Recommended for GPU Workloads

| Resource | Recommended |
|----------|-------------|
| CPU | 8+ cores |
| Memory | 16-32GB RAM |
| Disk | 50GB+ |
| GPU VRAM | 6GB+ |

### Starting minikube with Appropriate Resources

```bash
# Set persistent defaults
minikube config set cpus 8
minikube config set memory 32768
minikube config set disk-size 50g

# Start with GPU support
minikube start --driver=docker --gpus=all
```

---

## Disabling Addons

```bash
# Disable in reverse order of dependencies
minikube addons disable kfabrik-monitoring
minikube addons disable kfabrik-model
minikube addons disable kfabrik-bootstrap
```

For complete cleanup including CRDs:

```bash
# Delete Helm releases
helm uninstall kserve -n kserve
helm uninstall istiod -n istio-system
helm uninstall istio-base -n istio-system
helm uninstall cert-manager -n cert-manager

# Delete namespaces
kubectl delete namespace model-serving monitoring \
  kserve istio-system cert-manager

# Delete CRDs
kubectl get crd | grep -E 'kserve|istio|cert-manager' | \
  awk '{print $1}' | xargs kubectl delete crd
```

---

## Troubleshooting

### Addon Enable Fails

```bash
# Check addon status
minikube addons list | grep kfabrik

# View installer logs
kubectl logs -n kserve -l job-name=kfabrik-installer

# Check events
kubectl get events -n kserve --sort-by='.lastTimestamp'
```

### Models Not Deploying

```bash
# Check InferenceService status
kubectl get inferenceservice -n model-serving

# Describe for detailed conditions
kubectl describe inferenceservice <name> -n model-serving

# Check pod status
kubectl get pods -n model-serving
kubectl logs -n model-serving <pod-name>
```

### GPU Not Detected

```bash
# Verify device plugin is running
kubectl get pods -n kube-system | grep nvidia

# Check plugin logs
kubectl logs -n kube-system -l name=nvidia-device-plugin-ds

# Verify GPU is advertised
kubectl describe node minikube | grep nvidia.com/gpu
```

### Out of Memory Errors

Increase minikube memory allocation:
```bash
minikube stop
minikube config set memory 32768
minikube start --gpus=all
```

---

## Version Compatibility

| minikube | Kubernetes | Cert Manager | Istio | KServe | Status |
|----------|------------|--------------|-------|--------|--------|
| 1.37.x | 1.28-1.32 | v1.16.1 | 1.22.0 | v0.15.0 | Tested |
| 1.36.x | 1.28-1.31 | v1.16.1 | 1.22.0 | v0.15.0 | Untested |
| < 1.36 | < 1.28 | - | - | - | Not supported |

---

## See Also

- [CLI Reference](cli-reference.md) - kfabrik command documentation
- [Architecture](architecture.md) - Detailed design documentation
- [Getting Started](getting-started.md) - Installation guide
