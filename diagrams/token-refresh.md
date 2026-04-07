# Token Refresh Single-Flight Flow

```
Request → 401?
   │
   ├─ No → proceed
   │
   └─ Yes → isRefreshing?
            │
            ├─ Yes → queue resolver
            │
            └─ No  → set isRefreshing
                    ↓
              refreshAccessToken()
                    │ success
                    ▼
          flush queue with new token
                    │
                    ▼
          replay original request
                    │
                    ▼
               isRefreshing = false
```

**Key points**
- Mark retried requests to avoid loops (`config._retried = true`).
- Queue holds callbacks that resolve/reject pending Promises.
- On refresh failure: clear queue, logout, show session-expired UI.
- Store refresh token in Keychain/Keystore; access token in memory/encrypted MMKV.
