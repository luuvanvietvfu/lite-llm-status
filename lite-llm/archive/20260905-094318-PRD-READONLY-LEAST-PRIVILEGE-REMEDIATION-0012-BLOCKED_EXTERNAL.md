# LITE.LLM ID006 Read-Only Least-Privilege Remediation

```text
PROJECT=LITE.LLM
FLOW_ID=ID006
OBJECTIVE_ID=PRD-READONLY-LEAST-PRIVILEGE-REMEDIATION-0012
STATE=BLOCKED
RESULT=BLOCKED_EXTERNAL
UPDATED_AT=2026-09-05T09:43:18+07:00
PRODUCTION_STATE=UNCHANGED
PRODUCTION_APPLICATION_MUTATION=NONE
PRD_DEPLOYMENT=NO
PRD_READY=NO
QUALIFIED_SOURCE_SHA=354eda7e89f28cdf7601ea7a342ca48437ab8696
QUALIFIED_SOURCE_TREE=d42e856126a4a674729bac8a9d4e349b5a1cd40b
PROTECTED_MAIN_MERGE_SHA=557b0647beae5fd1bf4a7c58e933a8e1440a8320
PROTECTED_MAIN_MERGE=PASS
MERGED_TREE_EQUALS_QUALIFIED_SOURCE=PASS
QAS_POST_MERGE_SMOKE=PASS_PRESERVED
PRD_PROMOTION_PACKAGE=PASS_PRESERVED
AUDIT_ACCOUNT_EXISTS=YES
AUDIT_ACCOUNT_DOCKER_ACCESS=NO
AUDIT_ACCOUNT_SYSTEM_PATH_WRITE=NO
AUDIT_ACCOUNT_SUDO=FAIL_CONSTRAINED_NOPASSWD_ALLOWLIST_PRESENT
APPROVED_LOCAL_KEY_AUTHENTICATES_AS_AUDIT_ACCOUNT=NO
AUDIT_AUTHORIZED_KEY_EQUALS_APPROVED_LOCAL_KEY=NO
PRD_READONLY_PREFLIGHT=NOT_RUN_STOPPED_AT_LEAST_PRIVILEGE_GATE
DO_NOT_MODIFY_CODEX_DEPLOY_PRIVILEGES=PASS
OBSOLETE_LEGACY_KEY_RECOVERY_REQUIRED=NO
SECRET_SCAN=PASS
BLOCKERS=AUDIT_IDENTITY_HAS_PASSWORDLESS_SUDO_ALLOWLIST;APPROVED_LOCAL_KEY_IS_NOT_ENROLLED_FOR_AUDIT_IDENTITY
CURRENT_WORK=NONE
NEXT_STEP=REMOVE_AUDIT_SUDO_CAPABILITY_OR_REPLACE_IT_WITH_A_NON_SUDO_READONLY_MECHANISM_AND_ENROLL_THE_APPROVED_LOCAL_PUBLIC_KEY_WITH_EXISTING_RESTRICTIONS
ETA=WAITING_FOR_SERVER_SIDE_SECURITY_REMEDIATION
```

## Inspection

The dedicated `aoaudit` account exists in its own group, is not in the Docker group, cannot access the Docker socket, and cannot directly write relevant system paths. It nevertheless has a passwordless sudo allowlist for the root-owned read-only inspection wrapper. ID006 requires no sudo capability, so this is an explicit least-privilege failure.

The account has one authorized key with `restrict` and a forced read-only dispatcher. Its fingerprint is `SHA256:bIoY3MEmRtMJ/KhhTd2G/5/83HGcTwRbGjd0Uo4OWLM`, which differs from the approved local key fingerprint `SHA256:U3jWvy5GShEpOC/MWQOp/Y+F+NceEd3GcsTLKNUr/Xs`. Direct `aoaudit` authentication with the approved key was denied.

## Stop Condition

ID006 requires a stop before permission changes when the audit account is overprivileged. Automation did not modify sudoers, account membership, authorized keys, SSH configuration, `codex-deploy`, production configuration, services, containers, databases, routing, or application state. The health and integrity preflight was not run after the gate failure.

The protected-main merge, exact tree equality, QAS post-merge smoke, and promotion/rollback package remain unchanged and passing. Full QAS and feature tests were not rerun because source did not change.

`PRD_READY=NO`. Server-side security remediation is required before rerunning only the bounded read-only preflight.
