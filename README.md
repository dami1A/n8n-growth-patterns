# n8n growth patterns

Reusable automation patterns I use to run growth and marketing ops as an operator, not just a strategist. These are the architectures behind AI-assisted outbound, ops intelligence and self-healing automation that let a small team run a full marketing function.

> **Anonymized by design.** No client data, no credentials, no real IDs. These describe the *architecture and the reasoning*, so they can be adapted to any stack. The point is the design decisions, not a copy-paste export.

## Why this exists

Most marketing automation breaks in the same places: it floods Slack, it retries non-idempotent steps, it has no observability, and nobody can tell a healthy run from a wrong result. These patterns are the fixes I converged on after running them in production across multiple missions.

## Patterns

| Pattern | Problem it solves |
|---|---|
| [Centralized Slack dispatcher](patterns/centralized-slack-dispatcher.md) | Notification chaos: every workflow posting its own way, flooding channels, no levels, no dedup. |
| [Self-healing workflow auto-debug](patterns/self-healing-workflow-autodebug.md) | Silent failures: errors that nobody sees until a client does. |
| [RAG over meeting transcripts](patterns/rag-over-transcripts.md) | Knowledge lost in calls: no searchable memory of what was decided. |
| [Lead enrichment pipeline](patterns/lead-enrichment-pipeline.md) | Raw leads with no context: manual research that doesn't scale. |

## Principles I hold across all of them

- **Idempotence over retries.** On a non-idempotent pipeline, an automatic retry is a fault amplifier, not a safety net.
- **Verify the real artifact, not the return code.** A `200` proves nothing; the post must be visible, the file readable, the row written.
- **Signal over noise.** Notify only when a human action is needed or something unexpected happened. Heartbeats train people to ignore the channel.
- **One source of truth per fact.** State, backlog and live status never live in two places.

---

Built and maintained by [Damien Arnaud](https://www.rayz.io) — Fractional CMO / Growth Lead.
