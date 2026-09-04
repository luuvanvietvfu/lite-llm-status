# LITE.LLM — User Identity Model Policy Pre-PRD Report

```text
PROJECT=LITE.LLM
FLOW_ID=ID002
OBJECTIVE_ID=USER-IDENTITY-MODEL-POLICY-0008
STATE=PASS
RESULT=PASS
UPDATED_AT=2026-09-04T22:46:22+07:00
EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
SOURCE_BRANCH=codex/public-status-channel
SOURCE_BASE_SHA=158ab014a7217e6813814e4fb200dfaf713e4ff1
SOURCE_STATE=UNCOMMITTED_WORKTREE
PRODUCTION_STATE=UNCHANGED
PRODUCTION_MUTATION=NONE
DEPLOYMENT_STATE=QAS_QUALIFIED_PRE_PRD_APPROVAL_REQUIRED
QAS_RESULT=PASS
ROLLBACK_USED=NO
ROLLBACK_RESULT=NOT_REQUIRED
BLOCKERS=NONE_FOR_DEVELOPMENT_AND_QAS_OBJECTIVE
CURRENT_WORK=NONE
NEXT_STEP=ADMINISTRATOR_APPROVAL_FOR_SOURCE_COMMIT_REVIEW_AND_SEPARATE_PRD_PROMOTION
ETA_OR_COMPLETION_STATE=COMPLETE
SECRET_SCAN=PASS
```

## Summary

Implemented and QAS-qualified per-user identity/model policy for Gateway Admin, User Portal, Windows Codex setup delivery, and LiteLLM request enforcement. Two distinct QAS users were validated with different allowlists, display names, model-switch permissions, default models, and reasoning defaults. Unauthorized model switches and unsupported reasoning levels were rejected before provider dispatch.

Production was not accessed or changed. The source remains uncommitted in the approved development worktree and requires separate review and production-promotion approval.

## Changes Made

- Added `billing.user_model_policies` with display name, allowed models, default model, model-switch permission, default reasoning effort, timestamps, and updater identity.
- Updated portal identity functions to return policy metadata through least-privilege database functions.
- Added shared model/reasoning validation and server-side LiteLLM enforcement in both pre-call hook paths.
- Added Gateway Admin policy controls and the User Portal `POST /preferences/default-model` route.
- Updated Windows Codex setup to use the policy allowlist plus `image-auto`, policy default model, and supported reasoning default.
- Preserved `image-auto` on existing dedicated Codex credentials when policy changes.
- Added QAS-only aliases `codex-sol`, `codex-terra`, and `antigravity-fast`.
- Added atomic mode-and-policy persistence and compensating cleanup for new-user persistence failure.
- Added callback database precedence: `BILLING_DATABASE_URL`, `BILLING_MIGRATION_DATABASE_URL`, then `DATABASE_URL`.

## Schema And Security

Migration: `billing/migrations/008_user_identity_model_policy.sql`.

- `allowed_models` must be a JSON array and contain `default_model`.
- Reasoning default is limited to empty, `low`, `medium`, `high`, or `xhigh`.
- `portal_app` receives read access and restricted preference updates; `gateway_admin_billing` receives policy CRUD.
- Public function access is revoked and portal functions are granted only to `portal_app`.
- Denied model/reasoning requests do not call the provider.
- Configured policy fails closed on database failure unless a valid cached policy exists.
- QAS Gateway Admin least-privilege connectivity was restored using its existing stored QAS credential; no credential was generated or exposed.
- New-user provisioning removes the newly created LiteLLM user/team/key if billing persistence fails.
- No production database, API, Cloudflare, routing, or identity mutation occurred.

## Local Validation

```text
PYTHON_COMPILEALL=PASS
LEGACY_LIFECYCLE_TESTS=PASS
NEW_USER_COMPENSATION_TEST=PASS
BILLING_CALLBACK_AND_MODEL_POLICY_TESTS=PASS count=32
CODEX_WINDOWS_SETUP_TESTS=PASS count=12
USER_PORTAL_UX_TESTS=PASS count=3
POSTGRES_INTEGRATION_TESTS=SKIPPED_NO_LOCAL_FIXTURE count=22
GIT_DIFF_CHECK=PASS
CHANGED_CONTENT_SECRET_SCAN=PASS
REPOSITORY_WIDE_SCAN=KNOWN_BASELINE_FAIL findings=23
```

The 23 repository-wide findings are pre-existing examples, QAS scripts, tests, and a tracked diagnostic log. Changed diff content and all new files passed the same scanner.

## QAS Qualification

Qualified images:

- LiteLLM: `sha256:2c504ada96f047e648358e82887ad07daace3a7abfe8de04a43ef71542b4f366`
- Gateway Admin and Codex setup delivery: `sha256:346c37cffa427d9d70b7185d79f422dec69b5de38c72d9c59d291d19caecc5e1`
- User Portal: `sha256:d8d4e322bcf01b15697e34bca98139d88f905b6a2e31e2a412175037cf1cdb96`

