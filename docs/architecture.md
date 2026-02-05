# Architecture

This document describes the architecture, component interactions, and operational characteristics of the KFabrik platform.

## Overview

KFabrik consists of four primary components that work together to provide a complete ML inference platform:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         User Workstation                                │
│                                                                          │
│   ┌─────────────┐                                                       │
│   │   kfabrik   │  CLI for model deployment, querying, and management  │
│   │    CLI      │                                                       │
│   └──────┬──────┘                                                       │
│          │                                                               │
│          │ kubectl / kubernetes API                                      │
│          ▼                                                               │
├─────────────────────────────────────────────────────────────────────────┤
│                     Minikube Cluster (Docker Driver)                    │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                    kfabrik-bootstrap addon                      │   │
│   │                                                                  │   │
│   │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │   │
│   │   │ Cert-Manager │  │    Istio     │  │   KServe     │         │   │
│   │   │   v1.16.1    │  │   v1.22.0    │  │   v0.15.0    │         │   │
│   │   └──────────────┘  └──────────────┘  └──────────────┘         │   │
│   │                                                                  │   │
│   │   ┌──────────────────────────────────────────────────┐         │   │
│   │   │           NVIDIA Device Plugin v0.14.1           │         │   │
│   │   └──────────────────────────────────────────────────┘         │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                    kfabrik-model addon                          │   │
│   │                                                                  │   │
│   │   ┌──────────────────────────────────────────────────┐         │   │
│   │   │              model-serving namespace             │         │   │
│   │   │                                                  │         │   │
│   │   │   ConfigMap: model-config (5 pre-configured)    │         │   │
│   │   │   InferenceServices: user-deployed models       │         │   │
│   │   │   Predictor Pods: model inference containers    │         │   │
│   │   └──────────────────────────────────────────────────┘         │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                  kfabrik-monitoring addon                       │   │
│   │                                                                  │
│   │   ┌────────────┐  ┌────────────┐  ┌─────────────────┐          │   │
│   │   │ Prometheus │  │  Grafana   │  │  DCGM Exporter  │          │   │
│   │   │  v2.51.0   │  │  v10.4.0   │  │    v4.5.1       │          │   │
│   │   └────────────┘  └────────────┘  └─────────────────┘          │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

### kfabrik CLI

Provides a command-line interface for deploying models, querying inference endpoints, and managing the model lifecycle. The CLI communicates with the Kubernetes API to create InferenceService resources and uses kubectl port-forwarding to access inference endpoints.

**Kubernetes Client Architecture:**

- **Typed Clientset** (`kubernetes.Clientset`) for standard Kubernetes resources (ConfigMaps, Pods, Services)
- **Dynamic Client** (`dynamic.Interface`) for custom resources (InferenceServices)

### kfabrik-bootstrap Addon

Installs the foundational infrastructure required for model serving:

- **Cert-Manager** for TLS certificate management
- **Istio** for service mesh routing
- **KServe** for model lifecycle management
- **NVIDIA Device Plugin** for GPU resource scheduling

The addon uses a Kubernetes Job to orchestrate Helm-based installations in dependency order.

### kfabrik-model Addon

Creates the model-serving namespace and deploys a ConfigMap containing pre-configured model definitions. These definitions specify HuggingFace model URIs, resource requirements, and inference server parameters optimized for 6GB VRAM GPUs.

### kfabrik-monitoring Addon

Deploys the observability stack:

- **Prometheus** for metrics collection
- **Grafana** for visualization
- **DCGM Exporter** for GPU-specific metrics

Prometheus is pre-configured to scrape KServe inference metrics and DCGM GPU metrics.

## Bootstrap Installation Sequence

The bootstrap addon installs components through a sequenced process with explicit health checks:

