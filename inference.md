# AI Unlocked: Hands-on Inference

**Time:** 00:35-01:05

Participants first call the NRP managed LLM endpoint from the prepared JupyterHub terminal, then deploy a Hugging Face Text Generation Inference server on an allocated NVIDIA A10 GPU.

Run all commands from a JupyterHub terminal. Command blocks are formatted for copy/paste into that terminal.

## Schedule

| Time | Topic | Outcome |
| --- | --- | --- |
| 00:35-00:40 | Setup check | Confirm namespace, username, LLM endpoint, and the TGI manifest. |
| 00:40-00:48 | Managed LLM inference | Call the NRP OpenAI-compatible endpoint from `curl` and Python. |
| 00:48-01:00 | Hugging Face TGI on A10 | Deploy a model server on a requested GPU and call it locally. |
| 01:00-01:05 | Cleanup and extensions | Stop pods and point participants to RAG and Milvus follow-up material. |

## 00:35-00:40 - Setup Check

Use the same username value from `intro.md`.

```bash
cd ~/ai-unlocked-tutorial
export TUTORIAL_USER=<username>
kubectl get pods -n nrp-training-k8s
```

Confirm the managed LLM environment is available:

```bash
python3 - <<'PY'
import os

print("OPENAI_API_BASE =", os.environ.get("OPENAI_API_BASE", "not set"))
print("OPENAI_API_KEY  =", "set" if os.environ.get("OPENAI_API_KEY") else "not set")
PY
```

Before applying a manifest, create a per-user copy in `/tmp` and replace `<username>` so pod names do not collide.

## 00:40-00:48 - Managed LLM Inference

NRP serves open-weight models through an OpenAI-compatible endpoint:

```text
https://ellm.nrp-nautilus.io/v1
```

The available model set can change. Use the model-list command below during the session; this summary is the expected workshop catalog.

| Model | Status | Params | Context | Tools | Reasoning | Vision |
| --- | --- | --- | --- | --- | --- | --- |
| `qwen3` | main | 397B, A17B active | 262K | yes | yes | image, video |
| `qwen3-small` | main | 27B | 262K | yes | yes | image, video |
| `gpt-oss` | main | 120B | 131K | yes | yes | no |
| `gemma` | main | 31B | 262K | yes | yes | image, video |
| `minimax-m2` | main | 230B | 204K | yes | yes | no |
| `qwen3-embedding` | main | 8B | embeddings | no | no | no |
| `glm-4.7` | evaluating | 358B | 202K | yes | yes | no |
| `kimi` | evaluating | 1T MoE | 262K | yes | yes | image, video |
| `olmo` | evaluating | 32B | 64K | yes | no | no |

The training JupyterHub already exports `OPENAI_API_BASE` and `OPENAI_API_KEY`. List the available models:

```bash
curl -s -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  "${OPENAI_API_BASE}/models" | python3 -m json.tool | head -40
```

Send a short chat request:

```bash
curl -s -X POST "${OPENAI_API_BASE}/chat/completions" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "minimax-m2",
    "messages": [
      {"role": "system", "content": "Answer in one sentence."},
      {"role": "user", "content": "What is the National Research Platform?"}
    ]
  }' | python3 -c 'import json,sys; print(json.load(sys.stdin)["choices"][0]["message"]["content"])'
```

Call the same endpoint with the OpenAI Python SDK:

```bash
python3 - <<'PY'
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    base_url=os.environ["OPENAI_API_BASE"],
)

resp = client.chat.completions.create(
    model="minimax-m2",
    messages=[
        {"role": "system", "content": "You are a concise teaching assistant."},
        {"role": "user", "content": "Explain Kubernetes namespaces in two sentences."},
    ],
)

print(resp.choices[0].message.content)
PY
```

Key point: the same OpenAI-style client can target the managed NRP endpoint or a model server you deploy yourself. Only the `base_url` changes.

## 00:48-01:00 - Hugging Face TGI on an Allocated A10

The TGI manifest starts a Hugging Face Text Generation Inference server:

- image: `ghcr.io/huggingface/text-generation-inference:2.1.1`
- model: `HuggingFaceH4/zephyr-7b-beta`
- accelerator request: `nvidia.com/gpu: 1`
- required GPU product: `NVIDIA-A10`
- workshop reservation: `nrp-training=true` plus `nautilus.io/reservation=nrp:NoSchedule`

Create and apply a per-user copy:

```bash
cp yamls/tgi-inference.yaml /tmp/tgi-inference-${TUTORIAL_USER}.yaml
sed -i "s/<username>/${TUTORIAL_USER}/g" /tmp/tgi-inference-${TUTORIAL_USER}.yaml
kubectl apply -n nrp-training-k8s -f /tmp/tgi-inference-${TUTORIAL_USER}.yaml
kubectl get pod -n nrp-training-k8s tutorial-${TUTORIAL_USER}-tgi -w
```

Wait for the server logs to show that the model is connected:

```bash
kubectl logs -n nrp-training-k8s tutorial-${TUTORIAL_USER}-tgi --tail=80
```

Forward the TGI server port in a separate terminal and leave it running:

```bash
kubectl port-forward -n nrp-training-k8s pod/tutorial-${TUTORIAL_USER}-tgi 8080:80
```

From another JupyterHub terminal, test the TGI native API:

```bash
curl -s http://127.0.0.1:8080/info | python3 -m json.tool | head
curl -s http://127.0.0.1:8080/generate \
  -H "Content-Type: application/json" \
  -d '{"inputs":"Give two practical reasons a classroom might use a shared GPU cluster.","parameters":{"max_new_tokens":60}}'
```

Then point OpenAI-style Python code at the local server:

```bash
python3 - <<'PY'
from openai import OpenAI

client = OpenAI(api_key="not-needed", base_url="http://127.0.0.1:8080/v1")

resp = client.chat.completions.create(
    model="tgi",
    messages=[{"role": "user", "content": "Hi from my own GPU-backed model server."}],
)

print(resp.choices[0].message.content)
PY
```

Discussion points:

- What part of the YAML requested the GPU?
- What part constrained the workload to A10 nodes?
- Why did the local server require port-forwarding?
- What changed between the managed LLM client and the self-hosted TGI client?

## 01:00-01:05 - Cleanup and Extensions

Delete the TGI pod:

```bash
if [ -f /tmp/tgi-inference-${TUTORIAL_USER}.yaml ]; then
  kubectl delete -n nrp-training-k8s -f /tmp/tgi-inference-${TUTORIAL_USER}.yaml --ignore-not-found
fi
```

Stop any `kubectl port-forward` process you started, then confirm nothing from this section is still running:

```bash
kubectl get pods -n nrp-training-k8s
```

Optional extensions for a longer class:

- managed LLM examples with `curl` and Python
- `opencode` against the NRP managed LLM
- PyTorch GPU sanity check and one-epoch MNIST
- RAG over NRP docs using Milvus, managed LLMs, and local Ollama

## Takeaways

- The managed NRP LLM endpoint is the fastest classroom path because students do not launch model pods.
- Hugging Face models can still be deployed directly when the lesson requires runtime control.
- OpenAI-compatible APIs make the same client code portable across managed and self-hosted inference.
- Cleanup is part of the lesson whenever a workload reserves GPUs.
