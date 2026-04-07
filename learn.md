# Part 06 — System Design & Architecture

> **500 questions** · Most differentiating part at 10–12 LPA · Separates seniors from juniors

At the 10–12 LPA band, interviewers don't just ask *what* you know — they ask *how you would build it*. System design questions reveal whether you can own a feature end-to-end: architecture decisions, trade-offs, scalability, and real-world constraints. This part covers both **mobile system design** (app architecture, offline-first, state management patterns) and **backend system design** (APIs, caching, queues) as they relate to React Native apps.

---

## 📊 Topic Map & Question Distribution

| # | Topic | Questions | Frequency | Key Skills |
|---|-------|-----------|-----------|-----------|
| 1 | App Architecture Patterns (MVVM, Clean Arch, MVC) | Q1–Q50 | 🔴 Very High | Folder structure, separation of concerns |
| 2 | State Management Architecture | Q51–Q100 | 🔴 Very High | Redux, Zustand, Context at scale |
| 3 | Offline-First Architecture | Q101–Q150 | 🔴 Very High | Sync, conflict resolution, queuing |
| 4 | API Design & Integration Patterns | Q151–Q200 | 🔴 Very High | REST, GraphQL, caching, retry |
| 5 | Authentication & Security Architecture | Q201–Q250 | 🔴 Very High | JWT, OAuth, biometrics, token refresh |
| 6 | Performance Architecture | Q251–Q290 | 🟡 High | TTI, render optimisation, memory |
| 7 | Real-Time Architecture (WebSockets, SSE) | Q291–Q320 | 🟡 High | Live updates, reconnect, presence |
| 8 | Micro-Frontend / Modular Architecture | Q321–Q350 | 🟡 High | Mono-repo, code splitting, teams |
| 9 | ERP / Enterprise App Design Case Studies | Q351–Q420 | 🔴 Very High | Your project-based answers |
| 10 | System Design Interview Framework | Q421–Q500 | 🔴 Very High | How to structure a 45-min answer |

---

## 🔥 Must-Know Before Any System Design Interview

These are the high-signal questions that separate 8 LPA engineers from 12 LPA engineers. Know the *reasoning* behind each answer, not just the answer itself.

### Architecture Patterns
1. **How would you architect a React Native ERP app for 50,000 employees?**
2. **What is Clean Architecture and how do you implement it in React Native?**
3. **What is the difference between MVVM, MVC, and MVP in mobile development?**
4. **How do you structure folders in a large React Native project (feature-based vs type-based)?**
5. **What is the Repository pattern and why use it in React Native?**

### Offline & Sync
6. **How do you design an offline-first attendance system that syncs when online?**
7. **What is conflict resolution in offline sync? How do you handle it?**
8. **How do you build an optimistic UI for a slow network environment?**
9. **Design a queue system for failed API requests that retries when connectivity is restored.**
10. **How does WatermelonDB's sync protocol work?**

### State Management
11. **When do you choose Redux vs Zustand vs React Query vs Context API?**
12. **How do you prevent Redux from becoming a performance bottleneck?**
13. **What is the flux pattern and why does Redux follow it?**
14. **How do you handle derived state vs server state vs local UI state?**
15. **Design the state architecture for a multi-tab ERP app with shared data.**

### API & Networking
16. **How do you design a token refresh interceptor that handles concurrent requests?**
17. **What is the difference between REST and GraphQL from a mobile client perspective?**
18. **How do you implement request deduplication for concurrent identical API calls?**
19. **Design a caching strategy for an app that works offline but has fresh data when online.**
20. **How do you implement pagination — offset vs cursor — and which is better for mobile?**

### Security
21. **How do you securely store a JWT in a React Native app?**
22. **What is certificate pinning and when is it required?**
23. **How do you prevent reverse engineering of a React Native APK?**
24. **Design an authentication flow with biometrics + PIN fallback.**
25. **What is the OWASP Mobile Top 10 and which items apply to your app?**

### Real-World Case Studies
26. **Design a real-time chat feature for an ERP app (like internal messaging).**
27. **Design a payroll calculation engine that works offline.**
28. **Design the architecture for a multi-company ERP app (tenant isolation).**
29. **How would you migrate a monolithic React Native app to a modular architecture?**
30. **Design a push notification system for attendance reminders at scale.**

---

## 📥 Quick Answers to the 30 Must-Know Questions

Use these as a speaking crib sheet; expand with your own project anecdotes when interviewing.

