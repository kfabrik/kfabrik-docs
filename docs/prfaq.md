# KFabrik PRFAQ

---

## PRESS RELEASE

### KFabrik Launches Open-Source Platform for Local LLM Development on Kubernetes

**Minikube addons and CLI enable local LLM development environments in under 10 minutes**

**TAMPA, FL — February 2026** — The KFabrik project today announced the release of its open-source platform for local development and testing of Large Language Models (LLMs) on Kubernetes. KFabrik enables ML developers to deploy LLM inference servers on minikube for development purposes with a single command, eliminating weeks of manual configuration.

**The Problem: Infrastructure Friction Slows ML Development**

ML developers face significant friction when testing LLM deployments locally. The current state requires manual installation and configuration of multiple interdependent components: container runtimes with GPU support, Kubernetes clusters, service mesh infrastructure, model serving frameworks, and monitoring stacks.

Deploying a single model requires creating and maintaining dozens of Kubernetes manifests across multiple namespaces. Developers must understand the intricacies of KServe InferenceServices, Istio VirtualServices, Prometheus scrape configurations, and GPU resource scheduling.

The ML serving stack has strict ordering requirements. Cert-manager must be operational before Istio can create TLS certificates. Istio must be running before KServe can configure ingress routing. NVIDIA device plugins must register GPUs before model pods can request GPU resources. Manual installation frequently fails due to race conditions or missing prerequisites.

"We saw talented ML engineers spending days configuring infrastructure instead of building AI applications," said a KFabrik contributor. "Every project required the same tedious setup: installing Cert-Manager, then Istio, then KServe, then debugging why they don't work together. KFabrik eliminates this entirely."

**The Solution: Integrated ML Inference Platform**

KFabrik solves these problems by providing an opinionated, fully-integrated ML inference platform for local development that deploys with a single command.

The platform consists of three minikube addons and a command-line interface:

- **kfabrik-bootstrap**: Installs Cert-Manager, Istio, KServe, and NVIDIA Device Plugin in the correct dependency order
- **kfabrik-model**: Provides pre-configured LLM definitions optimized for consumer GPUs (6GB VRAM)
- **kfabrik-monitoring**: Deploys Prometheus, Grafana, and GPU metrics with DCGM Exporter
- **kfabrik CLI**: Provides commands for cluster management, model deployment, and inference queries

**How It Works**

Developers run a single command to start a GPU-enabled cluster with the complete ML serving stack:

```bash
kfabrik cluster start
kfabrik deploy --models qwen-small --wait
kfabrik query --model qwen-small --prompt "What is Kubernetes?"
```

A typical deployment takes under 10 minutes. The consistent local environment means all team members work with identical configurations, making it easy to share and reproduce issues.

"KFabrik transformed our team's workflow," said an early adopter. "Junior ML engineers who previously needed weeks of Kubernetes training can now deploy and test models in their first hour. The consistent local setup means everyone on the team has the same development environment."

**Built for GPU-First Workflows**

KFabrik assumes GPU workloads are the primary use case. Resource defaults are tuned for NVIDIA GPUs with 6GB+ VRAM. Five pre-configured models are included:

- **qwen-small** (0.5B parameters, ~1GB VRAM) - Quick testing
- **qwen-medium** (1.5B parameters, ~3GB VRAM) - Balanced performance
- **tinyllama** (1.1B parameters, ~2.5GB VRAM) - Lightweight inference
- **smollm2** (1.7B parameters, ~3.5GB VRAM) - Efficient small model
- **phi2** (2.7B parameters, ~5.5GB VRAM) - Higher quality output

**Design Principles**

KFabrik follows five guiding tenets:

1. **Simplicity over flexibility**: Optimized for the common case of deploying HuggingFace models on GPU-enabled minikube clusters
2. **Consistent environments**: All developers get identical local configurations
3. **Explicit over implicit**: All configuration is visible and auditable as standard Kubernetes resources
4. **Fast feedback**: Parallel installations, aggressive health checks, and clear progress reporting
5. **GPU-first**: Design decisions favor GPU scheduling efficiency

**Availability**

