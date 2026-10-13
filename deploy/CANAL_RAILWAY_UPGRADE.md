# Maintained Railway deployment

This branch includes upstream v0.2.16 and main at 4d9ac3e482106d2da3a39fe3e546d8d6801f1cc1.
It preserves the reviewed GPT HTTP cancellation, checked response delivery, and confirmed-usage accounting fixes from the previous 0.2.13 deployment.

The existing `242_add_usage_log_response_outcome.sql` remains byte-for-byte unchanged. Upstream adds `242_group_i18n.sql` and `243`–`245` Gemini cached-content migrations: additive columns, a new table and indexes. Migrations are tracked by full filename, so the shared `242` prefix does not collide. Binary rollback retains completed migrations. Gemini cached contents created after this upgrade may keep incurring upstream storage charges until expiry; review and settle those resources before rolling back to a binary without cache management.

The ops error logger merge keeps both the existing delivery/consumption audit message and upstream Google-format local-error classification. Upstream Responses-to-Chat fallback and long-context pricing changes preserve the maintained cancellation and observed-usage gates.

The HTTP lifecycle regression now verifies the maintained cancellation/observed-consumption behavior, using the existing fork regression from `c8a2573dba4f293eab8f03a02474c185492023af`. The Ollama stale-probe fixture uses a reset two seconds earlier than the original reset, so low-resolution clocks cannot accidentally make the old and new generations equal. No production Ollama code changes are included.

The runtime is an immutable, non-root Railway container. `/app/data` stays on the existing volume. Website version checks remain available; in-place update and official-binary rollback are blocked with `DEPLOYMENT_MANAGED_UPDATE` (HTTP 409) so custom fixes cannot be silently overwritten.

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
