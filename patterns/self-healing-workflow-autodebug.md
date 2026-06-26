# Self-healing workflow auto-debug

## Problem

A workflow fails at 3am. Nobody sees it until a client notices the missing artifact a week later. Manual monitoring does not scale across dozens of automations.

## Approach

Every workflow points to a shared error workflow. On failure, the error is logged, persisted as a job file, and picked up by a watcher that hands it to an AI coding agent. The agent reads the failing execution, identifies the faulty node, and proposes or applies a surgical fix. A human is notified only when the agent escalates.

## Architecture

```
workflow error
      ▼
[ error logger ]  → row in a tracking sheet/table (status: unresolved)
      ▼
job file written to a pending/ queue
      ▼
[ watcher ]  → invokes an AI agent in non-interactive mode
      ▼
agent reads the execution, fixes the node, marks resolved
      ▼
notify a human ONLY on escalation
```

## Key decisions

- **Reproduce before patching.** The agent must prove the defect from a real execution before touching anything. "Not crashed" is not the same as "produced the right result".
- **No auto-retry on non-idempotent pipelines.** A retry on a pipeline that posts or sends would duplicate the side effect. Idempotence is baked into the code; opt-in tags mark the rare workflows where a re-fire is safe.
- **Single change at a time, with a snapshot before.** The platform versions each workflow; rollback is always armed.
- **Escalate, don't guess.** If the fix is uncertain or touches credentials, the agent stops and asks a human.

## Why it matters for growth

A growth stack is only as good as its uptime. Self-healing turns a fragile pile of automations into something that survives the night, so the team spends its time on strategy, not on babysitting workflows.
