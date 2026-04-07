# Real-Time WebSocket Architecture

```
┌─────────────┐   wss   ┌──────────────┐
│   Client    │────────▶│   Gateway    │
│  (RN App)   │◀────────│ (WS server)  │
└─────┬───────┘  heart-  └─────┬────────┘
      │         beats         │ pub/sub
      │                       ▼
      │                ┌────────────┐
      │                │  Redis/Kafka│
      │                └────┬───────┘
      │                     │ fanout
      ▼                     ▼
┌───────────┐        ┌─────────────┐
│ WS Client │        │  REST API   │
│  Manager  │        │ (fallback)  │
└────┬──────┘        └─────┬───────┘
     │ topics               │
     ▼                      ▼
 State Store         React Query invalidate
 (Zustand/Redux)     / refetch for reconcile
```

**Client strategy**
- Single WS connection; multiplex topics (chat, presence, alerts).
- Reconnect with exponential backoff + jitter; heartbeats to detect dead links.
- Sequence numbers/timestamps to order events; reconcile via REST on gaps.
- Collapse duplicate notifications using `collapseKey`.
