# LITE.LLM PRD Gate And Token Efficiency Checkpoint

```text
PROJECT=LITE.LLM
FLOW_ID=ID005
OBJECTIVE_ID=PRD-GATE-TOKEN-EFFICIENCY-0011
STATE=BLOCKED
RESULT=BLOCKED_EXTERNAL
UPDATED_AT=2026-09-05T09:24:00+07:00
EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
PRODUCTION_STATE=UNCHANGED
PRODUCTION_MUTATION=NONE
PRD_DEPLOYMENT=NO
PRD_READY=NO
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
PROTECTED_MAIN_MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
PROTECTED_MAIN_MERGE=PASS
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS
PRD_READONLY_PREFLIGHT=BLOCKED_EXTERNAL
QAS_POST_MERGE_SMOKE=PASS
PRD_PROMOTION_PACKAGE=PASS
TOKEN_EFFICIENCY_POLICY=PASS
KNOWN_BLOCKERS=INTENDED_PRD_READONLY_IDENTITY_NOT_AUTHENTICATING;AVAILABLE_FALLBACK_IDENTITY_IS_PRIVILEGED_AND_REJECTED
SECRET_SCAN=PASS
```

## Protected Main

Internal merge request `!5` was re-read immediately before execution through the established DPAPI-protected Maintainer API path. The merge was allowed only after all of these checks passed:

```text
MR_STATE=opened
MR_SOURCE_BRANCH=release/final-prd-readiness-correction-0010
MR_TARGET_BRANCH=main
MR_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
MR_HAS_CONFLICTS=NO
MR_DETAILED_STATUS=mergeable
MR_SQUASH=NO
PROJECT_MERGE_METHOD=merge
PROTECTED_MAIN_MERGE_LEVEL=MAINTAINER
CURRENT_IDENTITY_ACCESS=OWNER
```

The merge completed without squash at `557b0647beae5fd1bf4a7c58e933a8e1440a8320`. Its two parents are protected-main base `9a57d198132700ca499fd93669654885efa9630e` and qualified source `354eda7e89f28cdf7601ea7a342ca48437ab8696`. The merge tree is `d42e856126a4a674729bac8a9d4e349b5a1cd40b`, exactly equal to the qualified source tree.

## PRD Read-Only Preflight

The intended `aoaudit` production read-only account did not accept the currently available workstation key. The documented dedicated `prd-ao-readonly-ed25519` private key remains absent from the approved workstation credential location.

The older `codex-deploy` identity was tested only with non-mutating capability checks and was explicitly rejected as a substitute because it has passwordless unrestricted sudo and Docker-group access. This fails the ID005 requirement that the preflight identity have no sudo or write capability.

```text
INTENDED_ACCOUNT=aoaudit
INTENDED_ACCOUNT_AUTHENTICATION=FAIL_PUBLICKEY
DEDICATED_READONLY_KEY=ABSENT
FALLBACK_ACCOUNT=codex-deploy
FALLBACK_ACCOUNT_SUDO=NOPASSWD_ALL
FALLBACK_ACCOUNT_DOCKER_ACCESS=YES
FALLBACK_ACCOUNT_ACCEPTED_FOR_PREFLIGHT=NO
PRD_RUNTIME_COMMANDS_EXECUTED=READ_ONLY_CAPABILITY_CHECKS_ONLY
PRD_RUNTIME_WRITES=NONE
```

## QAS Post-Merge Smoke

The source tree did not change, so the full successful ID004 suite was not rerun. The bounded post-merge smoke used container state, health checks, restart counts, immutable image IDs, and source revision labels.

```text
LITELLM=RUNNING_HEALTHY_RESTARTS_0_REVISION_354eda7e89f28cdf7601ea7a342ca48437ab8696
GATEWAY_ADMIN=RUNNING_HEALTHY_RESTARTS_0_REVISION_354eda7e89f28cdf7601ea7a342ca48437ab8696
CODEX_SETUP_DELIVERY=RUNNING_HEALTHY_RESTARTS_0_REVISION_354eda7e89f28cdf7601ea7a342ca48437ab8696
USER_PORTAL=RUNNING_HEALTHY_RESTARTS_0_REVISION_354eda7e89f28cdf7601ea7a342ca48437ab8696
NINEROUTER=RUNNING_HEALTHY_RESTARTS_0_UNCHANGED
POSTGRESQL=RUNNING_HEALTHY_RESTARTS_0_UNCHANGED
QAS_POST_MERGE_SMOKE=PASS
```

