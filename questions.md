# 500 React Native System Design & Architecture Q&A (10–12 LPA, 3 YOE)

Each answer is a concise, interview-ready explanation. Use them as flash cards; expand with your own project examples when asked to go deeper.

## How to Use This File (Senior Playbook)
- **Open with structure**: In every system-design answer say, “I’ll cover requirements → scale → architecture → data/state → failure → security → rollout.” Timebox yourself.
- **Anchor in your experience**: After each key choice add, “In my last RN ERP module we…” even if hypothetical but plausible.
- **Diagrams first**: Sketch layers and data flow in 30–60 seconds. Keep one legend: rectangles = components, cylinders = data, lightning = events.
- **Trade-off table**: Give 2 choices + why you picked one + what you gave up.
- **Mobile constraints call-out**: Mention offline, flaky networks, battery, bundle size, store releases, and platform gaps (iOS BG tasks vs Android Headless JS).
- **Ops & rollout**: End with monitoring, feature flags, staged rollout/OTA, and kill switches.

## Senior-Level Cheat Sheet by Topic
- **Architecture**: Clean Arch with strict inward deps; feature-first folders; DI for repos/services; UI stays dumb; domain pure TS. Diagrams: `diagrams/clean-architecture.md`, `diagrams/mvvm-pattern.md`.
- **State**: Server state → React Query/RTKQ; app state → Redux/Zustand; local UI → component state; derived via selectors; avoid context overuse. Diagram: `diagrams/erp-state-architecture.md`.
- **Offline**: Queue + local DB + sync loop (pull then push) with idempotency and conflict policy; connectivity listener; freshness labels. Diagram: `diagrams/offline-sync-flow.md`.
- **API**: Idempotent writes, retry/backoff, request dedupe, token refresh single-flight, ETag/TTL caching, cursor pagination. Diagram: `diagrams/token-refresh.md`.
- **Security**: Keychain/Keystore for refresh; access in memory/MMKV; cert pinning when needed; secure screens; step-up auth for sensitive flows.
- **Performance**: Hermes, code split, list virtualization, Reanimated for animations, avoid JS thread blocking, measure TTI/TTFMP.
- **Real-time**: Single WS with topics, heartbeats + backoff, ordering via seq/ts, reconcile with REST, collapse notifications. Diagram: `diagrams/realtime-websocket.md`.
- **Modularity**: Feature packages, route registry, shared design system package, linted boundaries, CI cache (turbo/Nx).
- **ERP Cases**: Tenant isolation everywhere; audit trails; offline approvals; geo + biometrics for attendance; push + WS for managers.

## 1. App Architecture Patterns (Q1–Q50)
**How to answer (senior lens):** Start with requirements & scale, sketch Clean Architecture layers, justify foldering and state split, call out testing lanes, close with trade-offs (complexity vs delivery speed).
1. Q: How would you architect a React Native ERP app for 50k users?
   A: Feature-first Clean Architecture (Presentation → Domain → Data) with strict inward dependencies; offline-first local DB; React Query for server state + Zustand/Redux for app state; API client with retry/backoff and idempotency; push + WebSocket for real-time deltas; CI/CD with OTA + staged store rollout. (See diagram: diagrams/clean-architecture.md)
2. Q: What is Clean Architecture in RN?
   A: Three layers: Presentation (screens/view models), Domain (use cases/entities, pure TS), Data (repos/adapters: API, DB, cache). Only Domain is dependency-free. Use DI to inject implementations so tests can mock Data. (Diagram: diagrams/clean-architecture.md)
3. Q: Difference MVVM vs MVC vs MVP?
   A: MVC couples controller to view → brittle at scale; MVP improves separation but presenters still manipulate views; MVVM exposes observable state/actions, keeping UI declarative and testable—best fit with hooks/stores in RN.
4. Q: Why feature-based folders over type-based?
   A: Co-locates screens/components/store/api per feature, enabling modular ownership, faster onboarding, and local refactors without touching unrelated code.
5. Q: Repository pattern purpose?
   A: UI calls repo interfaces; repos choose cache/local DB/network and map DTOs to domain models. This isolates data sources, enables offline-first and makes business logic testable.
6. Q: Where to put navigation in Clean Arch?
   A: Presentation layer; treat navigators/coordinators as UI concern. Domain must not import navigation.
7. Q: How to enforce layer boundaries?
   A: TS path aliases + eslint-plugin-boundaries; CI check for forbidden imports; Domain forbids RN/axios/SQLite imports; Data only sees Domain interfaces.
8. Q: When choose MVC in RN?
   A: Only for throwaway prototypes/POCs where setup overhead must be minimal; otherwise prefer MVVM.
9. Q: Benefit of ViewModel (MVVM) in RN?
   A: Holds UI state + commands, survives re-render, testable without rendering components; keeps components dumb/presentational.
10. Q: How to model use cases?
    A: Plain functions/classes in Domain, each performing a business action; accept repositories/gateways as interfaces; return Result/Either to avoid throwing; no platform imports.
11. Q: Why avoid fat components?
    A: Fat components hide business logic and are hard to test/reuse; split into hooks (logic) + presentational components (render only).
12. Q: What is a service locator anti-pattern?
    A: Global registry for dependencies; hides wiring and complicates tests. Prefer explicit DI container or constructor injection.
13. Q: Pros/cons of monorepo for RN + backend?
    A: Pros: shared types, unified tooling, atomic PRs. Cons: heavier CI, coupling risk; require workspaces, clear ownership, caching (turbo/Nx).
14. Q: How to share types between app and backend safely?
    A: Publish versioned shared package (npm workspace); semantic versioning; avoid importing backend code directly; use codegen for API clients.
15. Q: Where to keep feature flags?
    A: Config/remote-config service in Data layer; cache locally; expose typed hooks; evaluate flags in Presentation/Domain logic, not in components directly.
16. Q: How to design plugin-like modules?
    A: Define interfaces/registries for routes/screens; modules register via manifest; dynamic import bundles; contracts enforced via TS types.
17. Q: How to isolate experimental code?
    A: Guard with feature flags + DI; keep behind interfaces; delete branch easily when experiment ends.
18. Q: What is onion architecture difference?
    A: Concentric rings around domain; same rule as Clean Architecture: dependencies point inward. Often identical in practice.
19. Q: CQRS use in mobile?
    A: Split read models (cached/query) from write commands; pairs well with optimistic UI + sync queues; reduces overfetching.
20. Q: Event sourcing in RN—when?
    A: Only when offline auditability is critical (e.g., financial approvals). Store events locally, rebuild projections; higher complexity.
21. Q: Layer for analytics?
    A: Adapter in Data wrapping SDK; Domain defines events; Presentation calls tracker via interface—keeps SDK out of UI logic.
22. Q: Handling app configuration per environment?
    A: Build-time env files for endpoints/ids; runtime remote config for switches; never hardcode secrets; inject into DI container.
23. Q: Multi-brand white-label structure?
    A: Brand config package (theme, assets, routes toggles); DI per brand; avoid code forks; use runtime theme tokens.
24. Q: Advantages of modular navigation stacks?
    A: Teams own isolated stacks; lazy-load routes; tree-shake unused flows; clearer deep link handling.
25. Q: When to use turborepo/Nx for RN?
    A: Multi-package design system/native modules/backend shared contracts; need caching/parallelism; reduces rebuild times.
26. Q: Why keep domain pure TS?
    A: Fast deterministic tests, zero RN/IO coupling, reusable across platforms; simplifies dependency graph.
27. Q: Interface segregation in mobile?
    A: Small, focused repository interfaces per aggregate; consumers depend on minimal surface; fewer mocks and tighter contracts.
28. Q: How to avoid God repository?
    A: Split by aggregate/feature; compose via service layer; lint boundaries; codeowners per repo file.
29. Q: Compose vs inherit in services?
    A: Prefer composition of behaviors (e.g., caching + logging decorators) to avoid fragile inheritance trees.
30. Q: Folder naming for clarity?
    A: `features/<feature>/{screens,components,store,api,types}` plus `shared/`, `services/`, `navigation/`; prevents type-based sprawl and aligns with ownership.
31. Q: How to document architecture?
    A: ADRs for decisions, diagrams (C4), README per feature; keep close to code.
32. Q: What are ADRs?
    A: Architecture Decision Records—short docs logging context, decision, alternatives, consequences.
33. Q: Why avoid circular deps?
    A: Cause runtime bugs and bundler issues; break layering; enforce via lint/module-boundary rules.
