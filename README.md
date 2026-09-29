# AGNT wallet balance fix — maintainer audit

Proposed fix for authenticated users whose wallet balance fails to load on agnt.social. This repository contains a patch and review notes for the maintainer of [tuthouse/ClawMultiAgent](https://github.com/tuthouse/ClawMultiAgent).

## Files

- [wallet-balance-fix-commit.patch](wallet-balance-fix-commit.patch) — Git commit patch, including the backend change, frontend error handling, and regression tests.
- [wallet-balance-pr.md](wallet-balance-pr.md) — problem description, implementation summary, and validation details.

The patch was prepared against upstream commit `66fc777ae0d4822d4496cab8562b11909022a5ac`. The proposed fix commit is `89512d9aa4d0c90acb9420c5cc34204244581591`.

## Problem and proposed fix

The wallet balance endpoint uses optional authentication that discards verified Privy users unless their account status is `active`. An account marked `cancelled` or `payment_failed` can therefore access `/api/me` while receiving HTTP 401 from `/api/wallet/balance`.

The patch makes authenticated wallet balance requests use the same verification as `/api/me`. Anonymous address lookups remain available, and the shared optional-auth middleware stays unchanged. The sidebar displays failed balance requests explicitly, retains cached holdings with a warning on refresh failure, and hides misleading percentage changes when no balance data is available.

## Audit instructions for the maintainer or their agent

Review the patch in an authorized checkout of the upstream repository. The complete application source and dependencies are required; this handoff repository does not run the application by itself.

1. Compare the patch with current upstream authentication, subscription policy, and wallet behavior. Check for intervening changes since the base commit.
2. Apply it to a review branch in the upstream checkout:

   ```sh
   git switch -c codex/audit-wallet-balance-fix
   git am /path/to/wallet-balance-fix-commit.patch
   ```

3. Run the focused checks with upstream development dependencies installed:

   ```sh
   node --test tests/wallet-balance-auth.test.js tests/chat-wallet-error.test.js tests/wallet-invalidation-graph.test.js tests/wallet-refresh-bus.test.js
   node --check server.js
   ```

4. Run the upstream CI suite, frontend typecheck/build, and a staging check. Verify valid active and lapsed accounts, anonymous address lookups, invalid credentials, invite-gated users, and initial/background balance-loading errors.
5. Report findings and any required corrections before deciding whether to merge.

## Verification status

The author ran 81 focused tests successfully against the referenced base, plus server syntax and whitespace checks. The new regression tests were observed failing before the fix. An independent review identified a false-loss display, which was corrected with regression coverage.

Full application build, full backend suite, and staging verification have **not** been completed locally. The JSX tests execute extracted application JSX with test doubles; they are not a browser integration test. Provider failures returned inside an HTTP 200 response are outside this patch's request-error handling.

This is a proposal for audit. It has not been merged or deployed to agnt.social. Publication here does not change the upstream repository or any account's billing status.
