# Getting Started

This guide walks you through setting up KFabrik and deploying your first LLM model.

## Prerequisites

### Required Software

- **Go** 1.21 or later (for building)
- **Make** (for building)
- **Docker** with NVIDIA Container Toolkit (for GPU support)
- **kubectl** (installed automatically with minikube)

### For GPU Support (Linux only)

- NVIDIA GPU with drivers installed on host
- NVIDIA Container Toolkit configured for Docker

### Platform Support

| Platform | GPU Support | Notes |
|----------|-------------|-------|
| Linux with NVIDIA GPU | Full | Default configuration |
| Linux without GPU | CPU-only | Auto-detected |
| macOS | Coming soon | |
| Windows | Coming soon | |

## System Requirements

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

## Installation

KFabrik is distributed as a custom build of minikube. Clone the repository, build, and install.

### Build Requirements

- Go 1.21 or later
- Make
- Docker

### Clone and Build

```bash
# Clone the kfabrik minikube repository
git clone https://github.com/kfabrik/minikube.git
cd minikube

# Build minikube with kfabrik addons
make build

# Install kfabrik CLI and minikube
./scripts/install.sh
```

The install script places both `minikube` and `kfabrik` binaries in `/usr/local/bin`.

## Quick Start

### 1. Start the Cluster

```bash
# Start with GPU support (auto-detected)
kfabrik cluster start

# Or start in CPU-only mode
kfabrik cluster start --cpu-only

# Custom resource allocation
kfabrik cluster start --memory 16384 --cpus 4
```

The `cluster start` command:

1. Starts minikube with Docker driver and GPU passthrough
2. Enables kfabrik-bootstrap addon (installs KServe, Istio, Cert-Manager)
3. Enables kfabrik-model addon (creates model configurations)
4. Enables kfabrik-monitoring addon (deploys Prometheus, Grafana)
5. Deploys the default model (qwen-small)

### 2. List Available Models

```bash
kfabrik list
```

Output:
```
Available Models (all fit in 6GB VRAM):
  NAME         PARAMS  VRAM   DOWNLOAD  DISPLAY NAME
  qwen-small   0.5B    1GB    1GB       Qwen 2.5 0.5B Instruct
  qwen-medium  1.5B    3GB    3GB       Qwen 2.5 1.5B Instruct
  tinyllama    1.1B    2.5GB  2.2GB     TinyLlama 1.1B Chat
  smollm2      1.7B    3.5GB  3.4GB     SmolLM2 1.7B Instruct
  phi2         2.7B    5.5GB  5.5GB     Phi-2 (2.7B)

Deployed Models:
  NAME        READY  URL
  qwen-small  True   http://qwen-small-model-serving.example.com
```

### 3. Deploy a Model

```bash
# Deploy a single model
kfabrik deploy --models qwen-small

# Deploy and wait for readiness
kfabrik deploy --models qwen-medium --wait

# Deploy multiple models
kfabrik deploy --models qwen-small,tinyllama
```

### 4. Check Model Status

```bash
kfabrik status --model qwen-small
```

Output:
```
NAME        READY  REASON  URL
qwen-small  True   Ready   http://qwen-small-model-serving.example.com
```

### 5. Query a Model

```bash
kfabrik query --model qwen-small --prompt "What is Kubernetes?"
```

Output:
```
Model: qwen-small
Response:
Kubernetes is an open-source container orchestration platform...

[Tokens: prompt=34, completion=156, total=190]
```

Query with custom parameters:
```bash
kfabrik query --model qwen-medium \
  --prompt "Explain machine learning in simple terms" \
  --temperature 0.5 \
  --max-tokens 500
```

### 6. View Logs

```bash
# View recent logs
kfabrik logs --model qwen-small

# Follow logs in real-time
kfabrik logs --model qwen-small --follow

# Show last 100 lines
kfabrik logs --model qwen-small --lines 100
```

### 7. Delete Models

```bash
# Delete a specific model
kfabrik delete --model qwen-small

# Delete all models
kfabrik delete --all
```

### 8. Stop the Cluster

```bash
kfabrik cluster stop
```

## Accessing Dashboards

### Grafana

```bash
kubectl port-forward -n monitoring svc/grafana 3000:3000
```
Open http://localhost:3000 (Login: admin / admin)

### Prometheus

```bash
kubectl port-forward -n monitoring svc/prometheus 9090:9090
```
Open http://localhost:9090

## Manual Addon Management

If you prefer to manage addons separately (after building and installing):

```bash
# Start minikube with GPU support
minikube start --driver=docker --memory=32768 --cpus=8 --gpus=all

# Enable core infrastructure
minikube addons enable kfabrik-bootstrap

# Wait for installation to complete
kubectl wait --for=condition=complete job/kfabrik-installer \
  -n kserve --timeout=600s

# Enable model configurations
minikube addons enable kfabrik-model

# Enable monitoring (optional)
minikube addons enable kfabrik-monitoring

# Deploy a model
kfabrik deploy --models qwen-small --wait
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| KUBECONFIG | ~/.kube/config | Kubernetes configuration file |
| KFABRIK_CPU_ONLY | false | Force CPU-only mode |
| KFABRIK_MINIKUBE_BINARY | minikube | Path to minikube binary |
| KFABRIK_KUBECTL_BINARY | kubectl | Path to kubectl binary |

## Troubleshooting

### GPU Not Detected

```bash
# Verify NVIDIA driver on host
nvidia-smi

# Check device plugin logs
kubectl logs -n kube-system -l name=nvidia-device-plugin-ds

# Verify GPU is advertised
kubectl describe node minikube | grep nvidia.com/gpu
```

### Model Fails to Become Ready

```bash
# Check InferenceService status
kfabrik status --model <name>

# Check predictor pod logs
kfabrik logs --model <name>

# Check KServe controller logs
kubectl logs -n kserve -l control-plane=kserve-controller-manager
```

### Out of Memory Errors

Increase minikube memory allocation:

```bash
minikube stop
minikube config set memory 32768
minikube start --gpus=all
```

## Next Steps

- [CLI Reference](cli-reference.md) - Complete command documentation
- [Architecture](architecture.md) - Understand how KFabrik works
- [Addons](addons.md) - Addon configuration and customization