34. Q: When to introduce domain events?
    A: For decoupled reactions (e.g., user logged out → clear caches) without tight coupling.
35. Q: Benefits of SOLID in RN?
    A: Testability, maintainability; SRP for components/hooks; DI for repos/services.
36. Q: How to manage build variants?
    A: Separate env files, app ids, icons; config driven; CI matrix; avoid hardcoded switches.
37. Q: Role of TypeScript in architecture?
    A: Enforces contracts between layers; narrows runtime errors; aids refactors.
38. Q: KISS vs over-engineering balance?
    A: Start simple; add layers when scaling/team size demands; measure pain before adding complexity.
39. Q: Handling native module boundaries?
    A: Wrap in adapter layer; expose typed hooks; keep platform checks contained.
40. Q: UX fallbacks when architecture choices fail?
    A: Graceful degradation: skeletons, cached data, retries, offline modes; user messaging.
41. Q: How to audit dependencies?
    A: Regular `npm audit`, `yarn why`, SBOM; pin versions; remove unused.
42. Q: Dependency rule enforcement tools?
    A: eslint-plugin-boundaries, madge, depcruise; CI checks for forbidden imports.
43. Q: Handle design system within architecture?
    A: Shared package with tokens/components; theming provider; versioned releases; avoid per-feature forks.
44. Q: Why keep navigation outside feature logic?
    A: Prevent tight coupling; allows features to be screen-agnostic; easier deep links.
45. Q: Background tasks placement?
    A: Service layer modules; call from effects; abstract platform differences (Headless JS/iOS BG tasks).
46. Q: Where to put error mapping?
    A: Data layer mappers converting transport errors to domain errors/enums.
47. Q: How to ensure testability of architecture?
    A: Interfaces + DI; pure functions; mocked adapters; small units; integration tests per boundary.
48. Q: Code ownership in large RN codebase?
    A: Feature folder ownership + CODEOWNERS; review responsibility; module boundaries to reduce conflicts.
49. Q: When to refactor to modules?
    A: Signals: long build times, merge conflicts, unclear ownership, duplicated logic; plan gradual extraction.
50. Q: KPI to judge architecture success?
    A: Lead time, change failure rate, onboarding time, crash-free sessions, bundle size, TTI.

## 2. State Management (Q51–Q100)
**How to answer:** Bucket data into server/app/UI; map tools (React Query, Redux/Zustand, component state); mention perf guards (memo/selectors), hydration/persistence, and testing strategy.
51. Q: When to choose Redux over Zustand?
    A: Larger teams/needing middleware/devtools/time travel/strict patterns; Zustand for lightweight ergonomic global state.
52. Q: React Query vs Redux Toolkit Query?
    A: Both manage server state; React Query ecosystem-rich; RTKQ integrates with Redux store and codegen.
53. Q: Context API pitfalls?
    A: Re-render storms; hard to split concerns; no middleware; avoid for frequently-changing large state.
54. Q: Prevent Redux performance issues?
    A: Normalize data, memo selectors, slice granularity, `useSelector` with shallowEqual, RTK batching, avoid giant root re-renders.
55. Q: What is Flux pattern?
    A: Unidirectional data flow Action → Dispatcher → Store → View; Redux implements it with pure reducers.
56. Q: Derived vs server vs UI state separation?
    A: Server state via React Query; derived via selectors/memo; local UI with useState/useReducer; avoid duplication.
57. Q: Global error handling pattern?
    A: Error boundary + toasts + query error handlers; map domain errors to user-friendly messages.
58. Q: Handling forms at scale?
    A: Use form libs (React Hook Form/Formik), schema validation (Zod/Yup), controlled/uncontrolled balance, field-level validation.
59. Q: Undo/redo implementation?
    A: Keep history stack in reducer; limit size; apply inverse actions; careful with side effects.
60. Q: Normalization library?
    A: Normalizr or manual map-of-ids; keeps lists fast and deduped.
61. Q: Cross-tab shared state?
    A: Persist to storage + app restart hydration; React Query cache persister; avoid memory-only globals.
62. Q: UI skeleton vs spinner decision?
    A: Skeleton for list/detail; spinner for short blocking actions; match perceived performance expectations.
63. Q: Prevent stale data across tabs?
    A: Query keys include tab/tenant; refetch on focus; invalidate on mutations.
64. Q: How to gate screens on feature flags?
    A: Wrap routes in conditional; gate hooks returning availability; fallback UI; avoid dead code shipping.
65. Q: State for optimistic updates?
    A: Temp entities with client ids + status flags; rollback snapshot; conflict-safe merges.
66. Q: Share business rules between web and RN?
    A: Keep in domain package; publish to npm workspace; import in both apps.
67. Q: When to use useReducer locally?
    A: Complex local transitions, multiple fields; keeps logic testable; avoid global unless needed.
68. Q: Infinite scroll state handling?
    A: Keep pages array + nextCursor; merge uniquely; track loading/error per page.
69. Q: Toasts from anywhere?
    A: Central toast store/service; expose hook; avoid prop drilling.
70. Q: Idle/active session tracking?
    A: AppState + timers; store lastInteraction; logout/lock after threshold; reset on interactions.
71. Q: Theme switching?
    A: Token-based design system; context or Zustand slice; persist preference; respect system setting.
72. Q: Localization state?
    A: i18n library; language in store; resource bundles; fallback keys; memoized formatters.
73. Q: Error boundary scope?
    A: App-level for catastrophic errors; feature-level boundaries for isolating crashes to sections.
74. Q: Avoid prop drilling?
    A: Composition, context for stable values, hooks, collocated state.
75. Q: Derived selectors performance?
    A: Reselect memoization; depend on minimal slices; avoids recompute.
76. Q: Handling rehydration races?
    A: Block rendering until store rehydrates; show splash; use persisted-store callbacks.
77. Q: Authentication state shape?
    A: `status: idle/authenticated/unauthenticated/refreshing`, tokens, user profile, expiration, error; event-driven transitions.
78. Q: Caching forms draft?
    A: Persist drafts per form id with TTL; hydrate on reopen; clear on submit.
79. Q: Batch actions in Redux?
    A: Use `batch()` from react-redux or middleware; reduce renders.
80. Q: Detect and handle conflicts in shared state edits?
    A: Include version; compare before write; show overwrite prompt; merge if possible.
81. Q: Server push updates into state?
    A: WebSocket events → normalizing reducer/query cache updates; debounce to avoid storms.
82. Q: Handling permissions in state?
    A: Permission matrix in store; hooks to gate UI; invalidate on login/role change.
83. Q: What is selector leakage?
    A: Selector depending on broader state causing unnecessary recompute; fix by narrowing inputs.
84. Q: State tests focus?
    A: Reducer pure tests; selector correctness; mutation side effects via integration tests with store.
85. Q: Handling long-running polling?
    A: React Query polling with jitter/backoff; pause when app backgrounded; stop on terminal states.
86. Q: How to avoid cascading re-renders with Context?
    A: Split contexts, memo provider values, use selectors via use-context-selector.
87. Q: Handling stale auth token in state?
    A: Store exp time; refresh before expiry; queue requests during refresh; logout on hard failure.
88. Q: State for file uploads?
    A: Track per-upload progress/status/error; optimistic placeholder; cancel tokens; persisted queue if needed.
89. Q: Multi-tenant state separation?
    A: Include tenantId in keys; clear caches on switch; tenant-aware persistence namespace.
90. Q: Session restoration on app kill?
    A: Persist tokens securely; hydrate on launch; validate/refresh; route to correct stack.
91. Q: Handling modals globally?
    A: Central modal manager store; render portal; queue or stack modals; typed payloads.
92. Q: Avoid duplicate toasts?
    A: Use keyed toasts and dedupe logic; throttle frequency.
93. Q: State for search suggestions?
    A: Cache per query prefix; debounce input; cancel stale requests; show recent history.
94. Q: When to use Jotai/Recoil?
    A: Fine-grained atom-based updates; small/mid apps needing simple mental model and low re-render cost.
95. Q: Manage feature onboarding state?
    A: Store seen tooltips/coachmarks with version; reset when feature changes; persist per user.
96. Q: Animated state vs business state separation?
    A: Keep animation values in Reanimated/shared values; business state in JS store; sync via events.
97. Q: State for presence indicators?
    A: Map of userId→status/lastSeen; updates from WS; expire statuses after timeout.
98. Q: Prevent memory leaks in stores?
    A: Cleanup subscriptions, abort controllers; avoid storing heavy objects/blobs; use weak refs where applicable.
