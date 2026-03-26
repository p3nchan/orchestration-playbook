# Task Envelope Template (Lightweight)

For simple tasks that need < 5 tool calls. Use the [full template](task-envelope.md) for anything more complex.

---

```markdown
## Task: {task-id}

- **Goal**: {One sentence}
- **Inputs**: `{file-path}` — {brief description}
- **Output**: `{output-file-path}` — {expected format}
- **Stop if**: {condition to bail out}
```

---

## When to Use Lightweight vs Full

| Signal | Template |
|--------|----------|
| < 5 expected tool calls | Lightweight |
| Single clear objective | Lightweight |
| No risk of side effects | Lightweight |
| Multiple acceptance criteria | Full |
| Needs explicit tool restrictions | Full |
| Risk level medium or high | Full |
| Depends on other tasks | Full |