1) Feature-first Clean Architecture; offline-first; React Query + Zustand; WatermelonDB; retry/backoff API layer; modular auth/attendance/payroll; push + WebSocket for critical live data; OTA + staged rollout.
2) Presentation → Domain → Data layers; dependencies point inward; interfaces in Domain; repositories/adapters in Data; RN/axios/db are outer devices; DI to swap implementations; pure TS in Domain for easy tests.
3) MVC couples view/controller; MVP adds presenter but view-aware; MVVM uses observable state (stores) decoupled from view; best testability and reuse with hooks/observables in RN.
4) Feature-based folders (auth/attendance/etc) each with screens/components/store/api/types; shared/ for UI/utils; navigation/ + services/ + store/ for glue; avoids type-based sprawl.
5) Repository pattern decouples UI from data sources; screens call repo interfaces; repo chooses cache/local DB/network; enables offline-first, testing, and source swapping.
6) Offline attendance: write to queue + local DB, optimistic UI, pull-then-push sync with timestamps, idempotency keys, connectivity listener flush.
7) Conflicts: define source of truth (server-wins or merge); include version/updatedAt; deterministic merge; surface unresolved cases to user.
8) Optimistic UI: render immediately with temp id; stash rollback snapshot; call API; on success replace temp; on fail rollback + toast; server ops idempotent.
9) Retry queue: persist ops (type/payload/idempotency key) to MMKV/DB; FIFO with backoff on reconnect; mark successes; keep failed; dedupe server-side.
10) WatermelonDB sync: pull changes since last token, apply in batch, push pending locals, receive new token; conflict policy handled in app (often server-wins).
11) Server state → React Query/RTKQ; complex global/client rules or big teams → Redux; lightweight global UI → Zustand/Context; local UI → component state.
12) Prevent Redux perf issues: normalize; memo selectors; slice by feature; shallowEqual; avoid giant root re-renders; use RTKQ for server data; enable batching.
13) Flux: unidirectional flow Action → Dispatcher → Store → View; Redux is Flux with single store/pure reducers.
14) Server state with caching (React Query/RTKQ); derived via memoized selectors; local/ephemeral with `useState/useReducer`; avoid duplicating sources of truth.
15) Multi-tab ERP: global auth/session; per-feature slices; query keys per tenant/tab; shared selectors; refetch on focus/background; cache hydration per tab.
16) Token refresh: gate concurrent 401s; queue pending requests; single refresh call; replay with new token; `_retried` guard; logout on refresh failure.
17) REST vs GraphQL (mobile): REST simpler/CDN friendly but can over/under-fetch; GraphQL reduces round-trips and shapes data but needs caching/persisted queries and governance; choose per use case.
18) Dedup requests: in-flight map by URL+params returning same promise; or React Query `staleTime/dedupeInterval`; cancel outdated calls on navigation.
19) Caching offline+fresh: memory → local DB → network; stale-while-revalidate; ETag; background refresh on focus; TTL/size eviction; version schemas.
20) Pagination: offset simple but dupes with inserts; cursor (time/id) more reliable, smaller payloads, resumable; prefer cursor for mobile.
21) JWT storage: refresh in Keychain/Keystore; access in memory/encrypted MMKV; never AsyncStorage; rotate keys; pin domains; short-lived access tokens.
22) Cert pinning: bundle cert/SPKI hash; use for finance/health/enterprise; rotate pins with overlap; defends MITM.
23) Anti-reverse: R8/Proguard; remove debug logs; Hermes bytecode; root/jailbreak detection; integrity checks; keep secrets server-side; obfuscate native libs.
24) Biometrics + PIN: check capability; primary biometric prompt; fallback PIN screen; PIN hashed + salted in secure storage; throttle attempts; secure enclave keys.
25) OWASP Mobile Top 10 focus: M1 permissions, M2 storage (no secrets in AsyncStorage), M3 comms (TLS+pinning), M4 session mgmt, M5 crypto, M7 tampering, M9 reverse engineering.
26) Chat: WebSocket with reconnect/backoff; normalized store; optimistic send with temp ids; heartbeat presence; offline queue; signed-URL uploads; throttled typing indicators.
27) Payroll offline: versioned rule tables local; compute on device; queue adjustments; server reconciliation with audit logs; feature flags for rule updates.
28) Multi-tenant: tenant context in headers/cache keys; per-tenant schema/DB; feature/branding config; per-tenant rate limits and keys; isolation tests.
29) Monolith → modular: identify domains; extract packages (ui/data/domain); add dependency rules; shared design system; migrate feature by feature; use Nx/Turbo.
30) Push at scale: FCM/APNs via gateway; topic/segment targeting; device token store per user/tenant; dedupe + rate limit; actionable deep-links; permission fallbacks; background data sync.

