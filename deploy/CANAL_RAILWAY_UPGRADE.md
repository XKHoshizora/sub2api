# Maintained Railway deployment

This branch integrates upstream v0.2.17 and main at b0a02a3304a0a78cdd003ddda1dd23ba02c03084.
It preserves GPT HTTP cancellation, checked response delivery and flush errors, response outcome auditing, and exactly-once accounting of observed supplier consumption. Managed container update protection, the non-root runtime and the existing clock-resolution test fixtures remain intact.

Upstream adds Responses created_at normalization, subscription-model retirement checks after public alias projection, payment/refund correctness fixes, workspace-isolated Codex imports, and Gemini/Antigravity compatibility improvements. There are no new SQL migrations since the maintained v0.2.16 release. The existing 242_add_usage_log_response_outcome.sql is byte-for-byte unchanged; migrations 242 through 245 already present in production are retained on binary rollback.

The streaming conflict keeps upstream createdAtTracker.Value() and the maintained markClientGone(err). Its regression verifies that a failed synthesized response.failed write closes the supplier body, preserves observed input usage, reports delivery failure and does not trigger account failover. The maintained request cancellation and accounting paths match the refreshed upstream PR implementation.

The runtime is an immutable, non-root Railway container. /app/data stays on the existing volume. Website version checks remain available; in-place updates and official-binary rollback stay blocked with DEPLOYMENT_MANAGED_UPDATE so custom fixes cannot be overwritten.

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