```text
QAS_MIGRATION_008=PASS
QAS_GATEWAY_ADMIN_BILLING_CONNECT=PASS
QAS_GATEWAY_ADMIN_ROLE_CREDENTIAL_REALIGNED=PASS_EXISTING_QAS_CREDENTIAL
QAS_ADMIN_ATOMIC_WRITE=PASS
QAS_ATOMIC_MODE_ROLLBACK=PASS
QAS_ATOMIC_POLICY_ROLLBACK=PASS
QAS_TWO_USER_POLICIES=PASS
QAS_DISPLAY_NAME=PASS
QAS_PORTAL_ALLOWLIST_VISIBILITY=PASS
QAS_MODEL_SWITCH_LOCK_UI=PASS
QAS_SERVER_MODEL_ALLOWLIST=PASS
QAS_SERVER_SWITCH_LOCK=PASS
QAS_REASONING_CAPABILITY_VISIBILITY=PASS
QAS_UNSUPPORTED_REASONING_REJECTED=PASS
QAS_DEFAULT_REASONING_INJECTION=PASS
QAS_PROVIDER_NOT_CALLED_ON_DENIAL=PASS
QAS_FALLBACK_WITHIN_PUBLIC_ALIAS=PASS_NO_CROSS_MODEL_FALLBACK_CONFIGURED
QAS_SYNTHETIC_CLEANUP=PASS
QAS_MUTABLE_SYNTHETIC_RESIDUE=0
QAS_IMMUTABLE_AUDIT_EVIDENCE=2
QAS_PRODUCTION_DB_CONNECTION=NO
QAS_PRODUCTION_API_WRITES=0
QAS_CLOUDFLARE_CONNECTION=NO
QAS_SEPARATE_NETWORK=YES
QAS_SEPARATE_VOLUMES=YES
QAS_ALL_REQUIRED_SERVICES_HEALTHY=PASS
QAS_ALL_REQUIRED_SERVICE_RESTART_COUNTS=0
```

The two immutable audit rows document the QAS atomic write check and remain under the existing financial-history immutability guard. All mutable test policy, mode, LiteLLM user, and key rows were removed.

## Restart And Downtime

- QAS LiteLLM was recreated while resolving callback database URL compatibility; final restart count is zero.
- QAS Gateway Admin was recreated for compensated provisioning; final restart count is zero.
- QAS Codex setup delivery was recreated to align it with the qualified Gateway Admin image; final restart count is zero.
- QAS User Portal, 9Router, PostgreSQL, and mock provider remained healthy.
- No production restart or downtime occurred.

## Rollback Plan

Captured QAS pre-change images:

- LiteLLM: `sha256:5590d27428c565a6d33a450930b4e12b7c773faaa8686aca5802eb4ad4eab9c0`
- Gateway Admin and setup delivery: `sha256:56376c82cb2ac9cfe656132ff91918b56d63f9005313a7e908396081e184e794`
- User Portal: `sha256:1e3621e96eddbc10d28fe8d15ce129b8ad2afdd289aded8edfd032c210c1f445`

QAS billing and Gateway Admin backups were captured before migration. A complete rollback restores the pre-change billing dump and Gateway Admin SQLite backup after stopping only affected QAS services, restores the prior images, and reruns baseline checks. Production rollback must use fresh pre-promotion backups and the exact production image digests captured during the approved promotion window.

## Exact PRD Promotion Steps

No step below is authorized by this report. Separate Administrator approval is required.

1. Review the complete worktree diff, commit it to the approved branch, and complete protected-main review.
2. Record the immutable merged SHA and build LiteLLM, Gateway Admin, User Portal, and setup-delivery images from that SHA.
3. Capture verified production billing and Gateway Admin backups, current image digests, health, restart counts, and routing baseline.
4. Confirm migration 008 is not partially applied, then apply it through the approved billing migration service.
5. Recreate only LiteLLM, Gateway Admin, Codex setup delivery, and User Portal on approved immutable images.
6. Verify health and restart counts after each service; rollback on unexpected 5xx, restart loops, database errors, provider leakage, or policy mismatch.
7. Validate two controlled production identities with distinct policies, portal visibility, locked switching, default reasoning, unsupported-reasoning rejection, and provider-not-called denial behavior.
8. Remove mutable production test identities, policies, and keys while retaining required immutable audit history.
9. Publish the production report only after backups, migration, health, cleanup, security, and rollback evidence pass.

## Final State

Development and QAS objective `USER-IDENTITY-MODEL-POLICY-0008` is complete. Production remains unchanged. The next action is Administrator approval for source review/commit and a separately authorized production promotion.
