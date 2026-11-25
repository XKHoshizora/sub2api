# Maintained Railway deployment

This branch integrates upstream 0.2.25 at 0f6671457132425d1b84f3d80536b53e02942424 (the release tag plus its VERSION update) from production commit aae6db2894b5446e786f86fee787ee5209028e24. It retains GPT HTTP cancellation, checked writes and flush errors, response outcomes, observed-consumption accounting, timing/correlation, routing-capacity service alerts, and managed non-root container update protection.

OpenAI compact terminal events preserve upstream structured client error codes and capacity-code normalization while checking the actual downstream write and flush. A delivery failure reaches the caller as a delivery error, not an upstream failover signal. The previous duration-weighted OpenAI throughput and delivery filtering are unchanged.

The deployed Docker build, go.mod and CI use Go 1.27.4, including the net/http fix for GO-2026-6859. This retains the deployed toolchain alignment and matches current upstream.

This release adds or changes no migration files. Check that the full migration ledger and configuration identities match before and after cutover. The preceding maintained version uses the same schema. Keep existing data volumes during a code rollback; do not run downgrade SQL or restore snapshots automatically.

The runtime is an immutable non-root container. The existing /app/data mount remains in use. Version checks stay available; direct binary updates and official-binary rollback remain blocked with DEPLOYMENT_MANAGED_UPDATE so they cannot overwrite maintained fixes.

## Upgrade procedure

1. Fetch a specific upstream release tag and merge it into this maintained branch. Resolve conflicts without discarding the GPT delivery/accounting patch.
2. Update `backend/cmd/server/VERSION` and the Dockerfile commit identifier. Review dependency and migration changes.
3. Run managed-update tests, GPT delivery/cancellation/usage regressions, upstream security regressions, frontend locale/type/build checks, and the backend build. Real model calls require a separate bounded budget.
4. Create and read back Railway snapshots for PostgreSQL, application data, and Redis; record the current successful deployment as the rollback target.
5. Verify the merged tree matches the tested tree. Pin the Railway production service to that merged commit and verify the staged change contains only the source commit SHA before applying it.
6. Wait for the exact new deployment to succeed. Verify `/health`, the public runtime version, managed update guidance, and unchanged account/key/configuration identities. Do not publish based only on upload success.
7. If acceptance fails, roll back to the previous successful **patched** Railway deployment. Do not point the service at the unpatched official image. Retain additive usage-log columns; no destructive database rollback is needed.

This maintenance release uses `[skip ci]` after completing the bounded local acceptance suite, avoiding duplicate Actions runs. Existing workflow permissions are unchanged. Do not activate upstream release workflows or push upstream-shaped release tags just to ship this branch. Railway deployment does not require GitHub Actions; record acceptance for every maintenance release.

The Docker cache IDs are scoped to the existing Railway Sub2API service. Other installations must replace that prefix. No credentials or customer data belong in this repository.
