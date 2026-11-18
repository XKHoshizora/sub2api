# Maintained Railway deployment

This branch integrates upstream main at 2af3f1e15c98c893a89255a037f551046883b8c0 (VERSION 0.2.23) from deployed maintenance commit 2e377c14f8047db7ca0a17ff832ab22fbfe19140. It retains GPT HTTP cancellation, checked writes and flush errors, response outcomes, observed-consumption accounting, timing/correlation, routing-capacity service alerts, and managed non-root container update protection.

The merge keeps cancellation and observed-usage accounting while applying upstream coalesced raw-chat flushes, response-model audit metadata, bounded idle failover, GPT-6.2 support and cache-usage fixes. Coalesced flushes still check actual downstream flush errors and cannot silently retry a billed attempt.

The deployed Docker build, go.mod and CI use Go 1.27.4, including the net/http fix for GO-2026-6859. This retains the deployed toolchain alignment and matches current upstream.

This release adds group Ultrafast policy (default false), optional channel Ultrafast pricing (default NULL), and nonblocking group-usage-rollup build state. It replaces rollup invalidation functions without rewriting usage rows or balances. Review rollup build/watermark state if a binary rollback is needed; retain the added fields and assess function compatibility separately. Existing migration files, including response-outcome and cache-write migrations, are unchanged. Do not run downgrade SQL or restore snapshots automatically. Verify migration checksums, pricing defaults and core configuration identities after deployment.

The runtime is an immutable non-root container. The existing /app/data mount remains in use. Version checks stay available; direct binary updates and official-binary rollback remain blocked with DEPLOYMENT_MANAGED_UPDATE so they cannot overwrite maintained fixes.

## Upgrade procedure

1. Fetch a specific upstream release tag and merge it into this maintained branch. Resolve conflicts without discarding the GPT delivery/accounting patch.
2. Update `backend/cmd/server/VERSION` and the Dockerfile commit identifier. Review dependency and migration changes.
3. Run managed-update tests, GPT delivery/cancellation/usage regressions, upstream security regressions, frontend locale/type/build checks, and the backend build. Real model calls require a separate bounded budget.
4. Make a Railway backup of the existing PostgreSQL and application volumes; record the current successful deployment as the rollback target.
5. Push the reviewed commit to the connected maintained branch. Railway builds the Dockerfile from that source. If deploying an already-pushed revision manually, use **Deploy Latest Commit**, not a redeploy that only restarts the previous uploaded source.
6. Wait for the exact new deployment to succeed. Verify `/health`, the public runtime version, managed update guidance, and unchanged account/key/configuration identities. Do not publish based only on upload success.
7. If acceptance fails, roll back to the previous successful **patched** Railway deployment. Do not point the service at the unpatched official image. Retain additive usage-log columns; no destructive database rollback is needed.

This maintenance release uses `[skip ci]` after completing the bounded local acceptance suite, avoiding duplicate Actions runs. Existing workflow permissions are unchanged. Do not activate upstream release workflows or push upstream-shaped release tags just to ship this branch. Railway deployment does not require GitHub Actions; record acceptance for every maintenance release.

The Docker cache IDs are scoped to the existing Railway Sub2API service. Other installations must replace that prefix. No credentials or customer data belong in this repository.
