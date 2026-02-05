# KFabrik Documentation

Documentation for KFabrik, an integrated platform for deploying and managing Large Language Models (LLMs) on local Kubernetes clusters.

## About KFabrik

KFabrik enables ML developers to deploy production-grade LLM inference servers on minikube with a single command. A typical deployment takes under 10 minutes and requires no manual configuration of Kubernetes resources, service mesh routing, or monitoring infrastructure.

The platform consists of:

- **kfabrik CLI** - Command-line tool for cluster management, model deployment, and inference queries
- **kfabrik-bootstrap** - Minikube addon that installs Cert-Manager, Istio, KServe, and NVIDIA Device Plugin
- **kfabrik-model** - Minikube addon with pre-configured LLM definitions optimized for 6GB VRAM GPUs
- **kfabrik-monitoring** - Minikube addon that deploys Prometheus, Grafana, and GPU metrics

## Quick Start

```bash
kfabrik cluster start
kfabrik deploy --models qwen-small --wait
kfabrik query --model qwen-small --prompt "What is Kubernetes?"
```

## Documentation Development

### Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Local Development

```bash
mkdocs serve
```

Visit http://127.0.0.1:8000

### Deployment

Documentation is automatically deployed to GitHub Pages on push to main branch.

## Documentation Structure

- `docs/index.md` - Home page and overview
- `docs/getting-started.md` - Installation and quick start guide
- `docs/cli-reference.md` - Complete CLI command reference
- `docs/architecture.md` - Technical architecture documentation
- `docs/addons.md` - Addon configuration and troubleshooting
- `docs/prfaq.md` - Press release and FAQ
- `docs/contributing.md` - Contribution guidelines

## Contributing

See [Contributing Guide](docs/contributing.md) for documentation guidelines.

## License

Apache License 2.0