99. Q: Handle large lists statefully?
    A: Virtualization, windowing configs, keyExtractor stability, incremental rendering; avoid storing giant derived arrays.
100. Q: When to reset state on logout?
    A: Clear all slices/caches, cancel queries, drop persistent stores, clear queues, navigate to auth stack.

## 3. Offline-First Architecture (Q101–Q150)
**How to answer:** Describe storage choice, sync loop (pull → push), idempotency keys, conflict policy, connectivity detection, and user freshness signals. Call out battery/data limits.
101. Q: Why offline-first for field workforce?
     A: Unreliable networks; preserves productivity; reduces support tickets; essential for trust.
102. Q: Local storage options?
     A: SQLite/WatermelonDB for relational sync; MMKV for small KV; AsyncStorage for non-sensitive configs.
103. Q: Sync strategies?
     A: Pull-then-push with timestamps; delta sync; or change streams if backend supports.
104. Q: Conflict resolution policies?
     A: Server-wins, client-wins, merge on fields; choose per entity; log conflicts.
105. Q: Idempotency in sync?
     A: Use client-generated ids/opIds; backend dedupes; safe retries without duplicates.
106. Q: How to detect connectivity?
     A: NetInfo; track `isConnected` + `isInternetReachable`; debounce changes.
107. Q: Designing operation queue?
     A: Persisted list of ops with type/payload/idempotency/timestamp; replay FIFO with backoff.
108. Q: Secure offline data?
     A: Encrypt sensitive rows, secure storage for keys, OS-level protection, remote wipe.
109. Q: Handling schema migrations offline?
     A: Versioned migrations, backward-compatible shape, lazy migration on access.
110. Q: Data expiry offline?
     A: TTL per entity; purge stale cache; show freshness indicators to user.
111. Q: Optimistic UI offline?
     A: Show immediate result; flag as pending; reconcile on sync; rollback on fatal errors.
112. Q: Retry policy for sync?
     A: Exponential backoff with jitter; cap attempts; stop on 4xx; resume on reconnect.
113. Q: Partial sync for bandwidth?
     A: Sync by tenant/site/time window; pagination; compression; hashed manifests.
114. Q: Detect divergence?
     A: Server version stamp; compare on pull; trigger full resync if drift detected.
115. Q: Background sync on iOS/Android?
     A: Use BG fetch/processing tasks; Headless JS; respect battery/OS limits; keep work brief.
116. Q: Queue persistence store choice?
     A: MMKV (fast KV) or SQLite table; avoid AsyncStorage for large queues.
117. Q: Attachments offline?
     A: Store files in app sandbox/temp; upload later with resumable transfers; show placeholder.
118. Q: Push-to-sync coupling?
     A: Prefer server change triggers via push/WS to start sync; still allow manual refresh.
119. Q: Version skew handling?
     A: API versioning; clients include app version; server can gate features or force upgrade.
120. Q: Geo-tagging offline events?
     A: Capture GPS with timestamp; store; validate on sync; server checks fences.
121. Q: Data compression?
     A: gzip/brotli on transport; optional row-level compression for large JSON blobs.
122. Q: Battery impact mitigation?
     A: Batch operations, debounce sync, prefer Wi‑Fi, respect Low Power Mode.
123. Q: Clock drift issues?
     A: Use server timestamps; send client clock for diagnostics; avoid relying on device time.
124. Q: Consistency model offline?
     A: Eventual consistency; surface sync status; avoid showing stale as confirmed.
125. Q: Detect duplicate ops on replay?
     A: Idempotency keys and server-side constraints; maintain op ledger client-side.
126. Q: How to pause sync?
     A: User toggle; auto-pause on low battery/roaming; resume with notification.
127. Q: Audit trail offline?
     A: Log operations with metadata; sync logs; server-side aggregation for audits.
128. Q: Handling large initial download?
     A: Preload via partial datasets, progress UI, allow cancel; compress; seed via OTA bundle if static.
129. Q: Conflict UI design?
     A: Show both versions, highlight fields, offer pick/merge; log defaults.
130. Q: Multi-profile devices?
     A: Separate storage namespaces per account/tenant; wipe on logout/switch.
131. Q: Server ordering guarantees?
     A: Include op sequence; server applies in order or per-resource concurrency control.
132. Q: Data integrity checks?
     A: Checksums for payloads; verify after download; retry if mismatch.
133. Q: Background queue visibility?
     A: Expose queue length/status in settings; allow manual flush/retry; logs for support.
134. Q: Handling 413 Payload Too Large?
     A: Chunk uploads; compress; resize images; negotiate limits.
135. Q: Deleting records offline?
     A: Tombstones with deleted flag; sync deletes; purge after confirmation window.
136. Q: Pagination offline cache?
     A: Store pages keyed by cursor; merge uniquely; invalidate on filters change.
137. Q: Testing offline flows?
     A: Unit tests for queue; integration with mocked NetInfo; e2e with network conditioning.
138. Q: Sync for referenced data?
     A: Order sync: reference tables first; validate foreign keys; reconcile missing refs.
139. Q: Binary diff sync?
     A: For big docs; send deltas (rsync-style); rare in RN; weigh complexity vs benefit.
140. Q: What to do on auth expiry offline?
     A: Prevent new writes; queue but mark blocked; prompt re-auth when online.
141. Q: Protect against replay on reconnect?
     A: Idempotency + server timestamps; discard stale ops; signature includes expiry.
142. Q: Multi-device edits?
     A: Use per-record version; last-writer-wins or merge; show recent changes.
143. Q: Handling ordered lists offline?
     A: Store position keys (fractional indices); resolve conflicts via reorder on sync.
144. Q: App size concerns with DB?
     A: Prune old data, use selective sync, vacuum DB; move assets to CDN.
145. Q: GDPR/PII offline?
     A: Minimize stored PII; encrypt; honor erase requests by purging local DB.
146. Q: Observability offline?
     A: Local logs; sync diagnostics; include device state; flags for retries/failures.
147. Q: Fallback if DB locked/corrupt?
     A: Retry with backoff; show recovery prompt; allow full reset with re-sync.
148. Q: How to cap queue growth?
     A: Max length/size; oldest-drop or block new ops; warn user; prioritize critical ops.
149. Q: Sync window selection?
     A: Based on lastSync time; allow manual backfill; server returns delta token.
150. Q: KPIs for offline success?
     A: Sync success rate, time to consistency, queue age, crash-free sessions offline.

## 4. API Design & Integration (Q151–Q200)
**How to answer:** Reliability first (retry/backoff, idempotency, dedupe), pagination choice, caching/ETags, token-refresh single-flight, observability, and limits (429, payload size).
151. Q: Token refresh interceptor concurrency?
     A: Single refresh gate; queue 401s; replay with new token; guard with `_retried`.
152. Q: REST vs GraphQL mobile trade-offs?
     A: REST simple/CDN; GraphQL fewer round-trips, tailored payloads but needs caching/persisted queries.
153. Q: Request deduplication?
     A: In-flight map keyed by URL+params; return same promise; React Query `dedupeInterval`.
154. Q: Caching strategy offline+fresh?
     A: Memory → disk DB; stale-while-revalidate; ETags; background refresh; TTL per resource.
155. Q: Pagination offset vs cursor?
     A: Offset simple but shifts on inserts; cursor stable and efficient—preferred for mobile.
156. Q: Handling retries?
     A: Exponential backoff with jitter; cap attempts; classify retryable status codes.
157. Q: Circuit breaker pattern?
     A: Trip after failures; short-circuit calls; half-open probes; protects backend and battery.
158. Q: API versioning?
     A: Version in path/header; backward-compatible changes; deprecate with sunsets; pin client version.
159. Q: Idempotent writes?
     A: Use PUT/PATCH with ids or idempotency keys; server dedupes; required for retry safety.
160. Q: File upload best practice?
     A: Pre-signed URLs, chunking for large files, progress callbacks, resume tokens.
161. Q: GraphQL caching in RN?
     A: Apollo/Relay normalized cache; policies; persisted queries; optimistic responses.
162. Q: Error taxonomy?
     A: Transport vs domain; map to user-friendly codes; structured error objects.
163. Q: How to secure APIs from app?
     A: Auth tokens, TLS, pinning, replay protection, minimal scopes, rate limiting per device/user.
164. Q: Request timeouts?
     A: Set per call; shorter for foreground; longer for uploads; cancel on unmount.
