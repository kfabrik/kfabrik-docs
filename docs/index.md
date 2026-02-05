# KFabrik Documentation

Welcome to KFabrik, an integrated platform for deploying and managing Large Language Models (LLMs) on local Kubernetes clusters.

## What is KFabrik?

KFabrik enables ML developers to deploy production-grade LLM inference servers on minikube with a single command. A typical deployment takes under 10 minutes and requires no manual configuration of Kubernetes resources, service mesh routing, or monitoring infrastructure.

## The Problem We Solve

ML developers face significant friction when testing LLM deployments locally:

- **Configuration Burden**: Deploying a single model requires creating and maintaining dozens of Kubernetes manifests across multiple namespaces
- **Dependency Management**: The ML serving stack has strict ordering requirements that frequently fail due to race conditions
- **Observability Gap**: Understanding model performance requires correlating inference latency metrics with GPU utilization
- **Reproducibility**: Development environments diverge from production when developers take shortcuts

KFabrik solves these problems by providing an opinionated, fully-integrated ML inference platform that deploys with a single command.

## Core Components

KFabrik comprises a CLI and three minikube addons:

### kfabrik CLI

Command-line interface for model deployment, querying, and management. Provides commands for:

- Starting/stopping clusters with GPU support
- Deploying and managing models
- Querying models via OpenAI-compatible API
- Viewing logs and status

### kfabrik-bootstrap Addon

Installs foundational infrastructure:

| Component | Version | Purpose |
|-----------|---------|---------|
| Cert-Manager | v1.16.1 | TLS certificate management |
| Istio | v1.22.0 | Service mesh and ingress routing |
| KServe | v0.15.0 | Model serving platform |
| NVIDIA Device Plugin | v0.14.1 | GPU resource scheduling |

### kfabrik-model Addon

Provides pre-configured model definitions optimized for consumer GPUs (6GB VRAM):

| Model | Parameters | VRAM | Description |
|-------|------------|------|-------------|
| qwen-small | 0.5B | ~1GB | Qwen 2.5 0.5B Instruct |
| qwen-medium | 1.5B | ~3GB | Qwen 2.5 1.5B Instruct |
| tinyllama | 1.1B | ~2.5GB | TinyLlama 1.1B Chat |
| smollm2 | 1.7B | ~3.5GB | SmolLM2 1.7B Instruct |
| phi2 | 2.7B | ~5.5GB | Microsoft Phi-2 |

### kfabrik-monitoring Addon

Deploys observability stack:

| Component | Version | Purpose |
|-----------|---------|---------|
| Prometheus | v2.51.0 | Metrics collection |
| Grafana | v10.4.0 | Visualization |
| DCGM Exporter | v4.5.1 | GPU metrics |

## Quick Start

```bash
# Start cluster with GPU support
kfabrik cluster start

# List available models
kfabrik list

# Deploy a model
kfabrik deploy --models qwen-small --wait

# Query the model
kfabrik query --model qwen-small --prompt "What is Kubernetes?"

# Clean up
kfabrik delete --model qwen-small
kfabrik cluster stop
```

## Design Principles

**Simplicity over flexibility**: Optimized for the common case of deploying HuggingFace models on GPU-enabled minikube clusters.

**Production parity**: Local deployments mirror production configurations using the same KServe InferenceService specifications and Istio routing rules.

**Explicit over implicit**: All configuration is visible and auditable. Standard Kubernetes resources that can be inspected, modified, and versioned.

**Fast feedback**: Deployment speed optimized through parallel installations, aggressive health checks, and clear progress reporting.

**GPU-first**: Assumes GPU workloads are the primary use case. Resource defaults are tuned for NVIDIA GPUs with 6GB+ VRAM.

## Quick Links

- [Getting Started](getting-started.md) - Installation and setup guide
- [CLI Reference](cli-reference.md) - Complete command documentation
- [Architecture](architecture.md) - Detailed design documentation
- [Addons](addons.md) - Addon configuration and troubleshooting
- [Contributing](contributing.md) - How to contribute to the project