## Final PRD Promotion Package

This package is preparation only. It does not authorize deployment.

### Immutable source and candidate artifacts

```text
SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
SOURCE_ARCHIVE_SHA256=dfd8b279f641871f5e8c5bfac7a66ea1b37432926385c483e69eea157920dcb7
SOURCE_BUNDLE_SHA256=6b67c395585db66c5b28aa088d8521ea7a47af178da3f8e9ea0a93ad7cf500a4
MIGRATION_008_SHA256=a4b8f7345a660adaa4f5ce6a9abbf742dd2ef075e4e8c434dc2b09e29891a51a
LITELLM_IMAGE=sha256:e7fed7bdfe858fca730fad9301f33e8b6797189034781115eca30ee87939c71f
GATEWAY_ADMIN_IMAGE=sha256:3b88d586020a7b456762d1dbceb6b4898b43921d2e1423a47a5d540c40c13d8b
CODEX_SETUP_DELIVERY_IMAGE=sha256:3b88d586020a7b456762d1dbceb6b4898b43921d2e1423a47a5d540c40c13d8b
USER_PORTAL_IMAGE=sha256:df52e449f3591713b641507e76676b071979d9eab5981642357e3fc29cc6c71b
NINEROUTER_CHANGE=NONE
POSTGRESQL_IMAGE_CHANGE=NONE
ROUTING_CHANGE=NONE
PROVIDER_CHANGE=NONE
```

### Ordered deployment steps for a separately approved flow

1. Reconfirm protected `main` is `557b0647beae5fd1bf4a7c58e933a8e1440a8320` and its tree equals `d42e856126a4a674729bac8a9d4e349b5a1cd40b`.
2. Authenticate with the dedicated strict read-only PRD identity and capture service state, image IDs, restart counts, active compose files, database migration state, free space, and current health. Stop if that identity has sudo, Docker write, or filesystem write capability.
3. Verify all four candidate images are present byte-for-byte by immutable image ID and that their revision label is `354eda7e89f28cdf7601ea7a342ca48437ab8696`. Do not rebuild in PRD.
4. Capture fresh billing PostgreSQL and Gateway Admin SQLite backups; validate them with `pg_restore --list` and SQLite `PRAGMA integrity_check` before mutation.
5. Stage candidate and rollback compose overlays in one timestamped release directory. Render every compose combination with `docker compose config`; verify only LiteLLM, Gateway Admin, Codex Setup Delivery, and User Portal image fields differ.
6. Apply `billing/migrations/008_user_identity_model_policy.sql` once inside its transaction and verify `billing.schema_migrations` contains `008_user_identity_model_policy`.
7. Recreate only LiteLLM, then wait for healthy status and HTTP liveliness/readiness PASS.
8. Recreate Gateway Admin and Codex Setup Delivery together, then wait for both health checks and restart count `0`.
9. Recreate User Portal, then wait for health check and restart count `0`.
10. Run controlled two-identity validation for display name, allowlist visibility, model-switch lock, reasoning visibility, unsupported reasoning rejection, default reasoning injection, provider-not-called denial, and same-public-alias fallback.
11. Remove synthetic user/key/policy rows, verify residue `0`, and observe all required services for at least two minutes with zero failures and zero restarts.

### Rollback checkpoint and commands

The QAS rehearsal proved these rollback image IDs:

```text
ROLLBACK_LITELLM_IMAGE=sha256:2c504ada96f047e648358e82887ad07daace3a7abfe8de04a43ef71542b4f366
ROLLBACK_GATEWAY_ADMIN_IMAGE=sha256:346c37cffa427d9d70b7185d79f422dec69b5de38c72d9c59d291d19caecc5e1
ROLLBACK_CODEX_SETUP_DELIVERY_IMAGE=sha256:346c37cffa427d9d70b7185d79f422dec69b5de38c72d9c59d291d19caecc5e1
ROLLBACK_USER_PORTAL_IMAGE=sha256:d8d4e322bcf01b15697e34bca98139d88f905b6a2e31e2a412175037cf1cdb96
ROLLBACK_POSTGRESQL_IMAGE=UNCHANGED
ROLLBACK_NINEROUTER_IMAGE=UNCHANGED
```

