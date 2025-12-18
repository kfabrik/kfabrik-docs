# KFabrik Documentation

Welcome to KFabrik, an open-source developer and CI tool that eliminates the complexity barrier in cloud-native AI development.

## What is KFabrik?

KFabrik enables developers to spin up complete, production-like CNCF AI stacks—including Kubernetes, KServe, Ray, vLLM, and vector databases—on their laptops or in CI pipelines in minutes instead of weeks.

## The Problem We Solve

Today's cloud-native AI platforms are a fragmented patchwork of specialized tools. While production-grade technologies like KServe, Ray, and vLLM exist, integrating them into a cohesive stack creates massive complexity. Organizations spend weeks wiring together Kubernetes, service meshes, model servers, vector databases, and observability tools—only to find that versions conflict, configurations break, and "it works on my laptop" becomes "it fails in CI."

## Our Solution

KFabrik provides pre-integrated, tested cloud-native AI stacks with:

- **Unified Integration**: Compatible versions of KServe, Ray, vLLM, and vector databases that work together out-of-the-box
- **Fast Setup**: Complete CNAI environments in 15-30 minutes instead of weeks
- **Reproducible Environments**: Identical configurations across dev, CI, and production-like environments
- **Opinionated Defaults**: Production-ready configurations based on CNCF best practices
- **Full Customization**: Every aspect is configurable through declarative configuration files

## Core Components

KFabrik comprises three core components:

- **kfabric-bootstrap**: Bootstraps complete, integrated CNAI stacks with a single command
- **kfabric-runtime**: Orchestrates runtime wiring, health checks, and service discovery across components
- **kfabric-control**: Provides declarative APIs for stack management and automation

## Project Status

KFabrik is currently in early development. The software components (kfabric-bootstrap, kfabric-runtime, kfabric-control) will be released in the coming weeks. This documentation site is being prepared to support the upcoming launch.

## Quick Links

- [Getting Started](getting-started.md) - Setup and installation guide (coming soon)
- [PRFAQ](prfaq.md) - Detailed press release and FAQ
- [Contributing](contributing.md) - How to contribute to the project

## Built for the CNCF Ecosystem

KFabrik is designed as a potential CNCF project, directly addressing the integration and cohesion gap identified in the CNCF AI landscape. Built on cloud-native principles with declarative configuration, reproducible builds, secure defaults, and zero vendor lock-in.