165. Q: Handling 429?
     A: Respect Retry-After; backoff; surface graceful message; slow down bursts.
166. Q: CORS relevance for RN?
     A: RN uses native networking—CORS not enforced; backend must still validate origins for web clients.
167. Q: Batch requests?
     A: Combine small calls to reduce round trips; server supports batching; consider partial failures.
168. Q: Request cancellation?
     A: AbortController in fetch/axios cancel tokens; cancel on unmount/navigation to avoid leaks.
169. Q: Localization on APIs?
     A: Accept-Language header or user locale param; server returns localized strings/format.
170. Q: Handling clock skew for auth?
     A: Server timestamps; tokens with leeway; avoid strict client clock dependency.
171. Q: WebSocket reconnect strategy?
     A: Exponential backoff with cap, jitter, max retries; resume subscriptions; heartbeat pings.
172. Q: Keep-alive tuning?
     A: Heartbeat intervals balanced with battery; detect missed pongs; close stale sockets.
173. Q: SSE vs WebSocket in RN?
     A: WS better cross-platform with libs; SSE limited on Android; pick WS for reliability.
174. Q: GraphQL subscriptions in RN?
     A: Use WS transport; auth on connectionParams; handle reconnect/resubscribe.
175. Q: Request signing?
     A: HMAC with secret not in app—use token from backend; sign limited-time URLs if needed.
176. Q: Prevent duplicate mutations?
     A: Idempotency keys; client ids; server dedupe table.
177. Q: Large list fetching?
     A: Paginate; server-side filtering/sorting; use cursors; compress responses.
178. Q: DTO mapping?
     A: Map transport DTO to domain models in Data layer; isolate API shape changes.
179. Q: Testing API layer?
     A: Contract tests with mocked server; MSW; integration tests hitting staging; snapshot DTOs.
180. Q: Handling partial failures in batch?
     A: Return per-item status; client retries failed ones; show granular errors.
181. Q: Upload from background?
     A: Use background upload modules/native APIs; ensure auth tokens refreshed; resume after app wake.
182. Q: Prefetching?
     A: React Query prefetch on navigation intent; warm caches; reduce TTI.
183. Q: ETag usage?
     A: Send If-None-Match; server 304; reduces payload; keep etag per resource.
184. Q: Rate limiting client-side?
     A: Throttle user-triggered calls; debounce search; apply token bucket in app for bursts.
185. Q: Multi-tenant headers?
     A: Include tenantId in headers/query; ensure caches keyed per-tenant; avoid leaks.
186. Q: Binary data handling?
     A: Use `responseType: arraybuffer` or `blob`; avoid base64 for large files; stream when possible.
187. Q: API mocking for dev?
     A: MSW/interceptors; stub data; contract fixtures; toggle via env.
188. Q: GraphQL overfetch mitigation?
     A: Lean queries, fragments, persisted queries, field-level resolvers; avoid `__typename` bloat where possible.
189. Q: Handling pagination + filters?
     A: Cursor includes filter params; reset cursor on filter change; invalidate cache.
190. Q: Multi-region endpoints?
     A: Route to closest region; fallback on failure; surface region in telemetry.
191. Q: SLA indicators client-side?
     A: Track latency, failure rates, retry counts; send to observability; alert on regressions.
192. Q: Protect sensitive headers on logs?
     A: Never log auth tokens; redact headers/body; use allowlist logging.
193. Q: Graceful degradation on backend outage?
     A: Serve cached data, show read-only mode, queue writes, surface status banner.
194. Q: API pagination UX?
     A: Infinite scroll with prefetch; show loading indicators; allow manual refresh; preserve scroll.
195. Q: Webhook vs polling for app?
     A: Push/WebSocket preferred; polling backup; keep intervals adaptive.
196. Q: Handling DNS failures?
     A: Retry with backoff; possibly fallback domain; log for support; avoid endless loops.
197. Q: Request size limits enforcement?
     A: Validate before sending; resize images; split payloads; warn user.
198. Q: How to secure GraphQL introspection?
     A: Disable in production or gate by auth; use persisted queries; rate limit.
199. Q: Streaming responses to RN?
     A: Limited; prefer chunked downloads via fetch with response body stream (Hermes supports) or native modules.
200. Q: KPI for API health in app?
     A: P99 latency, error rate, retry rate, cache hit rate, battery impact per call.

## 5. Authentication & Security (Q201–Q250)
**How to answer:** Token lifecycle/storage, transport security (TLS/pinning), device integrity, sensitive flows (biometrics/PIN), least-privilege scopes, and safe logging/audit.
201. Q: Secure JWT storage?
     A: Refresh in Keychain/Keystore; access token in memory/encrypted MMKV; short TTL; never AsyncStorage.
202. Q: Cert pinning need?
     A: For high-sensitivity apps to prevent MITM; pin SPKI/cert; plan rotation.
203. Q: Reverse engineering mitigation?
     A: R8/Proguard, remove debug logs, Hermes bytecode, root/jailbreak detection, integrity checks, keep secrets backend-side.
204. Q: Biometric + PIN flow?
     A: Prompt biometric first; fallback to PIN; PIN hashed + salted in secure storage; throttle attempts.
205. Q: OWASP Mobile Top 10 relevance?
     A: Key risks: insecure storage, comms, auth, code tampering, reverse engineering; audit features against list.
206. Q: Token refresh race handling?
     A: Single-flight refresh; queue requests; replay with new token; logout on refresh fail.
207. Q: Session timeout strategy?
     A: Idle timer; absolute token expiry; renew on activity; warn before logout.
208. Q: Secure deep links?
     A: Validate path/params; require auth; use signed payloads for sensitive actions.
209. Q: Protect API keys in RN?
     A: Avoid shipping secrets; use backend proxy; if unavoidable, restrict scope + domain.
210. Q: Device binding?
     A: Associate device identifier + public key; verify on server; use for step-up auth.
211. Q: Prevent tapjacking?
     A: Use secure flag on Android; detect overlays if critical flows.
212. Q: Screen security?
     A: Set secure window flag to block screenshots on sensitive screens; blur app switcher preview.
213. Q: Key rotation?
     A: Short-lived tokens; rotate refresh keys; backend tracks kid; app fetches new keys via JWKS.
214. Q: Storing passwords?
     A: Don’t store; only transient during login; send over TLS; backend hashes (bcrypt/argon2).
215. Q: SSO in RN?
     A: Use auth sessions with in-app browser/ASWebAuth; handle redirect URIs; PKCE for OAuth.
216. Q: Logout implementation?
     A: Revoke refresh tokens, clear local state/caches, navigate to auth stack, wipe queues.
217. Q: Detect compromised session?
     A: Backend risk scoring; force re-auth; client listens to logout push; clear state.
218. Q: Secure storage library choice?
     A: react-native-keychain or expo-secure-store; avoid generic AsyncStorage for secrets.
219. Q: Handling biometrics availability change?
     A: Listen to OS events; fall back to PIN; re-enroll required; update key binding.
220. Q: Jailbroken/rooted device policy?
     A: Detect and warn/block; log telemetry; balance UX vs security requirements.
221. Q: HTTPS everywhere?
     A: Enforce TLS; disable cleartext; Android network security config; ATS on iOS.
222. Q: Cookie-based auth in RN?
     A: Possible with fetch; but no httpOnly JS access; prefer tokens; manage cookie jar explicitly.
223. Q: Step-up auth?
     A: Trigger biometric/PIN for sensitive actions (payments/settings); backend enforces challenge.
224. Q: MFA flow mobile?
     A: TOTP/SMS/Email; bind device; recovery codes; fallback paths; secure entry UI.
225. Q: Protect local databases?
     A: Encrypt at rest; key from secure storage; avoid sensitive raw PII where possible.
226. Q: How to handle 401 vs 403?
     A: 401 → refresh/login; 403 → show permission error, maybe request access; don’t loop refresh.
227. Q: Avoid storing PII in logs?
     A: Redact fields; disable debug logging in prod; structured allowlist logging.
228. Q: Secure push payloads?
     A: Keep payload minimal/non-sensitive; fetch details in app after auth; use encryption if needed.
229. Q: CAPTCHA in mobile?
     A: Use device attestation (Play Integrity/App Attest) over web captchas; fallback to invisible recaptcha if webview.
230. Q: Fraud/abuse protections?
     A: Rate limiting, device fingerprinting, anomaly detection, step-up auth, server rules.
231. Q: Trusted execution for keys?
     A: Use Secure Enclave/TEE; generate keys on device; never export; sign challenges.
