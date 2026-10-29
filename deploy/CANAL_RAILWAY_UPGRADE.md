# Maintained Railway deployment

This branch integrates upstream main at a3298b46ebe239fb0720b068a1eb3383955bdac2 (VERSION 0.2.19). It retains GPT HTTP cancellation, checked writes and flush errors, response outcomes, observed-consumption accounting, timing/correlation, and managed non-root container update protection.

The non-streaming Responses paths combine upstream SSE-to-JSON fixes, including missing Content-Type and incomplete/cancelled terminal events, with checked downstream writes and consumption-preserving delivery errors. Aggregated JSON explicitly replaces the upstream SSE Content-Type. The existing bounded Grok model audit, cancellation contexts, image-endpoint billing boundaries, and queued/metadata retry behavior remain in place.

There are no new SQL migrations since the maintained 0.2.18 deployment. All existing migration files, including the response-outcome migration, are unchanged. Keep the database and data volumes on binary rollback; do not run downgrade SQL or restore snapshots automatically. Scoped administrator keys and image-count billing were introduced in 0.2.18 and remain available.

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
