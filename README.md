# AI Unlocked: AI-Enabled Education and Research on the National Research Platform

This repository contains the AI Unlocked tutorial materials for using NRP-hosted JupyterHub, Kubernetes, GPU resources, managed LLM inference, agentic CLI tools, and custom classroom JupyterHubs.

## Quick Start

To automatically clone this repository into the pre-authenticated JupyterLab environment on NRP, use:

**[Launch AI Unlocked Tutorial Workspace](https://training.nrp-nautilus.io/hub/user-redirect/git-pull?repo=https%3A%2F%2Fgithub.com%2Fnrp-nautilus%2Fai-unlocked-tutorial&branch=main&urlpath=lab%2Ftree%2Fai-unlocked-tutorial%2F)**

The working directory inside JupyterLab is:

```bash
cd ~/ai-unlocked-tutorial
```

Run tutorial commands from the JupyterHub terminal. The markdown command blocks are intended to be copied and pasted into that terminal.

## 90-Minute Path

1. [Introduction, Access, and Resource Requests](intro.md)
2. [Hands-on Inference](inference.md)
3. [Agentic Workflows](agentic.md)

## Reference Materials

1. [NRP and Kubernetes for Education and Research](1_nrp_kubernetes_education_research/nrp_kubernetes_education_research.md)
2. [Using AI and LLM Inference on NRP](2_ai_llm_inference_on_nrp/ai_llm_inference_on_nrp.md)
3. [Setting Up Custom JupyterHubs for Classroom and Research](3_custom_jupyterhubs_classroom_research/custom_jupyterhubs_classroom_research.md)

## Conventions

Hands-on examples use the **`nrp-training-k8s`** namespace and the workshop reservation pattern where applicable. In YAML files and commands, replace **`<username>`** with a unique lower-case username to avoid name collisions.