## 🗂️ Folder Structure

```
part-06-system-design/
├── README.md                         ← This file
├── questions.md                      ← All 500 questions with answers
└── diagrams/
    ├── clean-architecture.md         ← Clean Architecture layer diagram
    ├── offline-sync-flow.md          ← Sync protocol flowchart
    ├── token-refresh.md              ← JWT refresh interceptor flow
    ├── erp-state-architecture.md     ← ERP state management diagram
    ├── realtime-websocket.md         ← WebSocket reconnect architecture
    └── mvvm-pattern.md               ← MVVM data flow diagram
```

---

## 🧠 How to Answer a System Design Question

System design interviews are open-ended. Interviewers want to see your *thinking process*, not just the final answer. Use this framework:

### The REACT Framework (Mobile System Design)

```
R — Requirements Clarification  (2–3 min)
    Functional:      What does it DO? (features, user actions)
    Non-functional:  Scale, performance, offline, security, platform

E — Estimate & Constraints  (1–2 min)
    Users:    1,000 / 10,000 / 1,000,000?
    Data:     How much? How often updated?
    Network:  Always online? Offline first?
    Devices:  iOS only? Android? Both? Tablets?

A — Architecture Decision  (5–8 min)
    Draw the high-level component diagram
    Major layers: UI, State, Service, Storage, API
    Technology choices + WHY (not just what)

C — Core Design  (10–15 min)
    Data models and schema
    State management approach
    API design (endpoints, payloads, pagination)
    Key algorithms (sync, conflict resolution, caching)
    Navigation and screen flow

T — Trade-offs & Edge Cases  (5 min)
    What did you sacrifice for simplicity?
    What breaks at scale?
    Security considerations
    Error handling and fallbacks
```

---

## 📐 Architecture Patterns Quick Reference

### Clean Architecture (most important for interviews)

```
┌────────────────────────────────────────────────┐
│             Presentation Layer                 │
│   Screens, Components, ViewModels / Stores     │
│   (knows about: Domain)                        │
├────────────────────────────────────────────────┤
│                Domain Layer                    │
│   Use Cases, Business Logic, Entity Types      │
│   (knows about: nothing — pure TypeScript)     │
├────────────────────────────────────────────────┤
│               Data Layer                       │
│   Repositories, API clients, Local DB, Cache   │
│   (knows about: Domain interfaces only)        │
└────────────────────────────────────────────────┘

Golden Rule: Dependencies point INWARD only.
The Domain layer has zero external dependencies.
This makes it testable in isolation with plain Jest — no mocks for React Native.
```

### Feature-Based Folder Structure (recommended for large apps)

```
src/
├── features/
│   ├── auth/
│   │   ├── screens/        LoginScreen.tsx, RegisterScreen.tsx
│   │   ├── components/     LoginForm.tsx, BiometricButton.tsx
│   │   ├── hooks/          useAuth.ts, useSession.ts
│   │   ├── store/          authSlice.ts, authSelectors.ts
│   │   ├── api/            authApi.ts
│   │   └── types/          auth.types.ts
│   ├── attendance/
│   │   ├── screens/        AttendanceScreen.tsx, HistoryScreen.tsx
│   │   ├── components/     CheckInButton.tsx, AttendanceCard.tsx
│   │   ├── hooks/          useAttendance.ts, useGeofence.ts
│   │   ├── store/          attendanceSlice.ts
│   │   ├── api/            attendanceApi.ts
│   │   └── types/          attendance.types.ts
│   └── payroll/
│       └── ...
├── shared/
│   ├── components/         Button, Card, Modal, Input, Skeleton
│   ├── hooks/              useDebounce, useNetwork, usePermission
│   ├── utils/              formatCurrency, validateEmail, dates
│   ├── constants/          routes, colors, typography, config
│   └── types/              common.types.ts
├── navigation/             AppNavigator.tsx, AuthNavigator.tsx, TabNavigator.tsx
├── store/                  rootReducer.ts, store.ts, middleware/
├── services/               apiClient.ts, pushNotifications.ts, analytics.ts
└── assets/                 fonts/, images/, icons/
```

### State Management Decision Tree

```
                   Is this data from a remote server (API)?
                              /              \
                           YES               NO
                            |                 |
               Use React Query / RTK Query    Is it global UI state?
               (fetching, caching, sync)       (auth, theme, cart)
                            |                 /          \
               Does it need complex       YES             NO
               client-side transforms?    |               |
                  /          \         Zustand         useState /
                YES           NO      (or Redux        useReducer
                 |             |      for large        (local state)
               Redux         RTK        teams)
              (normalised    Query
               store)       cache
```

