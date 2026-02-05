# CLI Reference

Complete reference for the kfabrik command-line interface.

## Synopsis

```
kfabrik [GLOBAL OPTIONS] command [COMMAND OPTIONS]
```

## Global Options

| Option | Description |
|--------|-------------|
| `-h, --help` | Show help message and exit |
| `--kubeconfig FILE` | Path to kubeconfig file (default: $KUBECONFIG or ~/.kube/config) |
| `-n, --namespace NAMESPACE` | Kubernetes namespace for model deployments (default: model-serving) |

## Commands

### cluster

Manage the minikube cluster lifecycle with GPU or CPU-only support.

#### cluster start

Start a new minikube cluster configured for ML workloads.

```bash
kfabrik cluster start [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--cpu-only` | Force CPU-only mode, even if GPU is available |
| `--driver NAME` | Minikube driver to use (default: auto-detect) |
| `--memory MB` | Memory to allocate in megabytes (default: 32768) |
| `--cpus N` | Number of CPUs to allocate (default: 8) |
| `--skip-model` | Skip deploying the default model |
| `--model NAME` | Model to deploy (default: qwen-small) |

**Examples:**

```bash
# Start with GPU (auto-detected)
kfabrik cluster start

# Start in CPU-only mode (works on macOS)
kfabrik cluster start --cpu-only

# Start with custom resources
kfabrik cluster start --memory 16384 --cpus 4

# Start without deploying a model
kfabrik cluster start --skip-model
```

#### cluster stop

Stop and delete the minikube cluster, cleaning up all resources.

```bash
kfabrik cluster stop
```

---

### deploy

Deploy one or more LLM models to Kubernetes using KServe InferenceServices.

```bash
kfabrik deploy [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--models MODEL[,MODEL...]` | Comma-separated list of models to deploy |
| `--all` | Deploy all configured models |
| `--wait` | Wait for models to become ready before returning |

**Examples:**

```bash
# Deploy a single model
kfabrik deploy --models qwen-small

# Deploy multiple models
kfabrik deploy --models qwen-small,tinyllama

# Deploy and wait for readiness
kfabrik deploy --models qwen-medium --wait
```

---

### delete

Delete one or more deployed models from Kubernetes.

```bash
kfabrik delete [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--model NAME` | Name of the model to delete |
| `--all` | Delete all deployed models |

**Examples:**

```bash
# Delete a specific model
kfabrik delete --model qwen-small

# Delete all models
kfabrik delete --all
```

---

### list

List models available for deployment or currently deployed.

```bash
kfabrik list [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--available` | List available models from configuration |
| `--deployed` | List currently deployed models with their status |

Without flags, lists both available and deployed models.

**Example Output:**

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

---

### status

Check the status of deployed models.

```bash
kfabrik status [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--model NAME` | Name of the model to check |
| `--all` | Check status of all deployed models |
| `-o, --output FORMAT` | Output format: table (default) or json |

**Examples:**

```bash
# Check specific model
kfabrik status --model qwen-small

# Check all models as JSON
kfabrik status --all --output json
```

---

### query

Send an inference query to a deployed model using the OpenAI-compatible chat completions API.

```bash
kfabrik query [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--model NAME` | Name of the model to query (required) |
| `--prompt TEXT` | Prompt to send to the model (required) |
| `--temperature FLOAT` | Sampling temperature 0.0-2.0 (default: 0.7) |
| `--max-tokens INT` | Maximum tokens to generate (default: 256) |
| `--top-p FLOAT` | Top-p sampling parameter 0.0-1.0 (default: 0.9) |
| `--timeout SECONDS` | Request timeout in seconds (default: 120) |

The command automatically sets up port-forwarding to the model's predictor service.

**Examples:**

```bash
# Basic query
kfabrik query --model qwen-small --prompt "What is Kubernetes?"

# Query with custom parameters
kfabrik query --model qwen-medium \
  --prompt "Explain machine learning in simple terms" \
  --temperature 0.5 \
  --max-tokens 500
```

**Example Output:**

```
Model: qwen-small
Response:
Kubernetes is an open-source container orchestration platform...

[Tokens: prompt=34, completion=156, total=190]
```

---

### logs

View logs for a deployed model's pods.

```bash
kfabrik logs [FLAGS]
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--model NAME` | Name of the model (required) |
| `-f, --follow` | Stream log output (like tail -f) |
| `--lines INT` | Number of lines to show (default: 50) |
| `-c, --container NAME` | Container name (default: first container) |

**Examples:**

```bash
# View recent logs
kfabrik logs --model qwen-small

# Follow logs in real-time
kfabrik logs --model qwen-small --follow

# Show last 100 lines
kfabrik logs --model qwen-small --lines 100
```

---

### version

Print the version of kfabrik.

```bash
kfabrik version
```

## Available Models

The following models are pre-configured and optimized for consumer GPUs with 6GB VRAM or less:

| Name | Parameters | VRAM | Download | Description |
|------|------------|------|----------|-------------|
| qwen-small | 0.5B | ~1GB | ~1GB | Qwen 2.5 0.5B Instruct |
| qwen-medium | 1.5B | ~3GB | ~3GB | Qwen 2.5 1.5B Instruct |
| tinyllama | 1.1B | ~2.5GB | ~2.2GB | TinyLlama 1.1B Chat |
| smollm2 | 1.7B | ~3.5GB | ~3.4GB | SmolLM2 1.7B Instruct |
| phi2 | 2.7B | ~5.5GB | ~5.5GB | Microsoft Phi-2 |

Models are deployed one at a time; you don't need VRAM for all models simultaneously. System RAM requirements are approximately 2-3x the model size for inference server overhead.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| KUBECONFIG | ~/.kube/config | Kubernetes configuration file |
| KFABRIK_CPU_ONLY | false | Force CPU-only mode |
| KFABRIK_MINIKUBE_BINARY | minikube | Path to minikube binary |
| KFABRIK_KUBECTL_BINARY | kubectl | Path to kubectl binary |

## Exit Status

| Code | Description |
|------|-------------|
| 0 | Success |
| 1 | General error (invalid arguments, connection failed, etc.) |

## See Also

- [Getting Started](getting-started.md) - Installation and setup
- [Addons](addons.md) - Addon configuration
- [Architecture](architecture.md) - How KFabrik works