```
Phase 1: Cert-Manager Installation (timeout: 600s)
├── Add Jetstack Helm repository
├── Helm install cert-manager with CRDs enabled
├── Wait for cert-manager deployment (3 replicas)
├── Wait for cert-manager-webhook deployment
├── Wait for cert-manager-cainjector deployment
└── Verify certificates.cert-manager.io CRD exists

Phase 2: Istio Installation (timeout: 600s)
├── Add Istio Helm repository
├── Helm install istio-base (CRDs and cluster resources)
├── Helm install istiod (control plane)
├── Wait for istiod deployment
├── Verify gateways.networking.istio.io CRD exists
└── Create istio IngressClass resource

Phase 3: KServe Cleanup (idempotency preparation)
├── Delete existing ClusterRoles
├── Delete existing ClusterRoleBindings
├── Delete existing ValidatingWebhookConfigurations
├── Delete existing MutatingWebhookConfigurations
├── Delete existing CRDs
└── Wait 5 seconds for cleanup completion

Phase 4: KServe Installation (timeout: 600s)
├── Helm install kserve-crd from OCI registry
├── Helm install kserve controller with RawDeployment mode
├── Wait for kserve-controller-manager deployment
└── Verify inferenceservices.serving.kserve.io CRD exists

Phase 5: GPU Node Labeling
├── Query nodes for nvidia.com/gpu capacity
├── Label GPU nodes with nvidia.com/gpu.present=true
└── Log warning if no GPU nodes found
```

## Design Decisions

### Why RawDeployment Mode?

KServe supports two deployment modes: Serverless (using Knative) and RawDeployment (using standard Kubernetes Deployments). KFabrik currently uses RawDeployment for simplicity—it's easier to set up, debug, and operate for local development.

Knative support is planned for a future release.

### Why Sequential Installation?

Parallel installation would be faster but creates race conditions. Istio requires Cert-Manager for TLS certificates. KServe requires Istio for ingress routing. The sequential approach adds approximately 2 minutes to installation time but eliminates entire categories of failure modes.

### Why Cleanup Before KServe Installation?

KServe's webhook configurations and CRDs can become orphaned when previous installations fail mid-process. The cleanup phase ensures idempotent behavior: running the installer multiple times produces the same result.

## Model Deployment Flow

When a user runs `kfabrik deploy --models qwen-small`:

```
1. CLI reads model-config ConfigMap from model-serving namespace
2. CLI parses YAML and extracts qwen-small configuration
3. CLI applies default resource values for any unspecified fields
4. CLI checks if InferenceService qwen-small already exists
   └── If exists: skip deployment (idempotent)
   └── If not exists: continue
5. CLI creates InferenceService resource via Kubernetes API
6. KServe controller watches InferenceService creation
7. Controller creates:
   ├── Predictor Deployment (model inference container)
   ├── Predictor Service (internal ClusterIP)
   ├── Istio VirtualService (ingress routing)
   └── Istio DestinationRule (traffic management)
8. Predictor pod starts:
   ├── Downloads model from HuggingFace (storageUri)
   ├── Initializes inference server (vLLM/TGI)
   └── Exposes prediction endpoint on port 80
9. Pod readiness probe succeeds
10. InferenceService status updated to Ready=True
11. CLI detects Ready status (if --wait flag specified)
```

## InferenceService Specification

The CLI generates InferenceService resources with the following structure:

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: qwen-small
  namespace: model-serving
spec:
  predictor:
    model:
      modelFormat:
        name: huggingface
      storageUri: hf://Qwen/Qwen2.5-0.5B-Instruct
      resources:
        requests:
          cpu: "1"
          memory: "2Gi"
          nvidia.com/gpu: "1"
        limits:
          cpu: "2"
          memory: "4Gi"
          nvidia.com/gpu: "1"
      env:
        - name: HF_MODEL_ID
          value: "Qwen/Qwen2.5-0.5B-Instruct"
        - name: MAX_MODEL_LEN
          value: "2048"
        - name: DTYPE
          value: "float16"