---

## 🏗️ Key Architectural Concepts to Master

### 1. Repository Pattern
Separates data access from business logic. Screens never call APIs directly — they go through a Repository, which decides whether to use cache, local DB, or network.

```typescript
// interface lives in Domain layer
interface EmployeeRepository {
    getAll(deptId: string): Promise<Employee[]>;
    getById(id: string): Promise<Employee>;
    create(data: Partial<Employee>): Promise<Employee>;
    sync(): Promise<void>;
}

// implementation lives in Data layer
class EmployeeRepositoryImpl implements EmployeeRepository {
    constructor(
        private api: EmployeeApi,
        private db: EmployeeDatabase,
        private cache: CacheService,
    ) {}

    async getAll(deptId: string): Promise<Employee[]> {
        const cached = this.cache.get(`employees:${deptId}`);
        if (cached) return cached;

        if (!isOnline()) return this.db.query(deptId);     // offline → local DB

        const data = await this.api.getEmployees(deptId); // online → network
        this.cache.set(`employees:${deptId}`, data, '10m');
        await this.db.upsertAll(data);                    // update local DB
        return data;
    }
}
```

### 2. Optimistic UI
Update the UI immediately before the API call completes. Roll back on failure.

```typescript
const markAttendance = async () => {
    const tempId = `temp_${Date.now()}`;
    const optimisticRecord = { id: tempId, status: 'checked_in', synced: false };

    // 1. Immediately update UI — user sees instant feedback
    addRecord(optimisticRecord);

    try {
        // 2. API call in background
        const confirmed = await api.post('/attendance/checkin');
        // 3. Replace optimistic record with confirmed server record
        replaceRecord(tempId, confirmed.data);
    } catch (error) {
        // 4. Roll back on failure
        removeRecord(tempId);
        showToast('Check-in failed. Please try again.');
    }
};
```

### 3. Offline Queue
Queue failed operations and replay when connectivity is restored.

```typescript
// Operations are queued to MMKV, replayed on reconnect
const offlineQueue = {
    add: async (operation: QueuedOperation) => {
        const queue = MMKV.getJSON<QueuedOperation[]>('offline_queue') ?? [];
        MMKV.setJSON('offline_queue', [...queue, { ...operation, queuedAt: Date.now() }]);
    },

    flush: async () => {
        const queue = MMKV.getJSON<QueuedOperation[]>('offline_queue') ?? [];
        if (queue.length === 0) return;

        const results = await Promise.allSettled(
            queue.map(op => executeOperation(op))
        );

        // Remove successfully replayed operations
        const failed = queue.filter((_, i) => results[i].status === 'rejected');
        MMKV.setJSON('offline_queue', failed);

        console.log(`Flushed ${queue.length - failed.length}/${queue.length} queued operations`);
    },
};

// Auto-flush on reconnect
NetInfo.addEventListener(state => {
    if (state.isConnected && state.isInternetReachable) {
        offlineQueue.flush();
    }
});
```

### 4. Token Refresh Interceptor (handles concurrent 401s)

```typescript
let isRefreshing = false;
let refreshQueue: Array<(token: string) => void> = [];

axiosInstance.interceptors.response.use(null, async (error) => {
    if (error.response?.status !== 401) return Promise.reject(error);
    if (error.config?._retried) return Promise.reject(error); // prevent infinite loop

    if (isRefreshing) {
        // Queue all concurrent 401s — wait for the refresh in progress
        return new Promise((resolve) => {
            refreshQueue.push((newToken: string) => {
                error.config.headers.Authorization = `Bearer ${newToken}`;
                error.config._retried = true;
                resolve(axiosInstance(error.config));
            });
        });
    }

    isRefreshing = true;
    try {
        const { accessToken } = await refreshAccessToken();
        refreshQueue.forEach(cb => cb(accessToken));      // replay queued requests
        refreshQueue = [];
        error.config.headers.Authorization = `Bearer ${accessToken}`;
        error.config._retried = true;
        return axiosInstance(error.config);               // replay original request
    } catch (refreshError) {
        refreshQueue = [];
        await performLogout();
        return Promise.reject(refreshError);
    } finally {
        isRefreshing = false;
    }
});
```

---

## 📋 Case Study: ERP Attendance System Design

This is exactly the type of question you will face at 10–12 LPA. Practice answering it in 30–45 minutes — out loud, without notes.

