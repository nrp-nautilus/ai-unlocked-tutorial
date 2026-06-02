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

## 📋 Pre-training survey — please take 2 minutes before we start

<a href="images/pre-training-survey-qr.png"><img src="images/pre-training-survey-qr.png" alt="Pre-training survey QR code" width="180" align="right"></a>

Scan the QR on the right (or open [`https://ucsantacruz.co1.qualtrics.com/jfe/form/SV_3wQP0UrsPXy3nMO?Q_CHL=qr`](https://ucsantacruz.co1.qualtrics.com/jfe/form/SV_3wQP0UrsPXy3nMO?Q_CHL=qr)) to take the **pre-training survey**. It's a quick set of questions about your prior Kubernetes / NRP / AI experience and what you hope to get out of the session.

Comparing pre- and post-training responses is how we measure whether these materials actually move the needle, and what to keep, cut, or rework for future cohorts. The more responses we get, the better the next group's experience will be.

<br clear="right">

## 90-Minute Path

The materials are numbered in the order you work through them:

1. [`1_intro.md`](1_intro.md) — Introduction, Access, and Resource Requests
2. [`2_inference.ipynb`](2_inference.ipynb) — Hands-on Inference + RAG (open in JupyterLab)
3. [`3_agentic.md`](3_agentic.md) — Agentic Workflows

The hands-on inference work runs entirely inside `2_inference.ipynb`: the NRP
managed LLM, a local LLM via Ollama on the JupyterHub session GPU, and two RAG
pipelines (a simple hand-written corpus and the full NRP documentation).
Everything runs **inside the notebook** — no pods to launch and no YAML to
apply — and the managed-LLM and RAG cells work on a CPU-only session. Spawn
the session with **1 × NVIDIA-A10** only if you also want to run the local-LLM
comparison cells. YAML equivalents for every step are kept inside the notebook
as collapsible reveals.

## Reference Materials

1. [NRP and Kubernetes for Education and Research](1_nrp_kubernetes_education_research/nrp_kubernetes_education_research.md)
2. [Using AI and LLM Inference on NRP](2_ai_llm_inference_on_nrp/ai_llm_inference_on_nrp.md)
3. [Setting Up Custom JupyterHubs for Classroom and Research](3_custom_jupyterhubs_classroom_research/custom_jupyterhubs_classroom_research.md)

## Conventions

Hands-on examples use the **`nrp-training-k8s`** namespace and the workshop reservation pattern where applicable. In YAML files and commands, replace **`<username>`** with a unique lower-case username to avoid name collisions.
