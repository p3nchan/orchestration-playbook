# Error Event Template

Standard format for subagent status reporting. Every subagent must emit one of these when it finishes.

---

## Status Formats

### Completed

```
COMPLETED {task_id}: completed
  Output: {file_path} ({file_size})
  Summary: {one-line description of what was produced}
```

### Failed

```
FAILED {task_id}: FAILED
  Error: {error_message}
  Type: {transient | auth | logic | state | catastrophic}
  Attempt: {n} of {max}
  Last step completed: {description}
  Suggestion: {what might fix this}
```

### Running (Progress Update)

```
RUNNING {task_id}: running
  Step: {n} of {total} — {description}
  Elapsed: {time}
  Next: {what happens next}
```

### Blocked

```
BLOCKED {task_id}: blocked
  Reason: {why the task cannot proceed}
  Needs: {what would unblock it — e.g., "API credentials", "human decision on X"}
  Can continue with: {any partial work that's usable}
```

---

## JSON Format (For Code-Based Orchestrators)

```json
{
  "task_id": "research-api",
  "status": "FAILED",
  "error": "API endpoint returned 401 Unauthorized",
  "error_type": "auth",
  "attempt": 1,
  "max_attempts": 3,
  "last_completed_step": "Fetched endpoint list",
  "suggestion": "Check API token expiration",
  "timestamp": "2026-03-15T10:30:00Z"
}
```

---

## Usage Notes

- Every subagent should emit exactly one final status event
- Status should be the **first** thing in the output, not buried in analysis
- The orchestrator uses the `error_type` field to decide: retry (transient), fix config (auth), restart with new prompt (logic), or escalate (catastrophic)