232. Q: Handling lost device?
     A: Remote logout/wipe via push; short token TTL; require re-auth.
233. Q: GDPR rights handling?
     A: Provide data export/delete; ensure local caches purged; respect consent flags.
234. Q: API error leaking info?
     A: Generic messages client-side; detailed logs server-side; avoid stack traces to client.
235. Q: Secure WebViews?
     A: Disable JS unless needed; allowlist origins; disable file access; inject CSP; handle origin swap attacks.
236. Q: Certificate rotation client impact?
     A: If pinning, ship overlapping pins; update via OTA; include backup key hash.
237. Q: Clipboard leakage?
     A: Avoid copying secrets; clear clipboard quickly; warn users.
238. Q: Protect against replay of signed requests?
     A: Nonces/timestamps with short validity; server checks; TLS.
239. Q: Handling biometric lockouts?
     A: Fall back to PIN/password; cooldown messaging; allow account recovery.
240. Q: Secrets in OTA updates?
     A: Never embed secrets; OTA is not secure for secrets; treat bundle as public.
241. Q: Security testing?
     A: SAST/DAST, dependency scans, pentests, jailbreak/root scenarios, intercept proxy tests.
242. Q: Prevent SQL injection in local DB?
     A: Use parameterized queries/ORM; never string-concat user input; validate data.
243. Q: Handling phishing via deep links?
     A: Validate host/path; disallow untrusted schemes; show confirmation for sensitive actions.
244. Q: Non-repudiation mobile?
     A: Sign actions with device key; server logs signature; couple with user identity.
245. Q: Cache headers respect?
     A: Avoid caching sensitive responses; set no-store where necessary.
246. Q: TLS protocol versions?
     A: Enforce TLS1.2+; disable weak ciphers; rely on platform defaults when possible.
247. Q: Session fixation?
     A: Issue new tokens after login/MFA; bind tokens to device; rotate on privilege change.
248. Q: Secure QR scanning flows?
     A: Validate scanned data format; sign important payloads; display confirmation before action.
249. Q: Penetration test findings—hot patch?
     A: Use OTA to mitigate low-risk items; major issues require app store release + backend fixes.
250. Q: KPI for security posture?
     A: Time-to-revoke compromised tokens, security incident count, % endpoints pinned, scan pass rate.

## 6. Performance Architecture (Q251–Q290)
**How to answer:** Frame around TTI/TTFMP; address JS/bridge bottlenecks, lists/images/animations; mention Hermes, code-splitting, measurement tools, and perf budgets in CI.
251. Q: Reducing TTI?
     A: Lazy load screens, code split, prefetch critical data, inline critical fonts, reduce JS bundle.
252. Q: Minimizing re-render frequency?
     A: Memo components, stable callbacks, selector-based state, keyExtractor stability, virtualization.
253. Q: List performance?
     A: FlatList virtualization props, getItemLayout, removeClippedSubviews, window sizes tuned, avoid inline functions.
254. Q: Avoiding bridge chattiness?
     A: Batch calls, move heavy work native side, use JSI/Reanimated for animations.
255. Q: Image optimization?
     A: Use correct sizes, caching (FastImage), WebP/AVIF, placeholders, prefetching, resize server-side.
256. Q: Hermes benefits?
     A: Faster start, lower memory, smaller bytecode; better GC; good for low-end devices.
257. Q: Monitoring performance?
     A: Use perf monitors (Flipper), custom metrics, trace spans around key flows, crash/perf dashboards.
258. Q: Memory leak detection?
     A: Retain cycle checks, remove listeners, cancel timers, avoid large globals; use profiling.
259. Q: JS thread blocking causes?
     A: Heavy loops, JSON.stringify on big objects, sync storage calls, animations not on UI thread.
260. Q: Optimize navigation performance?
     A: Native stacks, enable gesture handler, avoid heavy work on focus; preload next screen data.
261. Q: Prevent overdraw?
     A: Simple backgrounds, avoid nested transparent views, use native driver animations.
262. Q: Handling large JSON parsing?
     A: Streaming parsers or background threads, chunking, avoid deep copies.
263. Q: Measuring TTFMP?
     A: Track from cold start to first meaningful paint; instrument with perf markers.
264. Q: Bundle size control?
     A: Babel plugin for dead-code elim, tree-shake, remove unused locales/icons, dynamic imports.
265. Q: Animation jank fixes?
     A: Use Reanimated/Moti; run on UI thread; avoid setState per frame.
266. Q: Network performance tuning?
     A: HTTP/2, gzip, caching, request batching, avoid chatty polling, prioritize needed fields.
267. Q: Battery impact reduction?
     A: Lower polling, batch work, use background limits, efficient location use, avoid wake locks.
268. Q: Persisted state performance?
     A: Use MMKV for speed; avoid large AsyncStorage payloads; lazy load slices.
269. Q: Stutter during scroll?
     A: Remove heavy shadows, avoid onScroll setState, debounce analytics, use InteractionManager.
270. Q: Native module contention?
     A: Serialize heavy native calls; keep locks short; offload to background threads.
271. Q: Text rendering perf?
     A: Limit custom fonts, cache Text components, avoid nested Text for big lists.
272. Q: Profiling tools?
     A: Flipper, Perf Monitor, Systrace, Android Profiler, Xcode Instruments, React DevTools.
273. Q: Headless JS perf risks?
     A: Overuse drains battery; keep tasks short; respect OS quotas; schedule wisely.
274. Q: GC pressure reduction?
     A: Avoid temporary objects in hot paths; reuse arrays; memoize.
275. Q: JS heap size limits?
     A: Monitor; avoid loading huge datasets; paginate; keep cache capped.
276. Q: Rendering charts efficiently?
     A: Use native chart libs or Skia; memoized props; throttle updates.
277. Q: Startup prefetch boundaries?
     A: Only prefetch essentials; avoid blocking splash; defer low-priority loads.
278. Q: Detect slow frames?
     A: Frame rate monitors; log dropped frames; correlate with actions.
279. Q: App size vs performance trade-off?
     A: Hermes increases size slightly but better runtime; choose minimal dependencies; split APKs.
280. Q: Loading indicators best practice?
     A: Skeletons over spinners; optimistic rendering; avoid blocking UI; show progress for long ops.
281. Q: Offline cache warming?
     A: Preload critical tables after login; run in background; ensure user feedback.
282. Q: Reducing bridge serialization cost?
     A: Avoid passing large objects; use shared memory/JSI; compress if needed.
283. Q: Interaction blocking tasks?
     A: Use InteractionManager to defer non-urgent work until after animations.
284. Q: Feature toggles impact?
     A: Avoid branching deep in render; resolve toggles earlier; strip dead code via build flags.
285. Q: Performance budgets?
     A: Set limits for bundle size, TTI, memory; enforce in CI with regression alerts.
286. Q: Handling slow devices?
     A: Lower animation quality, reduce list window sizes, avoid heavy shadows, fallback UI.
287. Q: Minimizing re-render in forms?
     A: Field-level registration, controlled vs uncontrolled mix, memoized inputs, useController.
288. Q: Native driver vs JS driver?
     A: Use native/UI thread driver for smooth animations; JS driver prone to jank.
289. Q: Warm vs cold start strategies?
     A: Cache critical data, lazy load non-critical modules, avoid heavy sync on launch.
290. Q: KPI for perf?
     A: TTI, TTFMP, P75/95 FPS, crash-free rate, memory footprint, bundle size, battery per session.

## 7. Real-Time Architecture (Q291–Q320)
**How to answer:** WS vs polling, reconnect/heartbeats, ordering + dedupe, backpressure, reconcile with REST, and battery/noise control (collapse keys).
291. Q: Choosing WS vs polling?
     A: WS for bidirectional/low-latency; polling as fallback; SSE limited on Android.
292. Q: Reconnect logic essentials?
     A: Backoff with jitter, max retries, reset on success, detect fatal codes.
293. Q: Handling presence updates?
     A: Heartbeat pings, server timeouts, mark users offline on missed pings.
294. Q: Ordering of events?
     A: Include sequence numbers/timestamps; buffer and reorder; drop duplicates.
295. Q: Backpressure handling?
     A: Limit subscription scopes, debounce UI updates, queue size caps, drop oldest non-critical.
296. Q: Typing indicators design?
     A: Throttled emits; expire after timeout; scoped per conversation.
297. Q: Delivery receipts?
     A: State machine: sent → delivered → read; ack via server; optimistic updates.