```

## Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      monitoring namespace                           │
│                                                                      │
│  ┌─────────────────┐      scrape       ┌─────────────────────────┐ │
│  │   Prometheus    │◄──────────────────│    DCGM Exporter        │ │
│  │                 │      :9400        │    (DaemonSet)          │ │
│  │  • Scrape jobs  │                   │    GPU nodes only       │ │
│  │  • Alert rules  │                   └─────────────────────────┘ │
│  │  • TSDB storage │                                               │
│  └────────┬────────┘                                               │
│           │                                                         │
│           │ query :9090                                             │
│           ▼                                                         │
│  ┌─────────────────┐                                               │
│  │    Grafana      │                                               │
│  │                 │                                               │
│  │  • Dashboards   │                                               │
│  │  • Data sources │                                               │
│  │  • Alerting     │                                               │
│  └─────────────────┘                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
            ▲
            │ scrape
            │
┌───────────┴─────────────────────────────────────────────────────────┐
│                    model-serving namespace                          │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │
│  │  qwen-small     │  │  qwen-medium    │  │    phi2         │    │
│  │  predictor      │  │  predictor      │  │  predictor      │    │
│  │    :9090        │  │    :9090        │  │    :9090        │    │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prometheus Scrape Jobs

| Scrape Job | Targets | Metrics |
|------------|---------|---------|
| prometheus | Self (localhost:9090) | Prometheus internal metrics |
| kubernetes-apiservers | API server | API request latency, etcd metrics |
| kubernetes-nodes | All nodes | Node CPU, memory, disk |
| kubernetes-nodes-cadvisor | All nodes | Container resource usage |
| kubernetes-service-endpoints | Annotated services | Custom service metrics |
| kubernetes-pods | Annotated pods | Custom pod metrics |
| dcgm-exporter | GPU node pods | GPU utilization, temperature, memory |
| kserve-inferenceservices | Model predictor pods | Inference latency, throughput |

### GPU Metrics (DCGM Exporter)

| Metric | Type | Description |
|--------|------|-------------|
| DCGM_FI_DEV_GPU_UTIL | gauge | GPU utilization percentage |
| DCGM_FI_DEV_MEM_COPY_UTIL | gauge | Memory copy engine utilization |
| DCGM_FI_DEV_FB_FREE | gauge | Free framebuffer memory (MiB) |
| DCGM_FI_DEV_FB_USED | gauge | Used framebuffer memory (MiB) |
| DCGM_FI_DEV_FB_TOTAL | gauge | Total framebuffer memory (MiB) |
| DCGM_FI_DEV_GPU_TEMP | gauge | GPU temperature (Celsius) |
| DCGM_FI_DEV_MEMORY_TEMP | gauge | Memory temperature (Celsius) |
| DCGM_FI_DEV_POWER_USAGE | gauge | Power consumption (Watts) |

## Security Considerations

### Privilege Requirements

The kfabrik-bootstrap installer Job requires cluster-admin privileges for:

- Creating CustomResourceDefinitions (cluster-scoped)
- Creating ClusterRoles and ClusterRoleBindings
- Creating ValidatingWebhookConfigurations
- Installing Helm charts that modify cluster-wide resources

**Mitigations:**

- Installer Job has TTL of 300 seconds (auto-deleted after completion)
- ServiceAccount is scoped to kserve namespace
- Installer runs once during addon enablement, not continuously

### Network Exposure

By default, KFabrik does not expose services outside the cluster. All access occurs through:

- `kubectl port-forward` for CLI queries
- `minikube service` for Grafana/Prometheus dashboards

## Dependencies

### External Dependencies

| Component | Version | Source | Purpose |
|-----------|---------|--------|---------|
| Cert-Manager | v1.16.1 | Jetstack Helm | TLS certificate management |
| Istio | v1.22.0 | Istio Helm | Service mesh, ingress |
| KServe | v0.15.0 | KServe OCI | Model serving platform |
| NVIDIA Device Plugin | v0.14.1 | NVIDIA | GPU resource scheduling |
| Prometheus | v2.51.0 | Docker Hub | Metrics collection |
| Grafana | v10.4.0 | Docker Hub | Visualization |
| DCGM Exporter | v4.5.1 | NVIDIA | GPU metrics |

### Runtime Dependencies

| Dependency | Required By | Purpose |
|------------|-------------|---------|
| minikube | kfabrik CLI | Cluster management |
| kubectl | kfabrik CLI | Port-forwarding, API access |
| Docker | minikube | Container runtime |
| NVIDIA Driver | Host | GPU access |
| NVIDIA Container Toolkit | minikube | GPU passthrough |

## Resource Requirements

### Default Resource Requirements

| Component | CPU Request | CPU Limit | Memory Request | Memory Limit |
|-----------|-------------|-----------|----------------|--------------|
| KServe Controller | 100m | 500m | 256Mi | 512Mi |
| Prometheus | 250m | 500m | 512Mi | 1Gi |
| Grafana | 100m | 200m | 256Mi | 512Mi |
| DCGM Exporter | 100m | 500m | 512Mi | 1Gi |
| Model Predictor (small) | 1 | 2 | 2Gi | 4Gi |
| Model Predictor (medium) | 2 | 4 | 6Gi | 10Gi |

### Port Assignments

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| Prometheus | 9090 | HTTP | Metrics API, Web UI |
| Grafana | 3000 | HTTP | Dashboard UI |
| DCGM Exporter | 9400 | HTTP | GPU metrics endpoint |
| Model Predictor | 80 | HTTP | Inference API |
| Model Predictor Metrics | 9090 | HTTP | Prometheus metrics |
