# CNCF AI Landscape

KFabrik is part of the broader Cloud Native Computing Foundation (CNCF) ecosystem focused on AI and machine learning workloads. This page provides context on related projects and how they work together.

## Related CNCF Projects

### Model Serving & Inference

**[KServe](https://github.com/kserve/kserve)**  
Kubernetes-native model serving platform for machine learning models. Provides serverless inference with autoscaling, canary deployments, and multi-framework support.

**[vLLM](https://github.com/vllm-project/vllm)**  
High-throughput and memory-efficient inference engine for large language models. Optimized for serving LLMs at scale with advanced batching techniques.

### Orchestration & Management

**[Karmada](https://github.com/karmada-io/karmada)**  
Multi-cluster Kubernetes management system that enables workload distribution across multiple clusters, useful for distributed AI training and inference.

**[LLMD](https://github.com/llm-operator/llm-operator)**  
Kubernetes operator for managing large language model deployments and lifecycle operations.

## How KFabrik Fits In

KFabrik complements these projects by:

- **Simplifying deployment** of AI workloads across the CNCF ecosystem
- **Providing abstractions** that work with KServe, vLLM, and other inference engines
- **Enabling multi-cluster** AI workflows through integration with Karmada
- **Democratizing access** to cloud-native AI tools for developers

## Integration Patterns

### With KServe
KFabrik can deploy models that are served by KServe, providing a simplified interface for model deployment and management.

### With vLLM
Use KFabrik to orchestrate vLLM deployments for high-performance LLM serving in local development environments.

### With Karmada
Leverage Karmada's multi-cluster capabilities to distribute KFabrik-managed AI workloads across different regions or cloud providers.

## Learn More

- [CNCF AI Working Group](https://github.com/cncf/tag-app-delivery/tree/main/ai-wg)
- [CNCF Landscape - AI/ML Category](https://landscape.cncf.io/card-mode?category=machine-learning)
- [KServe GitHub Organization](https://github.com/kserve)