298. Q: Offline with WS?
     A: Detect disconnect; switch to queue; resync on reconnect; fetch missed events.
299. Q: Push vs WS for notifications?
     A: Push for wakeup/background; WS for in-app live data; combine for reliability.
300. Q: Secure WS?
     A: wss, token auth on connect, renew tokens, validate origin/tenant, rate limit.
301. Q: Scaling WS backend?
     A: Use gateway + pub/sub (Redis/Kafka), sticky sessions or token-based routing, fanout control.
302. Q: Presence list large scale?
     A: Server aggregation, paginate presence, only send diffs, throttle broadcasts.
303. Q: Live counters (likes/unread)?
     A: Incremental deltas via WS; reconcile periodically via REST; avoid over-broadcasting.
304. Q: Offline message ordering?
     A: Use server timestamps; temp ids; reconcile on sync; handle gaps by fetch missing range.
305. Q: Error handling in real time?
     A: Distinguish fatal/temporary; surface banners; auto-retry with limits; log events.
306. Q: WS connection per feature?
     A: Prefer single connection with multiplexed topics; reduces battery and sockets.
307. Q: Thundering herd reconnect?
     A: Jittered backoff; random delays; limit concurrent reconnects per client.
308. Q: Live location sharing?
     A: Throttle updates, quantize coords, privacy controls, expire shares, TLS.
309. Q: Video/live streaming in RN?
     A: Use native SDKs/WebRTC; handle permissions; adapt bitrate; offload heavy work native side.
310. Q: Data consistency with WS + REST?
     A: Treat WS as hints; REST as source of truth; periodic reconciliation.
311. Q: Handling partial outages?
     A: Failover to polling/push; degrade features; status banner; retry timers.
312. Q: Subscriptions auth refresh?
     A: Reconnect with fresh token before expiry; server may send `token_expiring` event.
313. Q: Monitoring WS health?
     A: Heartbeats, latency metrics, reconnect counts, failure reasons; log to observability.
314. Q: Encrypting WS payloads?
     A: TLS plus optional payload encryption for sensitive topics; key management server-side.
315. Q: Handling multi-device sessions in real time?
     A: Use user+device IDs; deliver to all authorized; dedupe on client; sync read states.
316. Q: Push notification collapse keys?
     A: Collapse similar notifications to reduce noise; keep highest priority/latest payload.
317. Q: Rate limiting emits from client?
     A: Throttle user events; enforce server-side too; drop excessive updates.
318. Q: Offline WS queue size?
     A: Cap length; drop non-critical events; prioritize commands; warn user if full.
319. Q: Retry policy for subscription failures?
     A: Exponential backoff per topic; limit attempts; fallback to polling.
320. Q: KPI for real-time features?
     A: End-to-end latency, message loss/dup rate, reconnect frequency, battery impact.

## 8. Micro-Frontend / Modular Architecture (Q321–Q350)
**How to answer:** Start with “why” (teams/build time), then boundaries (packages, route registry, design system), dependency rules, versioning/rollout, and trade-offs (overhead vs speed).
321. Q: Why modularize RN app?
     A: Parallel development, faster builds, clearer ownership, smaller bundles, optional loading.
322. Q: Techniques for code splitting?
     A: Dynamic import screens, lazy navigation, bundle splitting per feature, CDN-hosted chunks.
323. Q: Shared component library design?
     A: Tokens, primitives, patterns; versioned; no feature logic; themable.
324. Q: Dependency governance in monorepo?
     A: Package boundaries, lint rules, ownership, CI per package, version constraints.
325. Q: Native module sharing?
     A: Publish as packages; podspec/gradle; version lock; avoid app-specific logic.
326. Q: Cross-team contracts?
     A: Type-safe APIs, schemas, semantic versioning, ADRs, consumer-driven contracts.
327. Q: Build time reduction strategies?
     A: Incremental builds, cache (turbo/Nx), selective tests, prebuilt libs, fewer metro restarts.
328. Q: Navigation in modular app?
     A: Route registry; features register screens; host resolves lazily; decouple navigation from features.
329. Q: Asset management modularly?
     A: Feature-local assets; hashed filenames; CDNs; avoid global namespace collisions.
330. Q: Publish/subscribe between modules?
     A: Event bus typed by contracts; avoid deep imports; keep events small and versioned.
331. Q: Avoiding version skew across modules?
     A: Lockstep releases or peer deps; CI to detect divergent versions; renovate bots.
332. Q: Micro-frontend pitfalls in mobile?
     A: App size, perf overhead of multiple bundles, complex navigation; weigh benefits.
333. Q: Feature toggles per module?
     A: Central flag service; modules read flags; tree-shake disabled code where possible.
334. Q: How to deprecate modules?
     A: Mark as deprecated, remove exports, warn consumers, remove routes, clean assets.
335. Q: Testing strategy modular app?
     A: Unit per package, contract tests between packages, e2e covering composed app.
336. Q: API client per module or shared?
     A: Shared core client with injected config; feature-specific API wrappers to avoid duplication.
337. Q: Theming across modules?
     A: Central tokens; ThemeProvider at root; module components consume tokens only.
338. Q: Code ownership conflicts?
     A: CODEOWNERS per package; RFCs for cross-cutting changes; align via guilds.
339. Q: Handling migrations across modules?
     A: Provide upgrade docs, codemods, deprecate old APIs gradually.
340. Q: Lazy native code loading?
     A: Use TurboModules/Codegen; split native libs if possible; otherwise load all native at start.
341. Q: RN + miniapps approach?
     A: Embed multiple bundles; load on demand; manage shared runtime; watch size/perf trade-offs.
342. Q: Error isolation between modules?
     A: Error boundaries per module; sandboxed state; fail-soft fallbacks.
343. Q: Analytics per module?
     A: Standard schema; module adds context props; central pipeline to avoid divergent events.
344. Q: Accessibility consistency?
     A: Shared a11y guidelines/tokens; lint rules; audits per module; central components enforce defaults.
345. Q: Microcopy consistency?
     A: Shared strings library; locale pipeline; avoid duplicate translations.
346. Q: Rollout control per module?
     A: Flags and staged rollouts; OTA partial updates; monitor metrics per module.
347. Q: Dynamic feature delivery?
     A: Android App Bundles split install; iOS on-demand resources (limited); weigh complexity.
348. Q: Security in modular apps?
     A: Shared auth/crypto core; modules cannot bypass; audit dependencies per module.
349. Q: KPI for modularization?
     A: Build time, bundle size per feature, defect isolation, lead time per module, adoption of shared components.
350. Q: When not to modularize?
     A: Small teams/simple app; overhead outweighs benefit; premature modularization slows delivery.

## 9. ERP / Enterprise Case Studies (Q351–Q420)
**How to answer:** Restate scale/constraints, sketch data flow, highlight offline + tenant isolation, conflict/idempotency, security/audit, and manager visibility. End with metrics/KPIs.
351. Q: Design attendance module offline-first?
     A: Local queue + DB, geo fence, biometric check, sync with idempotency, manager dashboards with WS.
352. Q: Payroll calculator offline?
     A: Ship rules tables, compute on device, queue adjustments, reconcile server, audit trail.
353. Q: Multi-company tenant isolation?
     A: TenantId in headers/cache, per-tenant DB/schema, isolate storage namespaces, RBAC per tenant.
354. Q: Inventory scanning app?
     A: Offline barcode scans queued, conflict resolution on stock counts, fast local search, batch sync.
355. Q: Leave management?
     A: Client-side validation of balances, optimistic requests, approvals workflow, push notifications for status.
356. Q: Expense capture?
     A: Offline receipts stored locally, image compression, OCR async, policy checks, approvals.
357. Q: Timesheet app?
     A: Local draft per day, auto-save, submit batch, manager approvals, conflict detection on overlapping time.
358. Q: Visitor management?
     A: Pre-registered QR codes, offline verification cache, photo capture, host notifications via push/WS.
359. Q: Asset tracking?
     A: Tag scans cached, geolocation, sync deltas, anomaly alerts, role-based visibility.
360. Q: Compliance training app?
     A: Downloadable course modules, offline quizzes, sync scores, certificates, progress tracking.
361. Q: Field service work orders?
     A: Offline job list, attachment uploads, status transitions with idempotency, parts inventory checks.
362. Q: Internal chat for factories?
     A: WS real-time chat, offline queue, image compression, presence, read receipts.
363. Q: Shift scheduling?
     A: Rule-based validation, conflict detection, drag-drop UI, notifications, manager overrides logged.
