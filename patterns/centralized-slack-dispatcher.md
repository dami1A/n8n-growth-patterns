# Centralized Slack dispatcher

## Problem

Every workflow posts to Slack its own way. Markdown breaks, line breaks render as literal `\n`, channels flood, there are no severity levels, and the same incident shows up ten times. Operators stop reading the channel.

## Approach

One workflow owns all Slack output. Every other workflow and script calls it instead of posting directly. The dispatcher enforces formatting, severity levels, threading and deduplication in a single place, so improving notifications never means patching N callers.

## Architecture

```
caller (workflow or script)
        │  { level, channel, title, fields, link, thread_root_ts }
        ▼
[ dispatcher workflow ]
   ├─ Build Payload      → maps level → emoji + color + tag
   ├─ Dedup Guard        → drops identical messages inside a sliding window
   └─ chat.postMessage   → renders attachment, returns ts for threading
```

## Key decisions

- **Five levels** (`critical / action / auto / ok / info`), each with a fixed emoji and attachment color. Routing is by channel meaning, not by level: a channel can carry all levels.
- **Threading**: one incident is one root message plus replies. The caller captures the returned `ts` and replays it as `thread_root_ts`. The channel feed stays one line per incident.
- **Broadcast only when a human action is expected.** Silent resolutions stay inside the thread; the channel feed stays clean.
- **Dedup guard**: hash the normalized message (neutralize counters, timestamps and long IDs) into a sliding window. Identical messages inside the window are suppressed with a `deduped:true` response, so the caller still gets a `200`. Born from a flood incident: a watchdog firing every 2 minutes put 60+ identical criticals a day into one channel.

## Why it matters for growth

Marketing automation that cries wolf gets muted, and a muted channel is where a broken campaign or a failed send hides. Clean, leveled, deduped signal is what makes an automated growth stack trustworthy enough to run unattended.
