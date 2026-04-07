# MVVM Data Flow (React Native)

```
User → View (JSX) ↔ ViewModel/Store ↔ Repository ↔ API/DB

View (React Component)
  - subscribes to ViewModel state
  - renders UI; sends user intents

ViewModel / Store
  - holds observable state
  - exposes actions (async)
  - depends on Domain/Repos, not UI kits

Repository (Data)
  - decides cache/local/network
  - maps DTOs ↔ Domain models

API / DB Adapters
  - axios/fetch client, SQLite/WatermelonDB
```

**Key points**
- One-way data flow: Intent → Action → State update → Re-render.
- ViewModel is platform-agnostic; test without rendering components.
- Keep effects (analytics, navigation) behind interfaces to avoid tight coupling.
