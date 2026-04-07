# Clean Architecture (React Native) — Text Diagram

```
┌──────────────────────────────────────────────┐
│                 Presentation                 │
│  Screens / Components / Navigation           │
│  ViewModels / Hooks / Stores                 │
│  Depends on: Domain interfaces only          │
└───────────────────▲──────────────────────────┘
                    │
┌───────────────────┼──────────────────────────┐
│                    Domain                    │
│  Entities / Value Objects / Use Cases        │
│  Pure TypeScript, no RN/IO imports           │
│  Knows nothing about Data or Presentation    │
└───────────────────▲──────────────────────────┘
                    │
┌───────────────────┼──────────────────────────┐
│                    Data                      │
│  Repositories (implements Domain interfaces) │
│  API Client / DB Adapter / Cache Adapter     │
│  DTO ↔ Domain mappers                        │
│  Depends on: Domain ONLY                     │
└───────────────────┴──────────────────────────┘
```

**Rules**
- Dependencies point inward; Domain is dependency-free.
- UI never imports API/DB; it only calls Use Cases or Repos interfaces.
- Replace adapters (REST ↔ GraphQL, SQLite ↔ WatermelonDB) without touching Domain/Presentation.

**Testing lanes**
- Domain: pure Jest, zero mocks.
- Data: contract tests with mocked transport/DB.
- Presentation: component tests with fake Domain interfaces.