364. Q: Incident reporting?
     A: Offline capture with media, geo-tag, severity triage, push escalation, audit logs.
365. Q: Audit trail requirements?
     A: Log user/action/time/device; immutable server log; synced offline logs; exportable.
366. Q: Role-based dashboards?
     A: Feature flags + RBAC; backend filters data; avoid client-side only gating.
367. Q: Data residency constraint?
     A: Route tenant traffic to region; store data regionally; feature gating per region.
368. Q: Integrating legacy SOAP backend?
     A: Backend adapter/service layer; convert to REST/GraphQL; client sees consistent contract.
369. Q: SLA monitoring in app?
     A: Telemetry for latency, failures; status banners; retry/backoff; circuit breakers.
370. Q: Disaster recovery client impact?
     A: Read-only mode, cached data, queued writes, status communication, forced logout if data at risk.
371. Q: ERP search performance?
     A: Server-side search with pagination, debounced queries, cached suggestions, offline recent searches.
372. Q: Large form performance?
     A: Sectioned forms, virtualization of fields, deferred validation, autosave drafts.
373. Q: Data export feature?
     A: Trigger server export; notify via push when ready; download link; stream with resume.
374. Q: KPI dashboards live?
     A: Aggregate server side; push deltas via WS; cache; fall back to polling.
375. Q: Integrating SSO (SAML)?
     A: Use in-app browser; backend token exchange to mobile tokens; handle session renewal.
376. Q: GDPR user deletion?
     A: Purge local caches, drop user namespaces, request backend deletion, log completion.
377. Q: Multi-language ERP?
     A: Central i18n with locale bundles; server sends localized data; right-to-left support; date/number formatting.
378. Q: Attachment scanning for viruses?
     A: Upload to backend scanning service; hold until clean; mark status to client.
379. Q: Notifications hierarchy?
     A: Topics per feature/tenant/site; priority levels; quiet hours; inbox + push.
380. Q: Approvals workflow design?
     A: State machine with transitions, roles, delegation, audit logs, SLA timers.
381. Q: Data minimization for privacy?
     A: Collect least necessary, mask sensitive fields, expire data, encryption, access logging.
382. Q: Tenant admin tooling in app?
     A: Admin-only routes; feature flag; audit actions; scoped APIs; confirm dangerous actions.
383. Q: KPI for adoption?
     A: DAU/WAU, task completion, offline success rate, crash-free sessions, latency.
384. Q: Custom reports?
     A: Parameterized report requests; backend generates; app fetches results; cache per filter set.
385. Q: Email/SMS fallback when push disabled?
     A: User comm preferences; send via backend; app inbox mirrors state; respect opt-outs.
386. Q: Handling huge org charts?
     A: Lazy load tree, search, cache nodes, collapse levels, server pagination.
387. Q: Attendance spoofing prevention?
     A: Geo fence, Wi‑Fi/Bluetooth beacons, selfie check, device binding, anomaly alerts.
388. Q: BYOD security controls?
     A: MDM compatibility, jailbreak detection, selective wipe, minimal local PII, strong auth.
389. Q: Audit export limits?
     A: Pagination, date range limits, async export, rate limiting, signed URLs.
390. Q: Feature rollout per site?
     A: Config service keyed by site/tenant; flags; staged rollout; monitor metrics per site.
391. Q: Handling shift handover notes?
     A: Offline drafts, timestamped entries, attachments, search, edit history.
392. Q: ERP offline reports?
     A: Cache recent reports; show freshness; allow manual refresh; mark stale when outdated.
393. Q: KPI for reliability?
     A: Error rate, sync success, uptime of critical APIs, queue backlog, reconciling discrepancies.
394. Q: Custom branding per tenant?
     A: Theme tokens + assets from config; cache; apply at app start; avoid code forks.
395. Q: Accessibility in enterprise app?
     A: WCAG AA, large text support, screen reader labels, focus order, high contrast themes.
396. Q: Supervisory overrides?
     A: Elevated actions require secondary auth/logging; audit trails; notify affected users.
397. Q: Integrating IoT sensors?
     A: Backend aggregates sensor data; app subscribes; avoid direct device connections; show alerts.
398. Q: Enterprise search relevance?
     A: Weighted fields, synonyms, typo tolerance, personalization; server-driven ranking.
399. Q: Handling duplicates in data entry?
     A: Client pre-checks, server uniqueness constraints, fuzzy match warnings, merge flows.
400. Q: Bulk operations UX?
     A: Multi-select, preview changes, batch API, progress + partial failure handling, undo if possible.
401. Q: Audit-proof approvals?
     A: Signed actions with device/user/time, immutable logs, read-only history.
402. Q: Live floor headcount display?
     A: WS/subscriptions from backend, cached fallback, dedupe updates, presence timeouts.
403. Q: Cross-tenant data leak prevention?
     A: Tenant scoping everywhere; cache keys include tenant; strict server auth; tests for leaks.
404. Q: Legacy database consistency?
     A: Backend sync jobs; API abstracts legacy; client sees unified model; handle eventual consistency.
405. Q: Handling daylight savings in timesheets?
     A: Store UTC, display in local, handle DST transitions explicitly, prevent overlap gaps errors.
406. Q: Attachments size policy?
     A: Limit size, compress, resize images, chunk upload, enforce MIME allowlist.
407. Q: Offline approvals?
     A: Queue decisions with signatures/time; sync with idempotency; mark pending in UI.
408. Q: Tenancy switch UX?
     A: Clear caches, reload config/theme, show indicator; require explicit confirmation.
409. Q: Data archival policies?
     A: Move old records to cold storage; app fetches on demand; mark archived state.
410. Q: Real-time SLA alerts?
     A: Server monitors; push/WS to managers; in-app banner; actionable links.
411. Q: Integrating BI dashboards?
     A: Embed web views with SSO tokens or native charts fed by APIs; cache; secure tokens.
412. Q: Payment processing in ERP?
     A: Use PSP SDKs; PCI scope minimal; tokenization; strong auth; audit logs.
413. Q: Offline license checks?
     A: Cache license with expiry; grace period; re-validate when online; disable after expiry.
414. Q: Handling massive tenant data sets?
     A: Server-side pagination/filtering, delta sync, search endpoints, avoid full downloads.
415. Q: SLA for sync?
     A: Define target latency; measure; alert on breach; adaptive sync frequency.
416. Q: Smart defaults UX?
     A: Prefill from last used/site context; reduce input; allow override; track adoption.
417. Q: Non-functional requirements capture?
     A: Latency, uptime, offline, security, compliance; include in design doc; test for them.
418. Q: Disaster drills?
     A: Simulate API outages; verify read-only mode, queues, user messaging; track MTTR.
419. Q: Data ownership disputes?
     A: Clear audit trails, role-based visibility, approvals; immutable logs for edits/deletes.
420. Q: KPI for enterprise success?
     A: Task completion rate, sync reliability, crash-free sessions, P75 latency, support tickets volume.

## 10. System Design Interview Framework (Q421–Q500)
**How to answer:** Use a fixed template: clarify → scale → high-level → deep dive → failures → security → rollout → metrics → recap. Timebox and park tangents.
421. Q: First 5 minutes of interview?
     A: Clarify requirements, users, scale, constraints, success metrics; restate scope.
422. Q: How to estimate scale quickly?
     A: Ask user counts, DAU/MAU, RPS, data size; rough orders of magnitude.
423. Q: Draw high-level diagram components?
     A: Client, API gateway, services, DB, cache, queue, CDN, auth, analytics.
424. Q: Choosing state management in design answer?
     A: Map data types to tools (React Query/Zustand/Redux); justify with scale and team size.
425. Q: Handling trade-offs explicitly?
     A: State what you sacrifice (simplicity vs flexibility, latency vs cost); why chosen.
426. Q: Timeboxing answer?
     A: Allocate: clarify (3m), high-level (5m), deep dive (15m), trade-offs (5m), recap (2m).
427. Q: Deep dive pick?
     A: Choose one critical area (auth, sync, real-time) based on interviewer hint; go deep.
428. Q: Functional vs non-functional requirements?
     A: Functional = features; non-functional = latency, availability, security, offline, cost.
429. Q: Capacity planning for backend?
     A: Estimate RPS, payload, storage; choose DB/indices; caching; autoscaling targets.
430. Q: Handling “what if it scales 10x”?
     A: Show sharding/partitioning, caches, queues, read replicas, horizontal scaling.
