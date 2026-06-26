# Lead enrichment pipeline

## Problem

Raw leads arrive with a name and maybe a domain. Qualifying them by hand (company, role, fit, contactability) does not scale, and a sales team burns its best hours on research instead of conversations.

## Approach

A batched pipeline takes raw leads, enriches them from multiple sources, scores fit against the ICP, and routes only the qualified ones to outreach. Everything is dynamic: routing, ICP and messaging read from a single source of truth, never hardcoded, so a change is one update, not a patch across N workflows.

## Architecture

```
raw leads (form, list, scrape)
        ▼
[ enrichment pipeline ]  (Split In Batches, never recursion)
   ├─ resolve company + domain
   ├─ enrich (firmographics, role, signals)
   ├─ score fit vs ICP  (composite: semantic + role + recency + tags)
   └─ branch: qualified → outreach | weak → nurture | junk → drop (logged)
        ▼
[ outreach ]  → personalized, with dedup across channels
```

## Key decisions

- **Dynamic over hardcoded.** No emails, names or IDs baked into workflows. Routing data lives in one store; a contact who changes is one update, not a code change.
- **Composite scoring with thresholds.** High score auto-qualifies, mid score qualifies with a log for review, low score drops with a reason. The log is what lets you tune the model.
- **Cross-channel dedup.** A lead reached by email and by social is one touch, not two. Dedup is enforced before send.
- **Batches, not recursion.** Large lists run through batched iteration with state stored between iterations, never a recursive fan-out that melts the instance.

## Why it matters for growth

This is the repeatable demand-gen engine in practice: raw volume in, qualified pipeline out, instrumented at every step. It is what separates an engine that scales across markets from one that stalls the moment volume grows.
