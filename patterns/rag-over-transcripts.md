# RAG over meeting transcripts

## Problem

Decisions, context and commitments live in call recordings and never become searchable. Every recap is rebuilt from memory, and institutional knowledge evaporates when someone leaves.

## Approach

Transcripts are ingested through an idempotent pipeline, chunked by speaker, embedded, and stored as vectors with rich metadata (source type, visibility, date, project). Downstream, recaps and briefs are generated from retrieved context instead of from scratch, with strict rules on what can be exposed externally.

## Architecture

```
recording webhook / nightly poll
        ▼
[ ingest pipeline ]  (idempotent: status pending → embedded → failed)
   ├─ resolve which project/mission the source belongs to (gate: unknown → stop)
   ├─ chunk by speaker, truncate to model limits
   ├─ embed (text-embedding model)
   └─ upsert into a vector store with metadata
        ▼
[ retrieval ]  → recaps, briefs, prep docs grounded in real context
```

## Key decisions

- **Idempotent ingestion.** A journal table tracks every source; re-running never double-inserts. Re-processing is triggered only when the source actually changed (compare last-edited vs last-embedded).
- **Visibility tags are load-bearing.** Internal chunks enrich steering context but are never surfaced in a client-facing output. The boundary is enforced at retrieval, not by hoping.
- **Source priority.** Definitive sources (emails actually sent to a client) outrank drafts; transcripts outrank ad-hoc notes. Retrieval weights reflect that.
- **Mission gate.** A source that can't be resolved to a known project is stopped silently rather than processed into the wrong context.

## Why it matters for growth

Grounded generation kills hallucination in client recaps and team briefs. It turns hours of calls into a searchable memory that makes every follow-up sharper, and it scales the same way across every account.
