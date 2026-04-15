<p align="center">
  <img src="assets/banner.webp" alt="Orchestration Playbook" width="100%">
</p>

# Orchestration Playbook

> **🐧 中文網頁版** — [penchan.co/ai/orchestration-playbook](https://penchan.co/ai/orchestration-playbook/)

**Battle-tested patterns for running multi-agent AI systems in production.** Not theory. Not a framework. Operational patterns from months of running 5+ agents across multiple models, distilled into reusable practices.

> Most multi-agent resources tell you *what* to build. This tells you *how to keep it running* -- file-based communication, failure recovery, cost control, human escalation, model selection. The boring stuff that makes the difference between a demo and a system.

---

## Who This Is For

- You run Claude Code (or similar) with subagents and want to stop reinventing coordination patterns
- You're building a multi-agent system and want to avoid the failure modes someone else already hit
- You need practical templates, not another framework to install

## What This Is NOT

- Not a framework or library -- no code to install
- Not about agent *topology* (Panel, Tournament, Debate) -- see [multi-agent-patterns](https://github.com/p3nchan/multi-agent-patterns) for that
- Not platform-specific -- patterns work with Claude Code, Codex, Gemini, or custom setups

---

## Table of Contents

<img src="assets/sections/patterns.webp" alt="Patterns" width="100%">

### Patterns

Core operational patterns for multi-agent coordination:

| Pattern | Summary | When You Need It |
|---------|---------|-----------------|
| [File Blackboard](patterns/file-blackboard.md) | Agents communicate through files, not messages | Always -- this is the foundation |
| [Task Envelope](patterns/task-envelope.md) | Structured task packaging for subagents | Every time you spawn a subagent |
| [Challenge Loop](patterns/challenge-loop.md) | Evidence-based adversarial review with anti-sycophancy | Spec reviews, algorithm validation, any iterative refinement |
| [Circuit Breaker](patterns/circuit-breaker.md) | Stop cascading failures before they burn your budget | Any system with retries or external APIs |
| [HITL Escalation](patterns/hitl-escalation.md) | Three-tier human-in-the-loop gating | Any autonomous system with real-world side effects |
| [Structured Error Events](patterns/structured-error-events.md) | Standard format for agent status reporting | Multi-step workflows with dependencies |
| [Checkpoint & Resume](patterns/checkpoint-resume.md) | Survive session crashes without losing progress | Long-running or multi-phase tasks |
| [Dead Letter Queue](patterns/dead-letter-queue.md) | Failed tasks don't disappear -- they wait for retry | Systems that can't afford to drop work |
| [Completion Notification](patterns/completion-notification.md) | Agents must report done/failed, never go silent | Every agent interaction |

<img src="assets/sections/guides.webp" alt="Guides" width="100%">

### Guides

Practical guides for day-to-day operations:

| Guide | Summary |
|-------|---------|
| [Model Selection](guides/model-selection.md) | When to use expensive vs cheap models -- with a decision matrix |
| [Error Handling](guides/error-handling.md) | The 5-layer fallback pyramid and retry-vs-restart decision tree |
| [Cost Control](guides/cost-control.md) | Per-task budgets, context pruning, and anti-patterns that burn money |
| [Code Review](guides/code-review.md) | Cross-family review beats self-review -- here's the evidence |
| [Spec-Driven Development](guides/spec-driven-development.md) | Spec quality is the bottleneck, not review stages |
| [Development Pipeline](guides/development-pipeline.md) | Phase-based workflow for algorithm and code development |
| [Context Management](guides/context-management.md) | Keep subagent prompts lean -- pass paths, not full files |
| [Security Guardrails](guides/security-guardrails.md) | Tool permissions, path restrictions, prompt injection defense |
| [Learning Loop](guides/learning-loop.md) | Turn failures into prevention -- debug KB, error SOP, quarterly audits |

### Templates

Copy-paste starting points:

| Template | Format |
|----------|--------|
| [Task Envelope](templates/task-envelope.md) | Markdown |
| [Task Envelope (lightweight)](templates/task-envelope-lightweight.md) | Markdown (for simple tasks) |
| [Error Event](templates/error-event.md) | Structured status format |
| [Checkpoint](templates/checkpoint.json) | JSON |
| [Decision Card](templates/decision-card.md) | Markdown |
| [Dead Letter](templates/dead-letter.json) | JSON |

---

## Quick Start

### 1. Start with File Blackboard

Your agents need a shared workspace. Pick a directory, establish conventions:

```
workspace/
  status.md          # project state -- only the orchestrator writes this
  tasks/             # task envelopes for subagents
  artifacts/         # subagent outputs
  checkpoints/       # crash recovery snapshots
```

One rule: **Only the orchestrator writes global state.** Subagents write to their designated output locations and report back. The orchestrator decides what to accept.

### 2. Wrap Every Task in an Envelope

Don't just tell a subagent "do X". Give it structure:

```markdown
## Task: analyze-q1-data
- **Goal**: Summarize Q1 revenue trends
- **Acceptance Criteria**: Output contains trend direction, top 3 drivers, confidence level
- **Inputs**: data/q1-revenue.csv (quarterly revenue by product line)
- **Budget**: max 5 tool calls
- **Stop Condition**: If data file is missing or malformed, stop and report
```

### 3. Add a Circuit Breaker

Track failures per tool or API. Three consecutive failures in 60 seconds = stop calling. Wait 5 minutes, try once. This single pattern prevents the most expensive failure mode in multi-agent systems: retry storms.

### 4. Decide Your Escalation Tiers

| Tier | Risk | Action |
|------|------|--------|
| Autonomous | Low, reversible | Agent executes freely |
| Notify | Medium, important milestones | Execute then inform human |
| Gate | High, irreversible | Stop and wait for human approval |

---

## Architecture in 30 Seconds

```
                    ┌─────────────┐
                    │ Orchestrator │  ← owns global state, dispatches tasks,
                    │   (you/AI)   │    validates results, handles failures
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────┴─────┐ ┌───┴───┐ ┌─────┴─────┐
        │ Subagent A │ │  S.B  │ │ Subagent C │
        │ (executor) │ │       │ │ (reviewer) │
        └─────┬──────┘ └───┬───┘ └─────┬──────┘
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────┴──────┐
                    │    Files    │  ← the blackboard: status.md,
                    │ (workspace) │    task envelopes, artifacts, checkpoints
                    └─────────────┘
```

**Key properties:**
- **Hierarchical supervisor**: One orchestrator coordinates everything
- **Stateless subagents**: Receive context, execute, return results, terminate
- **File-based communication**: All state lives in files, survives session crashes
- **No peer-to-peer**: Subagents never talk to each other directly

Why this shape? Because most AI coding tools (Claude Code, Codex, etc.) give you stateless subagents with no inter-agent messaging. The file blackboard is the only reliable coordination mechanism.

---

## Key Insights From Production

1. **Subagents are stateless functions.** Treat them like pure functions: input context, get output, done. Don't expect them to remember anything.

2. **Files are your message bus.** In a world without persistent agent sessions, the filesystem is your only reliable shared state.

3. **Silent failures are the #1 killer.** An agent that fails and says nothing is worse than one that fails loudly. Mandate structured error events.

4. **Retries are not free.** Three retries of an expensive model call can cost more than the original task. Circuit breakers are not optional.

5. **Human escalation is a feature, not a failure.** Target 10-15% escalation rate. Too low = risky autonomy. Too high = human bottleneck.

6. **Context is an operational risk.** Bloated prompts cause hallucinations. Pass file paths and summaries, not full documents.

7. **The minimum viable multi-agent system is smaller than you think.** Two agents with opposing briefs + one judge + a shared workspace covers ~80% of use cases.

---

## Design Principles

| Principle | Meaning |
|-----------|---------|
| **File over wire** | All coordination through files. No message queues, no shared memory, no RPC. |
| **Crash-only design** | Any agent can die at any time. Checkpoints make this a non-event. |
| **Budget-aware by default** | Every task has a cost ceiling. Circuit breakers enforce it automatically. |
| **Escalation over autonomy** | When in doubt, ask the human. The cost of a wrong autonomous decision always exceeds the cost of a 5-minute wait. |
| **Compress, don't accumulate** | Completed phases become summaries. Old context gets pruned. Long sessions get checkpointed and restarted. |

---

## Contributing

PRs welcome for:
- New operational patterns with real production evidence
- Additional templates for specific platforms or languages
- War stories: documented failure modes and how you solved them

Please include **when you hit the problem** and **what you tried first**, not just the final solution.

## Research Basis

The development workflow patterns (Challenge Loop, Code Review, Spec-Driven Development, Development Pipeline) are grounded in a systematic 4-track deep research synthesis conducted in April 2026:

- **Track A**: Quantitative strategy development lifecycle (academic literature from 2018-2026 plus practitioner sources from Two Sigma, D.E. Shaw, Man AHL, Winton, and Jane Street)
- **Track B**: AI-assisted software engineering from 2024-2026 (the SWE-bench ecosystem, MetaGPT, AgentCoder, FunSearch, AlphaCodium, and the MAST failure taxonomy)

Key sources include Bailey et al. on PBO, Marcos Lopez de Prado on AFML, debate work from ICML 2024, code-generation research from ICML 2024, DeepMind's orchestration scaling study, Huang et al. on self-correction at ICLR 2024, and the UC Berkeley MAST taxonomy presented as a NeurIPS 2025 Spotlight.

## License

MIT -- use these patterns however you want.

---

*Built from production experience running a multi-model, multi-agent system with Claude Code, Codex CLI, and Gemini. Every pattern in this repo exists because something broke without it.*
