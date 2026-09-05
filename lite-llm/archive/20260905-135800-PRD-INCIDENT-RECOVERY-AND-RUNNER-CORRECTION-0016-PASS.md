# LITE.LLM ID010 Incident Recovery And Runner Correction

```text
PROJECT=LITE.LLM
FLOW_ID=ID010
OBJECTIVE_ID=PRD-INCIDENT-RECOVERY-AND-RUNNER-CORRECTION-0016
STATE=COMPLETED
RESULT=PASS
UPDATED_AT=2026-09-05T13:58:00+07:00
EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
PRODUCTION_STATE=HEALTHY
PRODUCTION_APPLICATION_DEPLOYMENT=NONE
PRODUCTION_APPLICATION_IMAGES=UNCHANGED
CREDENTIAL_ROTATION=PASS
OLD_CREDENTIALS_INVALIDATED=PASS
NEW_CREDENTIALS_OPERATIONAL=PASS
AT_REST_REENCRYPTION=PASS
DATABASE_ROLE_ROTATION=PASS
MIGRATION_008_PRESERVED=PASS
FINAL_SERVICE_RESTART_COUNTS=0
PRODUCTION_STABILITY_SECONDS=120
RUNNER_CORRECTION=PASS
RUNNER_UNIT_TESTS=7_PASS
QAS_RUNNER_REQUALIFICATION=PASS
QAS_APPLICATION_REDEPLOY=NO
PROMOTION_PACKAGE=PASS_PREPARED_NOT_DEPLOYED
SECRET_SCAN=PASS
KNOWN_BLOCKERS=NEW_EXPLICIT_PRD_PROMOTION_APPROVAL_REQUIRED
CURRENT_WORK=NONE
NEXT_STEP=AWAIT_NEW_EXPLICIT_PRD_PROMOTION_APPROVAL
ETA=WAITING_FOR_ADMINISTRATOR_DECISION
```

## Incident Recovery

The first ID010 rotation attempt successfully completed LiteLLM at-rest re-encryption, PostgreSQL role rotation, and all service readiness checks. Validation then failed and the mandatory rollback path restored the captured baseline before diagnosis continued.

Fresh post-rollback verification proved exact restoration of every affected dotenv and Compose authority, both 9Router databases, all old application and database credentials, and the additive migration `008_user_identity_model_policy`. Candidate credentials were inactive after rollback. All relevant containers and system services had zero restarts.

The failed gate was isolated to the PostgreSQL negative-password probe. It joined the database container network namespace and connected to loopback, where the local authentication rule trusted both correct and incorrect passwords. The application and credential rotation were not at fault. Validation was corrected to connect through the Docker bridge service address, which exercises SCRAM authentication, and every gate now emits a named PASS or FAIL result.

## Credential Rotation

A fresh protected checkpoint was captured before the retry. The retry followed the required sequence:

```text
PRECHECK=PASS
PRESTAGE_ROLLBACK=PASS
APPLY=PASS
READINESS=PASS
VALIDATE=PASS
ROLLBACK_USED=NO
```

The retained rotation changed all exposed LiteLLM, PostgreSQL role, Gateway Admin, Portal, 9Router, and correlation secret classes. LiteLLM at-rest material was re-encrypted before the salt transition. Both production and isolated shadow 9Router databases were updated consistently. Old LiteLLM, 9Router, Gateway Admin, LiteLLM UI, and four PostgreSQL role credentials were rejected; all replacements were accepted.

Replacement human login credentials were transferred without printing plaintext and stored in a current-user DPAPI-protected workstation artifact. A memory-only decrypt-and-structure check passed. The protected production checkpoints remain root-only, and no secret value or secret hash is included in this report.

## Production Validation

```text
LITELLM_READINESS=PASS
GATEWAY_ADMIN_HEALTH=PASS
USER_PORTAL_HEALTH=PASS
NINEROUTER_HEALTH=PASS
SHADOW_LITELLM_HEALTH=PASS
IMAGE_IDENTITIES_UNCHANGED=PASS
UNEXPECTED_RESTARTS=0
DOCKER_SERVICE=ACTIVE_RESTARTS_0
SSH_SERVICE=ACTIVE_RESTARTS_0
CLOUDFLARED_SERVICE=ACTIVE_RESTARTS_0
MIGRATION_008_PRESERVED=PASS
OBSERVATION_SECONDS=120
PRODUCTION_STABILITY=PASS
```

The credential-only incident response did not deploy qualified application images, change routing, change providers, or alter the protected source tree. The existing production application image baseline remained in place throughout the final retained state.

## Runner Correction

Added `scripts/ops/newline_safe_remote_runner.py` and `scripts/ops/test_newline_safe_remote_runner.py`. The runner normalizes UTF-8 shell payloads to LF, prevents CRLF-tail false failures, runs rollback only after a genuine payload failure, rejects known secret-bearing shell patterns, redacts sensitive output, handles Unicode safely on Windows, and always removes its remote temporary directory.

Seven focused tests passed, including exact reproduction of the former trailing-carriage-return failure, normalization, clean success without rollback, genuine failure with rollback, secret-bearing Compose-render rejection, output redaction, and Unicode output.

QAS requalification exercised both runner paths with CRLF source payloads. The deploy-path simulation passed without rollback. A forced exit `23` invoked rollback exactly once, removed its marker, and returned the original failure code. QAS application containers retained restart count zero and were not rebuilt, restarted, or redeployed.

## Promotion Package

A fresh preparation-only promotion package was created locally. It contains the exact qualified source archive, a verified complete Git bundle containing the qualified source and protected-main merge, and a manifest binding source, tree, migration, immutable image, and corrected-runner hashes.

```text
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
PROTECTED_MAIN_MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS
SOURCE_ARCHIVE_SHA256=dfd8b279f641871f5e8c5bfac7a66ea1b37432926385c483e69eea157920dcb7
SOURCE_BUNDLE_SHA256=8bc342d23440231738011c68103439c471ea6478bb85eb523fc3c8933e2af515
MIGRATION_008_SHA256=a4b8f7345a660adaa4f5ce6a9abbf742dd2ef075e4e8c434dc2b09e29891a51a
LITELLM_IMAGE=sha256:e7fed7bdfe858fca730fad9301f33e8b6797189034781115eca30ee87939c71f
GATEWAY_ADMIN_IMAGE=sha256:3b88d586020a7b456762d1dbceb6b4898b43921d2e1423a47a5d540c40c13d8b
CODEX_SETUP_DELIVERY_IMAGE=sha256:3b88d586020a7b456762d1dbceb6b4898b43921d2e1423a47a5d540c40c13d8b
USER_PORTAL_IMAGE=sha256:df52e449f3591713b641507e76676b071979d9eab5981642357e3fc29cc6c71b
PRODUCTION_DEPLOYMENT=NO
```

## Completion Boundary

ID010 is complete. The incident is remediated, production is stable, the runner is corrected and QAS-requalified, and a fresh promotion package is ready. No production promotion is authorized by this result. Stop and wait for a new explicit PRD promotion approval.
