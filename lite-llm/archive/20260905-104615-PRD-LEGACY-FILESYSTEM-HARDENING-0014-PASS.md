# LITE.LLM ID008 PRD Legacy Filesystem Hardening

```text
PROJECT=LITE.LLM
FLOW_ID=ID008
OBJECTIVE_ID=PRD-LEGACY-FILESYSTEM-HARDENING-0014
STATE=COMPLETED
RESULT=PASS
UPDATED_AT=2026-09-05T10:46:15+07:00
PRODUCTION_STATE=UNCHANGED
PRODUCTION_APPLICATION_MUTATION=NONE
PRD_DEPLOYMENT=NO
PRD_READY=YES
FILESYSTEM_PERMISSION_HARDENING=PASS
PERMISSION_CHANGE=REMOVE_OTHER_WRITE_BIT_ONLY
ENTRIES_CHANGED=3851
CONTENT_HASHES=UNCHANGED
OWNERSHIP=UNCHANGED
OWNER_GROUP_PERMISSION_BITS=UNCHANGED
ROLLBACK_MANIFEST=READY_ROOT_ONLY
ROLLBACK_MANIFEST_ENTRIES=3851
ROLLBACK_MANIFEST_MODE=600
PRD_READONLY_AUTHENTICATION=PASS_LOCAL_PROTECTED_DPAPI
PRD_READONLY_REMOTE_IDENTITY=aoaudit
PRD_READONLY_SUDO=NONE
PRD_READONLY_DOCKER_ACCESS=NO
PRD_READONLY_PRIVILEGED_GROUPS=NONE
PRD_READONLY_WRITE_CAPABILITY=NO
PRD_READONLY_PREFLIGHT=PASS
ACTIVE_PRODUCTION_SERVICES_HEALTHY=PASS
RESTART_REQUIRED=NO
SERVICE_IDENTITIES_UNCHANGED=PASS
SERVICE_RESTART_COUNTS_UNCHANGED=PASS
DO_NOT_MODIFY_CODEX_DEPLOY_PRIVILEGES=PASS
PROTECTED_MAIN_MERGE=PASS
PROTECTED_MAIN_MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS
QAS_POST_MERGE_SMOKE=PASS_PRESERVED
PRD_PROMOTION_PACKAGE=PASS_PRESERVED
EFFICIENCY_RESULT=PASS
KNOWN_BLOCKERS=NONE
CURRENT_WORK=NONE
NEXT_STEP=AWAIT_SEPARATE_ADMINISTRATOR_APPROVAL_BEFORE_ANY_PRD_DEPLOYMENT
ETA=COMPLETE
SECRET_SCAN=PASS
```

## Targeted Correction

ID007 identified 3,851 entries writable by `aoaudit` across five legacy `/opt` trees. Before mutation, every affected entry was writable solely through the world-write bit; none was owned by or grouped to `aoaudit`. The active container bind-mounted data directory contained no world-writable entries, and the affected legacy source trees were not required as live runtime mounts.

ID008 removed only the other-write bit from those exact regular files and directories. File contents, SHA-256 hashes, sizes, modification timestamps, owners, groups, and owner/group permission bits were verified unchanged. No unrelated path or permission was modified.

## Rollback Readiness

A root-owned mode `600` rollback manifest records the original mode, owner, group, size, modification time, and file hash for all 3,851 entries. Rollback consists only of restoring each recorded original mode from that manifest. The manifest is not accessible to `aoaudit`.

## Read-Only Preflight

Authentication through the existing DPAPI-protected credential passed as `aoaudit`. The account has only its dedicated group, no sudo capability, no Docker socket access, and zero writable regular files or directories under the checked system roots `/etc`, `/opt`, `/srv`, `/usr`, `/var/lib`, and `/var/log`. Normal writes inside its own home remain available.

LiteLLM liveliness and readiness returned HTTP `200`; Gateway Admin and User Portal returned their expected redirects; Codex Setup Delivery responded without connection failure. Docker, SSH, and cloudflared remained active with zero service restarts. All six required application/database containers retained the same container identities, images, start timestamps, and restart counts observed before the permission change; healthy containers remained healthy.

The protected-main merge tree still exactly equals the qualified source tree. Full QAS was not rerun because source was unchanged. No application deployment, service restart, database change, routing change, provider change, or production application mutation occurred.