431. Q: When to say no?
     A: Highlight trade-offs; propose staged approach; avoid gold-plating in 45 minutes.
432. Q: How to show mobile constraints?
     A: Mention latency, offline, battery, bundle size, platform differences, app store release cycles.
433. Q: Metrics to mention?
     A: Crash-free %, TTI, latency P95, sync success, DAU/MAU, churn, battery impact.
434. Q: Security talking points?
     A: AuthN/Z, token storage, pinning, encryption, rate limiting, audit logs, privacy.
435. Q: Testing strategy in design answer?
     A: Unit, integration, contract, e2e, performance/load tests, chaos drills.
436. Q: Rollout strategy?
     A: Staged rollout, feature flags, canary, OTA, monitoring, rollback plan.
437. Q: Handling failures?
     A: Retries with backoff, circuit breakers, idempotency, fallbacks, graceful degradation.
438. Q: Data model discussion?
     A: Key entities, relationships, indexes, sample schemas; how they map to UI flows.
439. Q: API design mention?
     A: REST/GraphQL choice, versioning, pagination, error contracts, rate limits.
440. Q: Caching layers to call out?
     A: Client cache, CDN, API cache, DB cache (Redis), per-tenant TTLs.
441. Q: Queue usage justification?
     A: For async tasks, smoothing spikes, retries, ordering; examples: notifications, sync.
442. Q: Consistency model mention?
     A: Eventual vs strong; per entity justification; user-facing implications.
443. Q: Back-of-envelope latency calc?
     A: Network RTT + processing + DB; show awareness of mobile constraints.
444. Q: Storage estimation?
     A: Rows * size * retention; include indices and attachments; plan archiving.
445. Q: Multi-region discussion?
     A: Latency, data residency, failover strategy, DNS, replication lag, conflicts.
446. Q: Observability?
     A: Logs, metrics, traces; client telemetry; dashboards; alerting; sampling.
447. Q: Privacy-by-design points?
     A: Minimize data, consent, encryption, retention limits, access controls, audits.
448. Q: Handling “build vs buy” question?
     A: Evaluate time, expertise, cost, compliance; pick proven services for non-core.
449. Q: Threat modeling brief?
     A: Identify assets, actors, entry points; mitigations; mention STRIDE-lite.
450. Q: Mobile release constraints?
     A: Store review delays, OTA limits, staged rollout; affects incident response.
451. Q: Offline requirements in design answer?
     A: Local cache, queue, conflict policy, sync strategy, connectivity detection.
452. Q: Real-time requirements mention?
     A: WS/subscriptions, reconnect logic, ordering, fallbacks, fanout control.
453. Q: Cost considerations?
     A: CDN to reduce egress, caching to cut DB load, right-size instances, avoid over-fetch.
454. Q: API evolution plan?
     A: Versioning, deprecation policy, backwards compatibility, contract tests.
455. Q: Security for PII/PHI?
     A: Encryption in transit/at rest, access controls, audit logs, masking, region residency.
456. Q: Blue-green vs canary?
     A: Blue-green = full env switch; canary = small % traffic; mobile often uses canary + flags.
457. Q: Handling migrations?
     A: Backward-compatible schema, dual writes, data backfill, feature gating, monitoring.
458. Q: DR strategy you’d mention?
     A: Multi-AZ, backups, RPO/RTO targets, runbooks, chaos drills.
459. Q: Performance budget talk?
     A: Set KPIs; instrument; regressions blocked in CI; budgets for TTI/bundle size.
460. Q: Pitfalls to avoid in interview?
     A: Skipping requirements, no trade-offs, ignoring security/ops, diving too deep too soon.
461. Q: Whiteboard structure?
     A: Client → API → Services → DB/Cache → Queue → Observability; annotate decisions.
462. Q: How to answer “what happens on failure”?
     A: Describe detection, retries, fallbacks, user messaging, eventual consistency.
463. Q: Data migration with zero downtime?
     A: Expand/contract, dual reads/writes, background backfill, cutover, cleanup.
464. Q: Rolling keys/tokens?
     A: JWKS rotation, short TTL, kid headers, client refresh, overlapping keys.
465. Q: CDN usage in RN?
     A: Static assets, images, config files; signed URLs; versioned paths.
466. Q: Feature flag safety?
     A: Kill switches, default safe state, audits, targeted rollout, cleanup old flags.
467. Q: Handling dependencies risk?
     A: Pin versions, SCA, review licenses, shrinkwrap, minimal deps.
468. Q: App telemetry minimal set?
     A: Screen views, key actions, latency/errors, device info (non-PII), version/build.
469. Q: Blueprints for auth flow to mention?
     A: Login → token storage → refresh → logout → lock screen; biometrics optional; secure storage.
470. Q: Load testing relevance to mobile?
     A: Ensures backend handles bursts; simulate realistic payloads; informs client retry/backoff.
471. Q: API pagination choice reasoning?
     A: Cursor for large/real-time; offset for simple small datasets; explain impact on mobile UX.
472. Q: Handling secrets in CI/CD?
     A: Store in vault, inject as env, never commit; restrict access; rotate.
473. Q: Code quality signals to mention?
     A: Lint, tests, type coverage, bundle size checks, crash-free metrics.
474. Q: Incident response steps?
     A: Detect (alerts), triage, mitigate (flags/rollbacks), communicate, postmortem, action items.
475. Q: Data retention you’d propose?
     A: Based on compliance; purge old logs/data; client cache TTLs; user-controlled deletion.
476. Q: Handling third-party SDK risk?
     A: Review permissions, size, perf; wrap in adapter; lazy load; kill switch.
477. Q: Global search design outline?
     A: Indexed search service, autocomplete, ranking, filters, pagination, analytics; cache queries.
478. Q: SLA commitments discussion?
     A: Uptime %, latency, support response; design redundancy; monitoring to meet SLAs.
479. Q: KPI for success of design?
     A: Meets requirements; metrics targets hit; maintainable; operable; secure; happy users.
480. Q: Techniques to simplify answer?
     A: Pick one deep dive, park tangents, use numbered trade-offs, keep diagrams simple.
481. Q: Managing time in interview?
     A: Watch clock; move on if stuck; prioritize big rocks; summarize frequently.
482. Q: Dealing with unknowns?
     A: State assumptions, choose defaults, propose validation/experiments.
483. Q: Handling interviewer pushes?
     A: Accept, adapt design, narrate trade-offs; don’t be defensive.
484. Q: Testing mobile-specific issues?
     A: Device matrix, network conditioning, background/foreground, low memory, battery tests.
485. Q: Compliance mentions?
     A: GDPR/PII storage, data residency, audit logs, encryption, access control.
486. Q: How to present trade-off tables?
     A: Use quick 2-column verbal: choice vs trade-off; helps clarity.
487. Q: Postmortem essentials?
     A: Timeline, impact, root causes, what went well/poorly, action items with owners/dates.
488. Q: Caching invalidation talking point?
     A: Invalidate on writes, time-based TTL, versioned keys, tenant-aware; hardest problem—acknowledge it.
489. Q: Partitioning strategy outline?
     A: By tenant/region/user id; pros: isolation/scaling; cons: cross-shard queries/joins.
490. Q: Rate limiting approaches?
     A: Token bucket/leaky bucket; per user/device/IP; return Retry-After; client backoff.
491. Q: API pagination UX gotcha?
     A: Maintain position on refetch, preserve filters, handle empty states gracefully.
492. Q: Observability on client?
     A: Structured logs, traces for critical flows, crash reporting, ANR monitoring, sample to reduce noise.
493. Q: Security logging?
     A: Log auth attempts, privilege changes, admin actions; avoid sensitive data.
494. Q: Data consistency wording?
     A: “Eventual consistency acceptable for feed; strong consistency for payments/auth.”
495. Q: Handling P0 outage in mobile?
     A: Flip kill switch/maintenance mode, serve cached data, push status, start rollback, keep comms open.
496. Q: Mobile vs web differences to mention?
     A: Offline, battery, network variability, store releases, smaller screens, native capabilities.
497. Q: Memory constraints statement?
     A: Design for low-end devices; avoid huge in-memory caches; use pagination/streaming.
498. Q: How to close the interview?
     A: Recap design, reiterate trade-offs, mention monitoring/rollout, invite questions.
499. Q: What to say if you’d do more with time?
     A: List next steps: load tests, POCs, security review, usability tests.
500. Q: Personal experience tie-in?
     A: Relate to your past project, what you learned, and how you’d apply it here.
