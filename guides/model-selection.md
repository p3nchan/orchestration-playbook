# Model Selection Guide

> Use the most capable model for judgment. Use the cheapest model for execution. Never use an expensive model for something a cheap one handles fine.

---

## The Two-Tier Rule

| Task Type | Model Tier | Examples |
|-----------|-----------|---------|
| **Thinking** -- strategy, architecture, analysis, quality review | Expensive (Opus, GPT-4+, Gemini Pro) | Design review, risk analysis, synthesizing expert opinions |
| **Executing** -- code, data collection, formatting, tool operation | Cheap (Sonnet, GPT-4o-mini, Gemini Flash) | Writing code, running tests, reformatting data, web scraping |

This single distinction drives most model selection decisions. When in doubt, ask: "Is this agent making a judgment call, or following instructions?"

## Decision Matrix

| Question | If Yes → | If No → |
|----------|---------|---------|
| Does the task require novel reasoning or strategy? | Expensive | Cheap |
| Would a wrong answer be costly or irreversible? | Expensive | Cheap |
| Is the task mostly mechanical (format, transform, fetch)? | Cheap | Consider expensive |
| Is this a quality review of another agent's output? | Expensive | -- |
| Can the output be objectively validated (tests pass, format correct)? | Cheap (validate after) | Expensive (harder to catch errors) |
| Is this a brainstorm or exploration? | Expensive | -- |

## Common Assignments

| Role | Model Tier | Rationale |
|------|-----------|-----------|
| Orchestrator | Expensive | Needs global context, judgment, error recovery |
| Code writer | Cheap | Output is objectively testable |
| Code reviewer | Expensive | Needs to catch subtle bugs and design issues |
| Data collector | Cheap | Mechanical fetch + format operations |
| Data analyst | Expensive | Interpretation, pattern recognition, judgment |
| Test writer | Cheap | Follows patterns, output is verifiable |
| Architecture designer | Expensive | High-stakes decisions, needs broad reasoning |
| Formatter / converter | Cheapest available | Pure transformation, no judgment |
| Security auditor | Expensive | Needs to catch what others miss |

## Cost Comparison (Approximate)

These are approximate as of early 2026 and will change:

| Tier | Examples | Relative Cost | Typical Use |
|------|---------|:---:|------------|
| Premium | Claude Opus, GPT-4+ | $$$$ | Orchestration, strategy, review |
| Standard | Claude Sonnet, GPT-4o | $$ | General coding, analysis |
| Budget | Claude Haiku, GPT-4o-mini, Gemini Flash | $ | Bulk processing, monitoring, formatting |

### Rule of thumb

A 3-agent task with all Opus models costs ~$3-5. The same task with 1 Opus orchestrator + 2 Sonnet executors costs ~$1-2. **The quality difference for execution tasks is negligible.**

## Multi-Model Benefits

Using different model providers (not just different tiers) adds value:

| Benefit | How |
|---------|-----|
| **Hallucination detection** | If Claude and GPT disagree on a fact, at least one is wrong. Cross-check. |
| **Blind spot coverage** | Each model has different training data biases. Multiple models reduce blind spots. |
| **Availability** | If one provider goes down, others still work. |

### When to mix providers

- **High-stakes decisions**: Run the same question through 2-3 providers, compare answers
- **Fact-checking**: One model generates, another verifies
- **Availability-critical workflows**: Primary + fallback provider

### When NOT to mix providers

- **Simple execution tasks**: One cheap model is fine
- **Tight budget**: Multi-model means multi-cost
- **Tight latency**: Parallel calls to multiple providers still take wall-clock time

## Cross-Family Review Evidence

The strongest empirical case for multi-model usage is review, not generation.

Research from Kim et al. in 2025 across 350+ LLMs found same-family error correlation was higher than cross-family correlation. Research from Young in 2026 pushes the idea further: when agents share the same training data, debate approaches zero benefit; when they come from different families, the value becomes meaningful.

That leads to a practical rule:

> Use a different model family for review whenever the review matters.

Claude reviewing GPT code, or GPT reviewing Claude code, is not just aesthetic diversity. It is the design choice with the clearest evidence behind it.

There is a limit. Research from Goel et al. at ICML 2025 found up to 90% output similarity among frontier models. Cross-family review improves your odds, but do not expect it to catch everything. Some blind spots are universal.

### Special case: review-tuned models

OpenAI's CriticGPT was trained specifically for code review via RLHF and was preferred over human critiques in 63% of cases for naturally occurring LLM errors. If you have access to a review-tuned model, use it for review. Review is one of the few places where specialization clearly pays.

## Anti-Patterns

### Using the expensive model for everything

"I want the best results, so I'll use Opus for all agents."

**Problem**: 80% of agent tasks are execution (code, fetch, format). Opus doesn't produce meaningfully better code than Sonnet for most tasks. You're paying 5-10x more for no quality improvement.

**Fix**: Default to cheap. Only escalate to expensive for judgment tasks.

### Using the cheap model for orchestration

"I'll save money by using Haiku as my orchestrator."

**Problem**: The orchestrator makes all the important decisions -- task decomposition, error recovery, quality validation, escalation judgment. A cheap model here means worse decisions everywhere downstream.

**Fix**: The orchestrator is the one place where the expensive model pays for itself.

### Same-model judge and generator

"My Claude agent will review the output of my other Claude agent."

**Problem**: Same-model self-review creates style bias. The reviewer tends to agree with outputs that match its own generation patterns.

**Fix**: Use a different model (or at least a different temperature/system prompt) for review tasks.

## Budget Allocation Strategy

For a typical multi-agent project:

```
Total budget: $X

Orchestrator (expensive):   40% of budget
Execution agents (cheap):   40% of budget
Review/validation (mixed):  15% of budget
Buffer for retries:          5% of budget
```

The orchestrator gets a disproportionate share because it runs throughout the entire project. Execution agents are spawned and terminated quickly.

---

*See also: [Cost Control Guide](cost-control.md) for per-task budgets and optimization.*