From the future timestamped release directory, rollback service images with the pre-rendered rollback overlays and the captured live base compose files:

```sh
docker compose -f live-litellm.yml -f rollback-litellm.yml up -d --no-deps --force-recreate litellm
docker compose -f live-gateway-admin.yml -f rollback-gateway-admin.yml up -d --no-deps --force-recreate litellm-user-admin codex-setup-delivery
docker compose -f live-user-portal.yml -f rollback-user-portal.yml up -d --no-deps --force-recreate user-portal
```

Migration `008` is additive and was preserved during the successful QAS rollback rehearsal. Database restore is reserved for a demonstrated integrity or semantic failure and must use the fresh validated backups captured in step 4.

### Go/no-go and expected impact

```text
GO_REQUIRES=SEPARATE_PRD_APPROVAL;DEDICATED_READONLY_PREFLIGHT_PASS;ALL_DIGESTS_PRESENT;BACKUPS_VALID;COMPOSE_RENDER_PASS;ZERO_UNEXPLAINED_DRIFT
NO_GO_ON=READONLY_IDENTITY_FAILURE;TREE_MISMATCH;DIGEST_MISMATCH;BACKUP_FAILURE;HEALTH_FAILURE;RESTART_COUNT_INCREASE;PROVIDER_LEAKAGE;POLICY_MISMATCH;SYNTHETIC_RESIDUE
EXPECTED_SERVICE_RECREATES=4
EXPECTED_DATABASE_RESTART=NO
EXPECTED_NINEROUTER_RESTART=NO
EXPECTED_DOWNTIME=BRIEF_PER_SERVICE_CONNECTION_INTERRUPTION_DURING_FOUR_BOUNDED_RECREATES
ROLLBACK_TRIGGER=ANY_NO_GO_CONDITION
```

## Token Efficiency

```text
TOKEN_EFFICIENCY_RESULT=PASS
EFFICIENCY_TELEMETRY=PARTIAL
FRESH_CONTEXT_USED=YES
HISTORICAL_REPORTS_BULK_LOADED=NO
FULL_REPOSITORY_LOADED=NO
UNCHANGED_FULL_QAS_RERUN=NO
DETERMINISTIC_CHECKS_RUN_IN_TOOLS=YES
ESTIMATED_MODEL_REQUEST_COUNT=35_TO_45
LARGEST_INPUT_REQUEST=NOT_OBSERVABLE_IN_RUNTIME
INPUT_REQUEST_OVER_100K=NOT_OBSERVED_RUNTIME_METRICS_UNAVAILABLE
```

Targeted transcript extraction was used only to recover the established DPAPI GitLab Maintainer workflow and prior QAS artifact commands. Several searches were stopped or narrowed when they produced excessive output. The full QAS suite was not repeated.

## Result

```text
RESULT=BLOCKED_EXTERNAL
PRD_READY=NO
BLOCKER=DEDICATED_PRD_READONLY_IDENTITY_CANNOT_AUTHENTICATE_AND_REQUIRED_KEY_IS_NOT_AVAILABLE
CURRENT_WORK=NONE
NEXT_STEP=RESTORE_OR_PROVISION_THE_ALREADY_APPROVED_DEDICATED_AOAUDIT_CREDENTIAL_WITHOUT_CHANGING_ITS_READONLY_SERVER_BOUNDARY_THEN_RERUN_ONLY_PRD_READONLY_PREFLIGHT
ETA=WAITING_FOR_CREDENTIAL_RESTORATION
ADMIN_ACTION_REQUIRED=RESTORE_THE_EXISTING_DEDICATED_PRD_READONLY_PRIVATE_KEY_FROM_APPROVED_BACKUP_OR_EXPLICITLY_AUTHORIZE_REENROLLMENT_IF_PERMANENTLY_LOST
```

Protected-main merge is complete. Production runtime was not deployed or mutated.