### Problem Statement
> *Design an attendance system for a manufacturing company with 5,000 employees across 8 factory locations. Employees check in and out via a React Native app. Factory floors have poor internet connectivity. Managers need real-time headcount reports.*

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                Employee Mobile App (React Native)        │
│                                                         │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Biometric  │  │  GPS / Geo-  │  │  WatermelonDB │  │
│  │    Auth     │  │   fence SDK  │  │  (SQLite ORM) │  │
│  └──────┬──────┘  └──────┬───────┘  └───────┬───────┘  │
│         └───────────────┬┘                  │           │
│                  ┌──────▼──────────────────▼┐           │
│                  │   AttendanceService       │           │
│                  │  (offline queue + sync)   │           │
│                  └──────────────┬────────────┘           │
└─────────────────────────────────┼───────────────────────┘
                                  │ HTTPS (when online)
                     ┌────────────▼────────────┐
                     │      API Gateway        │
                     │  (Node.js / NestJS)     │
                     └─────┬──────────┬────────┘
                      ┌────▼───┐  ┌───▼─────┐
                      │Postgres│  │  Redis  │
                      │(RDS)   │  │ (cache) │
                      └────────┘  └────┬────┘
                                       │ pub/sub
                               ┌───────▼──────┐
                               │  WebSocket   │
                               │ (Manager     │
                               │  Dashboard)  │
                               └──────────────┘
```

### Key Design Decisions + Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| Offline DB | WatermelonDB (SQLite) | Complex queries, reactive, battle-tested | Complex setup vs JSON flat files |
| Sync strategy | Pull-then-push with server timestamp | Simple, predictable | Eventual consistency (not real-time) |
| Conflict resolution | Server-wins | Easy to reason about | May lose a local edit in rare race condition |
| Auth token storage | Keychain + MMKV cache | Security (Keychain) + speed (MMKV) | Slightly complex token refresh flow |
| Location validation | Client-side geo-fence | Battery-friendly | Spoofable (acceptable for most companies) |
| Real-time updates | WebSocket per manager | Simple, direct | Doesn't scale beyond ~100 concurrent managers |
| Offline queue | MMKV-backed operation log | Survives app restart | Need idempotency keys to prevent duplicate records |

---

## 📝 Questions File

All 500 questions with full answers, code examples, architecture diagrams, and whiteboard-style explanations are in:

👉 [`questions.md`](./questions.md)

---

## 🗓️ Study Schedule for This Part

| Day | Focus Area | Questions | Target Outcome |
|-----|-----------|-----------|---------------|
| Day 1 | App architecture patterns | Q1–Q50 | Draw Clean Architecture from memory in 5 min |
| Day 2 | State management architecture | Q51–Q100 | Pick the right tool for any state scenario |
| Day 3 | Offline-first design | Q101–Q150 | Whiteboard a sync protocol with conflict resolution |
| Day 4 | API & networking patterns | Q151–Q200 | Implement token refresh interceptor without notes |
| Day 5 | Auth & security architecture | Q201–Q250 | Audit your own ERP app against OWASP Top 10 |
| Day 6 | Performance architecture | Q251–Q290 | Profile a slow screen and architect the fix |
| Day 7 | Real-time & ERP case studies | Q291–Q420 | Answer 2 case studies out loud, timed at 30 min each |
| Day 8 | Framework + mock interview | Q421–Q500 | Do 2 full mock system design interviews with a friend |

---

## 🎯 Interview Scoring Rubric

Interviewers at this level score system design answers on five dimensions:

| Dimension | Weight | What Earns Full Marks |
|-----------|--------|----------------------|
| **Requirements clarity** | 15% | Asked 3–5 clarifying questions before designing |
| **Architecture breadth** | 20% | Identified all major components needed |
| **Architecture depth** | 25% | Explained *why* each technology was chosen |
| **Trade-off awareness** | 20% | Explicitly called out what you gave up |
| **Real-world experience** | 20% | Connected to something you actually built |

**The winning move at every question:** Connect back to your ERP project.
> *"In our attendance module, we faced exactly this. We solved it by [X] because [Y]. We considered [Z] but rejected it because [trade-off]."*

---

## 🔗 Related Parts

- State management implementation → **[Part 03 — State Management](../part-03-state-management/)**
- API design patterns → **[Part 04 — APIs & Networking](../part-04-apis-networking/)**
- CI/CD and release architecture → **[Part 05 — DevOps & CI/CD](../part-05-devops-cicd/)**
- Explaining your designs in HR rounds → **[Part 08 — HR & Behavioural](../part-08-hr-behavioural/)**

---

*Part 06 of 8 — [← Back to main README](../README.md)*
