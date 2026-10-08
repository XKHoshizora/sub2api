# Maintained Railway deployment

This branch is based on upstream v0.2.14 (0363b8cdba8cec3e2ba4b2dbd49c4481143fa55d).
It preserves the reviewed GPT HTTP cancellation, checked response delivery, and confirmed-usage accounting fixes from the previous 0.2.13 deployment.

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