KFabrik is available now under the Apache 2.0 license. It is distributed as a custom build of minikube available at [github.com/kfabrik/minikube](https://github.com/kfabrik/minikube).

- Documentation: [kfabrik.io](https://kfabrik.io)
- Source: [github.com/kfabrik/minikube](https://github.com/kfabrik/minikube)

---

## FREQUENTLY ASKED QUESTIONS

### General Questions

**Q: What is KFabrik?**

A: KFabrik is an integrated platform for deploying and managing Large Language Models (LLMs) on local Kubernetes clusters. It consists of three minikube addons (kfabrik-bootstrap, kfabrik-model, kfabrik-monitoring) and a command-line interface (kfabrik CLI) that together provide a complete ML inference stack optimized for GPU workloads.

**Q: Who is KFabrik for?**

A: KFabrik targets ML developers who need to experiment with and test LLM deployments locally. This includes:

1. **ML Engineers**: Building and testing inference pipelines
2. **Data Scientists**: Experimenting with different models locally
3. **Platform Engineers**: Developing CI/CD pipelines for ML workloads
4. **Developers**: Integrating LLM capabilities into applications

**Q: What problem does KFabrik solve?**

A: KFabrik eliminates the infrastructure friction in local ML development:

- **Configuration burden**: Dozens of Kubernetes manifests reduced to a single command
- **Dependency management**: Correct installation order handled automatically
- **Observability gap**: GPU metrics and model performance monitoring included
- **Reproducibility**: Consistent configurations that work identically across all developer machines

### Technical Questions

**Q: What are the system requirements?**

A: Minimum requirements:

- CPU: 4 cores (8 recommended)
- RAM: 8GB (16-32GB recommended for GPU workloads)
- Disk: 40GB
- GPU: NVIDIA GPU with 6GB+ VRAM (optional but recommended)
- OS: Linux (macOS and Windows support coming soon)

**Q: What components does KFabrik install?**

A: KFabrik installs the following components:

| Component | Purpose |
|-----------|---------|
| Cert-Manager | TLS certificate management |
| Istio | Service mesh and ingress |
| KServe | Model serving platform |
| NVIDIA Device Plugin | GPU resource scheduling |
| Prometheus | Metrics collection |
| Grafana | Visualization |
| DCGM Exporter | GPU metrics |

**Q: How does KFabrik handle GPU support?**

A: KFabrik automatically detects GPU availability and configures the cluster accordingly:

- Linux with NVIDIA GPU: Full GPU acceleration (default)
- Linux without GPU: CPU-only mode (auto-detected)
- macOS: Coming soon
- Windows: Coming soon

The `--cpu-only` flag can force CPU-only mode even when a GPU is available.

**Q: What models are pre-configured?**

A: Five models optimized for consumer GPUs (6GB VRAM):

| Model | Parameters | VRAM | Use Case |
|-------|------------|------|----------|
| qwen-small | 0.5B | ~1GB | Quick testing, low latency |
| qwen-medium | 1.5B | ~3GB | Balanced performance |
| tinyllama | 1.1B | ~2.5GB | Lightweight inference |
| smollm2 | 1.7B | ~3.5GB | Efficient small model |
| phi2 | 2.7B | ~5.5GB | Higher quality, more resources |

**Q: Can I use custom models?**

A: Yes. KFabrik supports any HuggingFace model. Create a custom configuration file specifying the model URI and resource requirements, then deploy with the CLI.

**Q: How does the query command work?**

A: The `kfabrik query` command:

1. Sets up kubectl port-forwarding to the model's predictor service
2. Sends an OpenAI-compatible chat completion request
3. Displays the response with token usage statistics
4. Cleans up the port-forward

**Q: Why does KFabrik use RawDeployment mode instead of Knative?**

A: KServe supports two deployment modes: Serverless (using Knative) and RawDeployment (using standard Kubernetes Deployments). KFabrik currently uses RawDeployment for simplicity—it's easier to set up, debug, and operate for local development.

Knative support is planned for a future release.

### Operations Questions

**Q: How long does deployment take?**

A: A typical deployment takes under 10 minutes:

- Cluster start with addons: ~5-7 minutes
- Model deployment (first time): ~3-5 minutes (includes model download)
- Model deployment (cached): ~1-2 minutes

**Q: How do I troubleshoot deployment issues?**

A: Use the following commands:

```bash
# Check installer progress
kubectl get jobs -n kserve
kubectl logs -n kserve -l job-name=kfabrik-installer

# Check model status
kfabrik status --model <name>
kfabrik logs --model <name>

# Check KServe controller
kubectl logs -n kserve -l control-plane=kserve-controller-manager

# Verify GPU detection
kubectl describe node minikube | grep nvidia.com/gpu
```

**Q: How do I access monitoring dashboards?**

A: Use kubectl port-forward:

```bash
# Grafana (login: admin/admin)
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Prometheus
kubectl port-forward -n monitoring svc/prometheus 9090:9090
```

**Q: Can I use KFabrik for production?**

A: No. KFabrik is designed exclusively for local development and testing. It should not be used for production deployments. For production workloads, use appropriate tooling designed for multi-node clusters, high availability, and enterprise operations.

### Comparison Questions

**Q: How does KFabrik compare to deploying KServe manually?**

A: Manual KServe deployment requires:

1. Installing Cert-Manager (with correct CRD configuration)
2. Installing Istio (with correct IngressClass setup)
3. Installing KServe (with correct deployment mode)
4. Configuring GPU support (device plugins, node labels)
5. Setting up monitoring (Prometheus, Grafana, DCGM)

This process typically takes 2-4 weeks to get right. KFabrik handles all of this in under 10 minutes with tested, compatible configurations.

**Q: How does KFabrik compare to cloud-managed ML services?**

A: Cloud services (SageMaker, Vertex AI, Azure ML) are excellent for production but:

- Create vendor lock-in with proprietary APIs
- Expensive for persistent development environments
- Slow iteration cycles requiring cloud deployment
- Require internet connectivity

KFabrik provides local, open-source environments using the same KServe APIs that work across any Kubernetes cluster.

**Q: Why minikube addons instead of a standalone installer?**

A: Minikube addons provide:

1. **Dependency management**: Addons can declare dependencies on other addons
2. **Lifecycle integration**: Enable/disable with standard minikube commands
3. **GPU support**: Minikube handles GPU passthrough configuration
4. **Familiar tooling**: Developers already know minikube workflows

---

**Document Version:** 2.0
**Last Updated:** February 2026
**Status:** Current
