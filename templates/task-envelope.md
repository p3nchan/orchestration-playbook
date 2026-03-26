# Task Envelope Template (Full)

Copy and fill in for each subagent task.

---

```markdown
## Task: {task-id}

- **Goal**: {One sentence describing the desired outcome}
- **Acceptance Criteria**:
  - {Criterion 1 — objective, verifiable}
  - {Criterion 2}
  - {Criterion 3}
- **Inputs**:
  - `{file-path}` — {one-line description of what this file contains}
  - `{file-path}` — {one-line description}
- **Allowed Tools**: {explicit list: read files, write files, run tests, web search, etc.}
- **Forbidden**: {explicit list: do not delete files, do not access paths outside X, etc.}
- **Risk Level**: {low / medium / high}
- **Budget**: {max N tool calls, ~$X estimated}
- **Checkpoint**: {When to save progress — e.g., "after data collection, before analysis"}
- **Stop Conditions**:
  - {Condition that means "give up" — e.g., "input file is missing or malformed"}
  - {Condition — e.g., "3 consecutive failures on the same step"}
- **Output**:
  - Path: `{output-file-path}`
  - Format: {markdown / JSON / CSV / etc.}
- **Output Protocol**: Your FIRST line must be one of:
  - `COMPLETED: {summary}` if successful
  - `FAILED: {reason}` if unsuccessful
  - `BLOCKED: {reason}` if you cannot proceed
```

---

## Usage Notes

- **Goal**: Should be achievable in a single agent session. If it needs multiple phases, split into separate envelopes.
- **Acceptance Criteria**: Must be objectively verifiable. "Good analysis" is not verifiable. "Contains >= 3 comparison dimensions with data sources cited" is.
- **Inputs**: Always provide paths + brief descriptions. Never paste full file contents into the envelope.
- **Allowed Tools**: Be explicit. Omissions are ambiguous. If the agent shouldn't write files, say so.
- **Budget**: Estimate tool calls based on task complexity. Simple read+analyze = 3-5. Code+test = 5-10. Research = 5-15.
- **Stop Conditions**: Tell the agent when to give up. Without this, agents will retry forever or hallucinate a result.
