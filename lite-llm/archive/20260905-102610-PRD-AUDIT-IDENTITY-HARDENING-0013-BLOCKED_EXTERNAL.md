# LITE.LLM ID007 PRD Audit Identity Hardening

```text
PROJECT=LITE.LLM
FLOW_ID=ID007
OBJECTIVE_ID=PRD-AUDIT-IDENTITY-HARDENING-0013
STATE=BLOCKED
RESULT=BLOCKED_EXTERNAL
UPDATED_AT=2026-09-05T10:26:10+07:00
PRODUCTION_STATE=UNCHANGED
PRODUCTION_APPLICATION_MUTATION=NONE
PRD_DEPLOYMENT=NO
PRD_READY=NO
SECURITY_CHANGE=REMOVED_AOAUDIT_PASSWORDLESS_SUDO_RULE_ONLY
SUDOERS_SOURCE_REMOVED=YES
SUDOERS_ROOT_BACKUP_PRESERVED=YES
SUDOERS_VALIDATION=PASS
DO_NOT_MODIFY_CODEX_DEPLOY_PRIVILEGES=PASS
PRD_READONLY_REMOTE_IDENTITY=aoaudit
PRD_READONLY_AUTHENTICATION=PASS_LOCAL_PROTECTED_DPAPI
PRD_READONLY_SUDO=NONE
PRD_READONLY_DOCKER_ACCESS=NO
PRD_READONLY_PRIVILEGED_GROUPS=NONE
PRD_READONLY_ACTIVE_APPLICATION_ROOT_WRITE=NO
PRD_READONLY_WRITE_CAPABILITY=FAIL_PREEXISTING_WORLD_WRITABLE_LEGACY_OPT_TREE
PRD_READONLY_PREFLIGHT=FAIL_WRITE_CAPABILITY
COMPACT_HEALTH_PROBES=PASS
PROTECTED_MAIN_MERGE=PASS_PRESERVED
PROTECTED_MAIN_MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS_PRESERVED
QAS_POST_MERGE_SMOKE=PASS_PRESERVED
PRD_PROMOTION_PACKAGE=PASS_PRESERVED
EFFICIENCY_RESULT=PASS
KNOWN_BLOCKERS=PREEXISTING_WORLD_WRITABLE_LEGACY_OPT_SOURCE_TREE
CURRENT_WORK=NONE
NEXT_STEP=AUTHORIZE_A_SEPARATE_BOUNDED_PERMISSION_REMEDIATION_FOR_THE_LEGACY_WORLD_WRITABLE_OPT_TREE_THEN_RERUN_ONLY_THE_READONLY_PREFLIGHT
ETA=WAITING_FOR_SEPARATE_SECURITY_PERMISSION_SCOPE
SECRET_SCAN=PASS
```

## Security Change

Before-state inspection found the `aoaudit` passwordless sudo grant only in `/etc/sudoers.d/aoaudit-readonly`. ID007 approval authorized removal of that capability. The root-owned rule was preserved as a disabled root-only backup, removed from the active sudoers directory, and the complete sudoers configuration passed `visudo` validation. The `codex-deploy` account and its privileges were not changed.

After-state authentication used the existing DPAPI-protected `aoaudit` password credential. It authenticated as `aoaudit`; the account has only its dedicated group, no sudo command, no Docker socket access, and no write access to active application roots or other checked system roots. Local service and HTTP health probes remained responsive, including LiteLLM liveliness and readiness.

## Remaining Blocker

A literal write-capability scan found 3,851 pre-existing writable entries beneath `/opt`, including 1,874 entries in the legacy `9router-oauth-localhost-callback-src-20260818-221736` source tree. The active application roots are not writable, but ID007's PASS gate requires no write capability beyond normal home files. Changing those unrelated filesystem permissions was outside the sudo-only authorization, so automation stopped without broadening the mutation scope.

Full QAS and feature tests were not rerun because source and the protected-main tree are unchanged. No application deployment, service restart, database change, routing change, provider change, or production application mutation occurred.
