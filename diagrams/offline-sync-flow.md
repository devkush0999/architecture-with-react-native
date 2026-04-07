# Offline Sync Flow (Pull → Push)

```
       ┌─────────┐
       │ NetInfo │ detects connectivity
       └────┬────┘
            │ online?
            ▼
┌──────────────────────────┐
│ 1) Pull changes          │
│   - send lastSyncToken   │
│   - server returns diff  │
└───────┬──────────────────┘
        │ apply
        ▼
┌──────────────────────────┐
│ 2) Update local DB/Cache │
│   - batch upserts        │
└───────┬──────────────────┘
        │ then
        ▼
┌──────────────────────────┐
│ 3) Push pending ops      │
│   - queued w/ opId       │
│   - idempotent server    │
└───────┬──────────────────┘
        │ results
        ▼
┌──────────────────────────┐
│ 4) Resolve conflicts     │
│   - server-wins or merge │
│   - update local state   │
└───────┬──────────────────┘
        │
        ▼
┌──────────────────────────┐
│ 5) Persist new syncToken │
└──────────────────────────┘
```

**Design notes**
- Queue stored in MMKV/SQLite with `opId` for idempotency.
- Backoff + jitter on failures; stop on 4xx.
- Freshness labels in UI: “last synced 5m ago”.
- Manual “Sync now” button for supportability.
