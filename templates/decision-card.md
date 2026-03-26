# Decision Card Template

Use this when an agent hits a Gate-tier action and needs human approval.

---

```markdown
## Decision Required -- [Project] / [Task]

**Context**: {1-2 sentences of background. What were we doing? Why did we stop?}

**Options**:
  A. {Option} -- {Pros} / {Cons}
  B. {Option} -- {Pros} / {Cons}
  C. {Option} -- {Pros} / {Cons}

**Recommended**: {Letter} -- {One sentence justification}

**Risk**: {What could go wrong with the recommendation}

**Cost**: {Estimated cost of each option, if relevant}

**Default**: If no response in {timeframe} -> {safest default action}
```

---

## Example

```markdown
## Decision Required -- Data Pipeline / API Migration

**Context**: The v1 API will be deprecated on April 1. We need to migrate
to v2, but v2 has a breaking change in the auth flow.

**Options**:
  A. Migrate now (2h estimated) -- Pro: done early. Con: untested in production.
  B. Wait until March 28, test in staging first -- Pro: safer. Con: tight timeline.
  C. Build adapter layer for both v1 and v2 -- Pro: no deadline pressure. Con: 4h work, more complexity.

**Recommended**: B -- gives us 3 days to test while staying within the deadline.

**Risk**: If staging reveals issues, we have limited time to fix before April 1.

**Cost**: A ~$0.50, B ~$0.75 (staging + migration), C ~$1.50

**Default**: If no response in 24h -> proceed with B.
```

---

## Design Principles

- **The human should be able to reply with a single letter.** Don't make them think from scratch.
- **Always include a recommendation.** "What do you think?" is not helpful. "I recommend B because X" is.
- **Always include a default with timeout.** Work shouldn't stall indefinitely waiting for a response.
- **Keep it short.** If the decision card exceeds half a page, you're providing too much detail.
