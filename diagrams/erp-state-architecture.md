# ERP State Architecture (React Native)

```
┌─────────────────────────────────────────────┐
│ UI Screens (Tabs: Attendance / Payroll / …) │
└──────────────────────▲──────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
  Local UI State   App State     Server State
  (useState)       (Zustand/     (React Query /
                   Redux slices)  RTK Query)
         │             │             │
         │             │       Cache + Network
         │             └─────────▲─────────────
         │                       │
         │              Repositories (Domain)
         │                       │
         └──────────────Service Layer──────────
                                 │
                        API / DB / Cache
```

**Guidelines**
- Server state lives in React Query cache; invalidate on writes.
- App state (auth/session/feature flags) in a store; normalized selectors.
- Local UI state stays in components; don’t over-globalize.
- Tenant-aware cache keys: include `tenantId` to prevent leaks.
- Refetch on focus/app foreground; background prefetch critical data post-login.
