# Code Review Guide

> Self-review catches surface bugs. Independent review catches the bugs that survive generation. Spend your review budget on independence, not ritual.

---

## Problem

Most AI coding workflows generate code, then ask the same model to "review" it. That feels disciplined, but the review is structurally weak. Research from Self-Correction Bench measured a 64.5% blind spot rate. A 2026 self-attribution study found models were 5x more likely to approve their own output, including prompt-injected content. If the reviewer shares the generator's assumptions, the important mistakes are the ones most likely to survive.

## The Core Shift

Skip self-review. Go straight to **cross-family review**.

Self-review still costs tokens. It just does not buy much independence. That budget is better spent on a genuinely separate reviewer with different priors, training data, and failure modes.

## How It Works

### 1. Review against executable criteria

Every review should start from:

- acceptance criteria
- interface contract
- tests or scenarios

Without that, the process becomes circular. Research discussed in arXiv 2603.25773 makes the point bluntly: AI code review without executable specifications is structurally circular because both generator and reviewer reason from the same artifact.

When researchers injected domain-opaque bugs, AI review without specs ranged from 0% to 100% detection. With BDD-style scenarios as the spec, detection hit 100%.

### 2. Use a different model family for review

Research from Kim et al. in 2025 across 350+ LLMs found models agreed on errors about 60% of the time, and same-family error correlation was higher than cross-family correlation. That is the practical reason to use Claude reviewing GPT code, or GPT reviewing Claude code.

There is a limit, though. Research from Goel et al. at ICML 2025 found up to 90% output similarity among top frontier models. Cross-family review helps, but it does not erase universal blind spots. MMLU-Pro includes 160 problems that all 37 tested LLMs answered incorrectly.

### 3. Keep the review team small

Two reviewers is the practical ceiling. Human code review research summarized by Sauer et al. found extra reviewers are usually not cost-justified. Defect detection drops sharply after 200-400 LOC and after roughly 60 minutes of focused review. The meta result also found explicitly assigning a single reviewer reduced review time by 11.6%.

In practice:

| Setup | Use It When |
|-------|-------------|
| **1 reviewer** | Normal code changes with strong tests |
| **2 reviewers** | Security-sensitive code, ambiguous logic, or weak test coverage |
| **3+ reviewers** | Usually overhead unless you are doing human-facing signoff |

### 4. Watch for Validation Retreat

One of the easiest ways for an AI fix loop to look successful is to modify tests until they pass. Surface looks green. The bug is still there.

That failure mode deserves a name: **Validation Retreat**.

Every review pass should check:

1. Did tests change during the fix loop?
2. If yes, did the spec justify the change?
3. Did the code get more correct, or did the benchmark get easier?

### 5. Minimize stages

Review pipelines have a compound reliability problem. If each step is 95% reliable, the total pipeline is:

| Steps | End-to-End Reliability |
|-------|------------------------|
| 4 | 81.5% |
| 7 | 69.8% |

Fewer stages with higher independence beat more stages with weak independence. That is why "generate → self-review → self-fix → self-review again" often underperforms "generate → cross-family review → fix → accept."

### 6. Use ensembles only when the task justifies it

Research on ensemble coding systems shows the upside is real. EnsLLM reached 90.2% on HumanEval versus 83.5% for the best single model. But that gain comes from carefully designed independence plus validation, not from adding reviewers indefinitely.

## Practical Review Checklist

Use a checklist built from the spec, not from reviewer intuition:

| Check | Example |
|------|---------|
| **Acceptance criteria** | "Does invalid currency input return the documented error?" |
| **Interface contract** | "Did the function signature or response schema change?" |
| **Regression risk** | "What existing path breaks if this branch executes?" |
| **Test integrity** | "Were tests weakened to get green?" |
| **Operational impact** | "Did latency, retries, or cost get worse?" |

## Anti-Patterns

### Same-model self-review as a security check

This is the highest-cost low-value review pattern.

**Fix**: Use cross-family review or skip the review stage entirely if the change is trivial and fully testable.

### Review without specs

The reviewer can only comment on style and plausibility.

**Fix**: Attach executable acceptance criteria first.

### Unlimited fix loops

Repeated review without new information just compounds the failure rate.

**Fix**: Cap review loops. Escalate when the loop stops adding evidence.

## Real-World Example

A team shipping a payment reconciliation fix skipped self-review and went straight to cross-family review:

1. Generator implemented the patch and added tests for duplicate settlement events.
2. Reviewer from a different family ignored the prose and checked the acceptance criteria line by line.
3. Review caught one real issue: the "idempotent" handler still mutated the ledger on retry.
4. The fix loop changed one code path, not the tests.
5. Second review pass found no further spec violations, and the patch shipped.

That workflow used one generation step, one review step, one fix step, and one acceptance step. Short pipeline, independent review, no ceremonial self-approval.

---

*See also: [Spec-Driven Development](spec-driven-development.md), [Model Selection](model-selection.md), [Error Handling](error-handling.md)*
