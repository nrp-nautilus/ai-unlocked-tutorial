# AI Unlocked: Agentic Workflows

**Time:** 01:05-01:30

This section connects persistent Coder workspaces with CLI-based AI agents. The hands-on example uses `opencode`, while Coder provides the persistent workspace model for longer-running development work.

Run all commands from a JupyterHub terminal unless the instructor directs participants to use a local shell. Command blocks are formatted for copy/paste into that terminal.

## Schedule

| Time | Topic | Outcome |
| --- | --- | --- |
| 01:05-01:10 | Coder overview | Understand persistent workspaces and why they help classroom AI workflows. |
| 01:10-01:20 | Agent CLI workflow | Configure an OpenAI-compatible CLI agent for the NRP managed LLM. |
| 01:20-01:30 | Discussion and Q&A | Identify implementation strategies for under-resourced classrooms. |

## 01:05-01:10 - Coder Overview

[Coder on NRP](https://coder.nrp-nautilus.io) provides persistent development workspaces on the same shared infrastructure used by the rest of the tutorial. A Coder workspace can hold:

- source code
- package installations
- editor state
- logs and generated files
- credentials configured by the user

This differs from a short-lived inference pod. A Coder workspace is meant to be a repeatable development environment where an AI agent can work across multiple commands, files, and class sessions.

The high-level workflow is:

1. Log into Coder with institutional credentials.
2. Create a workspace from a template.
3. Connect with `coder ssh` or the browser IDE.
4. Install or launch an AI coding CLI.
5. Point the CLI at the NRP managed LLM endpoint.
6. Review generated changes before committing or sharing them.

Check whether the Coder CLI is available:

```bash
coder version
```

If it is not installed in the JupyterHub image, install it without `sudo`:

```bash
mkdir -p ~/.local/bin
CODER_VERSION=2.31.3
curl -sL "https://github.com/coder/coder/releases/download/v${CODER_VERSION}/coder_${CODER_VERSION}_linux_amd64.tar.gz" -o /tmp/coder.tgz
tar -xzf /tmp/coder.tgz -C /tmp
mv /tmp/coder_${CODER_VERSION}_linux_amd64/coder ~/.local/bin/coder 2>/dev/null || mv /tmp/coder ~/.local/bin/coder
chmod +x ~/.local/bin/coder
export PATH="$HOME/.local/bin:$PATH"
coder version
```

Log into NRP Coder. This step is interactive.

```bash
coder login coder.nrp-nautilus.io
```

For a live 90-minute session, the instructor can either use a pre-created Coder workspace or create one on the spot:

```bash
export PATH="$HOME/.local/bin:$PATH"
export TUTORIAL_USER=<username>
export WORKSPACE=agent-demo-${TUTORIAL_USER}

coder create "$WORKSPACE" -t general-template \
  --parameter '1. Region=Any' \
  --parameter '2. GPUs=0' \
  --parameter '3. CPU Cores=4' \
  --parameter '4. Memory (RAM)=8' \
  --parameter '5. GPU Type=None' \
  --parameter '6. Node=Any' \
  --parameter '7. Image=gitlab-registry.nrp-nautilus.io/nrp/coder-images/ubuntu' \
  --parameter '8. FPGAs=No' \
  -y
```

## 01:10-01:20 - Agent CLI Workflow with opencode

[`opencode`](https://opencode.ai) is the CLI agent example for this section. It can run directly in JupyterHub for a quick demo or inside a Coder workspace for persistence.

Fast path in the JupyterHub terminal:

```bash
cd ~/ai-unlocked-tutorial
curl -fsSL https://opencode.ai/install | bash
export PATH="$HOME/.opencode/bin:$PATH"
opencode --version
```

Write an opencode config that reads the already configured NRP LLM token from the environment:

```bash
python3 - <<'PY'
import json
import os
from pathlib import Path

path = Path.home() / ".config" / "opencode" / "opencode.json"
path.parent.mkdir(parents=True, exist_ok=True)

cfg = {
    "$schema": "https://opencode.ai/config.json",
    "provider": {
        "nrp": {
            "npm": "@ai-sdk/openai-compatible",
            "name": "NRP LLM",
            "options": {
                "baseURL": os.environ.get("OPENAI_API_BASE", "https://ellm.nrp-nautilus.io/v1"),
                "apiKey": "{env:OPENAI_API_KEY}"
            },
            "models": {
                "minimax-m2": {"name": "MiniMax M2"},
                "gpt-oss": {"name": "GPT-OSS"},
                "qwen3": {"name": "Qwen3"},
                "gemma": {"name": "Gemma"}
            }
        }
    },
    "model": "nrp/minimax-m2"
}

path.write_text(json.dumps(cfg, indent=2))
print("wrote", path)
PY
```

Create a small project and launch the agent:

```bash
mkdir -p ~/ai-unlocked-agent-demo
cd ~/ai-unlocked-agent-demo
opencode
```

Inside the opencode prompt, ask for a small, reviewable task:

```text
Write a single-file Python command-line study planner in study_planner.py.
It should accept a topic, number of days, and minutes per day, then print a
simple schedule. Add a README.md with install and run instructions. Keep the
implementation beginner-friendly and do not use external dependencies.
```

After opencode writes files, review before running:

```bash
ls -la
sed -n '1,200p' study_planner.py
python3 study_planner.py --help
```

Persistent path in Coder:

```bash
coder ssh "$WORKSPACE"
```

Then repeat the opencode install and project steps inside the workspace. If the workspace does not inherit `OPENAI_API_KEY`, set it from the class token source or mint a personal token from [https://nrp.ai/llmtoken](https://nrp.ai/llmtoken). Do not paste tokens into committed files.

Key teaching points:

- Agents should work in a controlled repository or workspace, not directly against production systems.
- The user still reviews diffs, runs code, and decides what to commit.
- Persistent workspaces make it possible to pause and resume work across class sessions.
- OpenAI-compatible agent tools can use the NRP managed LLM endpoint without students buying separate model access.

## 01:20-01:30 - Discussion and Q&A

Use the final discussion to connect the mechanics to classroom implementation. For longer-term courses, instructors can move from the shared training environment to a custom JupyterHub with controlled access, custom images, shared storage, and per-user resource profiles.

Suggested prompts:

- What parts of this workflow should be prepared before students arrive?
- Which resources should be shared services, and which should be per-student?
- How should instructors set quotas so one student cannot consume the whole class allocation?
- What should students be allowed to persist after the class ends?
- How will students verify AI-generated code before using it?

Practical classroom strategies:

- Use a prepared JupyterHub image with `kubectl`, `helm`, Python packages, and LLM environment variables already configured.
- Pre-create namespaces, quotas, secrets, and any needed resource exceptions before class.
- Use managed LLMs or shared services for first-time exercises.
- Reserve accelerator-heavy workflows for short, time-boxed demos.
- Give each student a unique username convention for pod and workspace names.
- Include cleanup commands in every activity.
- Prefer reviewable repository workflows over untracked generated files.
- Pair students when accelerator capacity is limited.
- Move to custom JupyterHubs when a course needs controlled enrollment, custom images, shared PVCs, or repeatable per-student profiles.

## Cleanup

If the workspace was created only for the live demo, delete it:

```bash
coder delete "$WORKSPACE" -y
```

If students will continue later, keep the workspace but remind them that deleting a Coder workspace also deletes its associated persistent storage.
