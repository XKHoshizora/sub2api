# Maintained Railway deployment

This branch integrates upstream main at 530c5e3a4f9c73ca2b0881191d0ef8e0d82c5484 (VERSION 0.2.21). It retains GPT HTTP cancellation, checked writes and flush errors, response outcomes, observed-consumption accounting, timing/correlation, and managed non-root container update protection.

The passthrough merge keeps linked downstream cancellation and usage-preserving partial results while applying upstream instructions echo handling and empty-instructions support. Upstream resource budgets, buffered reasoning fixes, image routing and MCP support remain in place alongside checked delivery, bounded Grok model audits and endpoint-specific image billing.

The deployed Docker build uses Go 1.27.4 to include the net/http security fix for GO-2026-6859. The upstream go.mod minimum remains 1.27.3; validation of the deployed backend uses the explicit patched toolchain. Other upstream build/release configurations remain unchanged.

This release adds a concurrent covering index for API-key usage costs and migration 250 for OpenAI cache-write token lifetimes and optional 12h cache-write prices. The index can consume I/O during startup; verify that it is valid after migration. New token columns default to zero and new prices remain NULL, preserving the existing fallback pricing. Existing migration files, including the response-outcome and MCP OAuth migrations, are unchanged. Retain the additive columns, index and data volumes on binary rollback; do not run downgrade SQL or restore snapshots automatically. The prior maintained binary ignores these new fields. Verify migration checksums and core account/key/configuration identities after deployment.

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
