# Getting Started

KFabrik is currently in early development. The software will be released in the coming weeks.

## System Requirements

When released, KFabrik will require:

### Minimum (Basic KServe Development)
- **CPU**: 4 cores (8 recommended)
- **RAM**: 8GB (16GB recommended)
- **Disk**: 20GB free space
- **OS**: Linux, macOS, or Windows with WSL2
- **Software**: Docker or Podman

### Recommended (Ray Clusters or LLM Serving)
- **CPU**: 8+ cores
- **RAM**: 32GB+
- **GPU**: Optional but recommended for LLM inference testing

## Quick Start (Coming Soon)

Once released, getting started will be simple:

```bash
# Install KFabrik
pip install kfabrik

# Bootstrap a complete KServe stack
kfabrik bootstrap --profile kserve-basic

# Verify the stack is running
kfabrik status
```

A complete KServe-based inference environment will be running locally in under 15 minutes.

## What You'll Get

A fully functional cloud-native AI environment including:

- Kubernetes cluster (kind or k3s)
- KServe with Istio/Knative
- Sample model deployment
- Observability tools (optional)

## Stack Profiles

KFabrik will support multiple stack profiles:

- **kserve-basic**: KServe with minimal dependencies for model serving
- **kserve-ray**: KServe + Ray for distributed workloads
- **rag-stack**: KServe + vector database for RAG patterns
- **full-stack**: Complete CNAI stack with all components

## Documentation Development

This documentation site is built with MkDocs. To contribute to the documentation:

### Prerequisites

- Python 3.8+
- pip

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Serve documentation locally
mkdocs serve
```

Visit `http://127.0.0.1:8000` to view the documentation locally.

## Stay Updated

- GitHub: [github.com/kfabrik](https://github.com/kfabrik)
- Documentation: [kfabrik.io](https://kfabrik.io)

Watch the repository for updates on the initial release.
