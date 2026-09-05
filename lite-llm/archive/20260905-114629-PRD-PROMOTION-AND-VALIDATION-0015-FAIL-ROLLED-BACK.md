# LITE.LLM ID009 PRD Promotion And Validation

```text
PROJECT=LITE.LLM
FLOW_ID=ID009
OBJECTIVE_ID=PRD-PROMOTION-AND-VALIDATION-0015
STATE=COMPLETED
RESULT=FAIL_ROLLED_BACK
UPDATED_AT=2026-09-05T11:46:29+07:00
EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
STARTING_GIT_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
ENDING_GIT_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS
PRD_DEPLOYMENT=ROLLED_BACK_NOT_RETAINED
GIT_COMMIT_DEPLOYED=NONE_FINAL
PRODUCTION_REPO_DRIFT=NO
ROLLBACK_USED=YES
ROLLBACK_RESULT=PASS
POST_DEPLOY_VALIDATION=NOT_RUN_ROLLBACK_PRECEDED_VALIDATION
SECURITY_INCIDENT=SECRET_BEARING_COMPOSE_VALUES_PRINTED_TO_TASK_OUTPUT
KNOWN_BLOCKERS=PRODUCTION_CREDENTIAL_ROTATION_REQUIRED;RUNNER_NEWLINE_FIX_AND_REQUALIFICATION_REQUIRED;NEW_PRD_APPROVAL_REQUIRED
CURRENT_WORK=NONE
ETA=WAITING_FOR_ADMINISTRATOR_DECISION
```

## Live State Before

- The ID008 least-privilege audit gate remained fresh and passing: `aoaudit` had no sudo, Docker access, privileged group, or writable checked system entry.
- Docker, SSH, and cloudflared were active with zero service restarts. LiteLLM liveliness/readiness returned HTTP `200`; Admin and Portal returned their expected redirects; setup delivery responded without connection failure.
- Fresh live rollback images were captured as LiteLLM `sha256:5590d27428c565a6d33a450930b4e12b7c773faaa8686aca5802eb4ad4eab9c0`, Admin/setup `sha256:56376c82cb2ac9cfe656132ff91918b56d63f9005313a7e908396081e184e794`, and Portal `sha256:8af37353584bee33d52c06075015358d7bebf0dedeae47085b05157b89c1801e`.
- 9Router and PostgreSQL identities were captured and remained outside the deployment scope.

## Prestage And Backups

- The three QAS-qualified images were transferred byte-for-byte without rebuilding. Their immutable IDs matched the approved package and every revision label equaled `354eda7e89f28cdf7601ea7a342ca48437ab8696`.
- Migration `008_user_identity_model_policy.sql` matched approved SHA-256 `a4b8f7345a660adaa4f5ce6a9abbf742dd2ef075e4e8c434dc2b09e29891a51a`.
- Candidate and rollback compose overlays rendered successfully. Structural comparison proved that only the four approved service image fields changed.
- A fresh PostgreSQL custom-format backup passed `pg_restore --list`. A fresh Gateway Admin SQLite online backup passed `PRAGMA integrity_check`.
- Root-only evidence and rollback material is stored at `/root/lite-llm-id009-20260905T042630Z` with directory mode `700`.

## Promotion Attempt

```text
MIGRATION_008=PASS
LITELLM_RECREATE_AND_READINESS=PASS
GATEWAY_ADMIN_RECREATE_AND_HEALTH=PASS
CODEX_SETUP_DELIVERY_RECREATE_AND_HEALTH=PASS
USER_PORTAL_RECREATE_AND_HEALTH=PASS
NINEROUTER_IDENTITY_UNCHANGED=PASS
POSTGRESQL_IDENTITY_UNCHANGED=PASS
```

The additive migration committed successfully. The four approved services were then recreated in the required order and reached their health gates with the qualified images. 9Router and PostgreSQL were not recreated.

After all promotion health gates printed PASS, a trailing carriage-return byte in the streamed local runner produced `bash: $'\r': command not found`. The rollback-first trap correctly treated the nonzero runner exit as a failure and immediately restored the fresh live image checkpoint. Per the no-retry policy, no second promotion attempt was made and the two-identity production validation was not started.

## Security Incident

During sealed-checkpoint review, normalized Compose excerpts containing secret-bearing environment values were printed to task output. No secret value is repeated in this report. The affected production runtime credentials must be treated as exposed, including the printed LiteLLM, database, 9Router, correlation, Gateway Admin, and Portal secret classes.

The root-only evidence directory remains protected. PRD candidate image tags and all transient transfer archives were removed after rollback; the qualified QAS images remain available on the isolated QAS host. A separate administrator-approved credential-rotation incident response is required before another promotion.

## Rollback And Final State

```text
ROLLBACK_USED=YES
ROLLBACK_RESULT=PASS
FINAL_LITELLM_IMAGE=sha256:5590d27428c565a6d33a450930b4e12b7c773faaa8686aca5802eb4ad4eab9c0
FINAL_GATEWAY_ADMIN_IMAGE=sha256:56376c82cb2ac9cfe656132ff91918b56d63f9005313a7e908396081e184e794
FINAL_CODEX_SETUP_DELIVERY_IMAGE=sha256:56376c82cb2ac9cfe656132ff91918b56d63f9005313a7e908396081e184e794
FINAL_USER_PORTAL_IMAGE=sha256:8af37353584bee33d52c06075015358d7bebf0dedeae47085b05157b89c1801e
FINAL_NINEROUTER_IMAGE=sha256:c37d860e563923017ff10ed8ec99b1476fd2ad6c519783f41085bd201046189c
FINAL_POSTGRESQL_IMAGE=sha256:57c72fd2a128e416c7fcc499958864df5301e940bca0a56f58fddf30ffc07777
FINAL_SERVICE_RESTART_COUNTS=0
FINAL_LITELLM_LIVELINESS=200
FINAL_LITELLM_READINESS=200
FINAL_ADMIN_HTTP=303
FINAL_SETUP_HTTP=404_EXPECTED_ROOT_ROUTE
FINAL_PORTAL_HTTP=303
ROLLBACK_STABILITY_SECONDS=120
ROLLBACK_STABILITY_CYCLES=12
ROLLBACK_STABILITY_FAILURES=0
MIGRATION_008_PRESERVED=PASS
SYNTHETIC_POLICY_RESIDUE=0
```

The application images are restored to the pre-attempt baseline and remained healthy for the required two-minute observation window. Migration `008` is additive and remains applied, matching the approved rollback contract. No synthetic user, key, or policy validation data was created.

## Next Step

Obtain separate administrator approval for a bounded production credential-rotation incident response covering every secret-bearing runtime value exposed in task output. After rotation, correct and requalify the promotion runner's newline handling offline/QAS, then request a new explicit PRD promotion approval. Do not retry ID009.
