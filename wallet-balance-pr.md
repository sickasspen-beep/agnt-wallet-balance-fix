# Fix wallet balance reads for authenticated lapsed accounts

Authenticated Privy users whose account status is `cancelled` or `payment_failed` can access `/api/me` but receive a 401 from `/api/wallet/balance`. That route's optional-auth middleware drops their verified identity because it also checks billing status. The chat sidebar then displays `No holdings yet`, and a successful history response can produce a false 100% loss.

This change makes wallet balance requests with credentials use the same authentication check as `/api/me` and `/api/wallet/net-curve`. Requests without credentials retain public address lookups; invalid supplied credentials and invite-gated accounts remain denied. The shared optional-auth middleware is unchanged, preserving other routes' eligibility behavior.

The sidebar now displays balance-load failures explicitly, hides percentage changes and charts when no balance data is available, and retains previously loaded holdings with a stale-data warning if a refresh fails.

## Validation

- 81 focused tests pass after rebasing onto `main` at `66fc777a`.
- New tests exercise the actual authentication middleware and registered balance route, plus the wallet JSX for initial failures, cached-data failures, empty responses, loading, and desktop/mobile header states.
- Server syntax and diff whitespace checks pass.
- Full backend suite, frontend typecheck/build, and staging verification have not run locally. CI should run these before merge. The clone has no installed project dependencies; the focused JSX tests used an already installed TypeScript compiler via `NODE_PATH`.

```sh
node --test tests/wallet-balance-auth.test.js tests/chat-wallet-error.test.js tests/wallet-invalidation-graph.test.js tests/wallet-refresh-bus.test.js
node --check server.js
```

No billing status or shared authentication policy is changed. Opening this PR does not deploy; merging the backend change to `main` uses the repository's existing deployment workflow.
