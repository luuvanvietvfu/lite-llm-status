# LITE.LLM ID020 Setup Effective Model Validation

    PROJECT=LITE.LLM
    FLOW_ID=ID020
    ATTEMPT=1
    OBJECTIVE_ID=SETUP-EFFECTIVE-MODEL-VALIDATION-0028
    STATE=COMPLETED
    RESULT=PASS
    UPDATED_AT=2026-09-06T10:26:10+07:00
    EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
    SOURCE_COMMIT=7a361a9fa8116ccbe7acf4e8eae497dd87de5fcb
    SOURCE_TREE=67f0eb26bf45c9fa9d895174c98320479944bf95
    RELEASE_ARCHIVE_SHA256=197b4938a3876d2e14311155738541fb3fda99cb497d85ae4ef8c6723018d624
    ADMIN_SETUP_IMAGE=ai-gateway/user-admin:id020-setup-effective-model-20260906T100036
    ADMIN_SETUP_IMAGE_ID=sha256:1f7823c6b23c23098a220288520d7739d23c12608c0d7a7031d21792f700662e
    TARGETED_TESTS=17_PASS
    SETUP_SHORT_LINK_EXECUTION=PASS
    SETUP_MODEL_VALIDATION_POLICY_ALIGNED=PASS
    IMAGE_AUTO_REQUIREMENT=AUTHORITATIVE_AND_CONDITIONAL
    USER_POLICY_KEY_ENTITLEMENT_SYNC=PASS
    CODING_AUTO_REAL_REQUEST=PASS
    GPT_5_5_REAL_REQUEST=PASS
    OUT_OF_POLICY_DENY=PASS
    VIET_CONTROL_ACCOUNT_UNCHANGED=PASS
    VIET_CODEX_REAL_REQUEST=PASS
    SETUP_TOKEN_TTL_60M=PASS
    SETUP_TOKEN_SINGLE_USE=PASS
    LEGACY_RAW_KEY_SETUP_DISABLED=PASS
    SYNTHETIC_RESIDUE=0
    AUDIT_HISTORY_PRESERVED=PASS
    DEPLOYED_SOURCE_MATCH=PASS
    PUBLIC_HTTPS_ADMIN_ACCESS=PASS
    ADMINISTRATOR_CAPABILITIES=PASS
    USER_PORTAL_LOGIN=PASS
    LITELLM_LIVELINESS=PASS
    APPLICATION_DEPLOYMENT=LITELLM_USER_ADMIN_AND_CODEX_SETUP_DELIVERY_ONLY
    SCHEMA_OR_MIGRATION_CHANGE=NONE
    FULL_STACK_RESTART=NO
    UNRELATED_SERVICE_RESTARTS=0
    FINAL_SERVICE_RESTART_COUNTS=0
    ROLLBACK_USED=NO
    PRODUCTION_STATE=HEALTHY_USER_VERIFIED
    NEW_SECRET_OUTPUT=NONE

## Root Cause and Fix

- The Windows bootstrap previously required `image-auto` unconditionally even when the user's authoritative effective entitlement excluded that model.
- The setup payload now carries the exact assigned model set from the existing per-user policy and dedicated key. The bootstrap validates every assigned model and does not require unassigned models.
- `lite-imagegen` is installed and reported only when `image-auto` is assigned. A user without that entitlement receives explicit `NOT_ASSIGNED` results instead of a false setup failure.
- Fail-closed behavior remains: setup fails if any model the user is actually assigned is unavailable.

## Targeted Validation

- Seventeen focused Windows setup tests passed. No full QAS or repository-wide test run was performed.
- One disposable user was created through the real Admin flow with the exact effective models `coding-auto,gpt-5.5`. Its stored setup policy, dedicated key, and Admin-selected entitlement matched exactly.
- A fresh 60-minute, single-use public setup link completed on a clean Windows profile. The resulting Codex configuration selected `coding-auto`; `lite-imagegen` was absent because `image-auto` was not assigned.
- Real public HTTPS requests for `coding-auto` and assigned `gpt-5.5` returned HTTP `200`. An `image-auto` request for the unassigned disposable user was denied server-side.
- Reuse of the redeemed setup link returned HTTP `404`. The legacy raw-key setup path remained disabled.
- The protected `viet/viet-codex` binding retained user ID `viet`, key alias `viet-codex`, and its existing model scope. A fresh real public HTTPS `coding-auto` request returned HTTP `200`.

## Promotion and Production State

- Commit `7a361a9fa8116ccbe7acf4e8eae497dd87de5fcb` was pushed before promotion. The release archive contains 46 entries, no cache or bytecode entries, and SHA-256 `197b4938a3876d2e14311155738541fb3fda99cb497d85ae4ef8c6723018d624`.
- Only `litellm-user-admin` and `codex-setup-delivery` were recreated with the candidate image. No full-stack or unrelated service restart occurred.
- The four changed runtime files matched the release archive byte-for-byte in `litellm-user-admin`; the bootstrap asset also matched byte-for-byte in `codex-setup-delivery`.
- LiteLLM liveliness returned HTTP `200`; the canonical User Portal login returned HTTP `200`; the Portal root redirected with HTTP `303`; the Admin Access boundary returned HTTP `302`; and an invalid setup token returned HTTP `404`.
- `litellm-user-admin`, `codex-setup-delivery`, `litellm-user-portal`, `litellm`, `9router`, and `litellm-db` are running. Every restart count is `0`; all services with configured Docker healthchecks are healthy.

## Cleanup

- The disposable LiteLLM user, key, team, membership, Admin user/key rows, Codex credential, setup token, portal session, model preference, model policy, and user mode all have count `0`.
- Two immutable audit events for the disposable flow remain preserved. No persistent user, key, model policy, billing data, provider configuration, schema, or migration was changed during cleanup.

## Final State

- Production setup validation is aligned with authoritative per-user effective models. Optional `image-auto` capability is no longer treated as a global prerequisite.
- Production is healthy and user-verified with no synthetic residue, no unrelated restart, no rollback, and no secret output.
