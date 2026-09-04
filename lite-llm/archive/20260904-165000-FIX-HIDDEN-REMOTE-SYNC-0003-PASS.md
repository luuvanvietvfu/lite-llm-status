# LITE.LLM Hidden Remote Sync Repair

```text
PROJECT=LITE.LLM
OBJECTIVE=FIX-HIDDEN-REMOTE-SYNC-0003
RESULT=PASS
START_TIME=2026-09-04T16:35:48+07:00
END_TIME=2026-09-04T16:50:00+07:00
ELAPSED_TIME=00:14:12
EXECUTION_HOST=ADMIN_WINDOWS_WORKSTATION
WORKSPACE_ROOT=C:\Users\admin\Documents\Lite LLM
GIT_REPO_ROOT=C:\Users\admin\Documents\Lite LLM\worktrees\short-codex-current-main
STARTING_GIT_SHA=4c85190894c36f86193c7f01929ddd8671f6fe46
ENDING_GIT_SHA=4c85190894c36f86193c7f01929ddd8671f6fe46
CURRENT_BRANCH=codex/public-status-channel
TARGET_ENVIRONMENT=ADMIN_WINDOWS_WORKSTATION_LOCAL_AUTOMATION
APPROVAL_SCOPE=REMOTE_OBJECTIVE_SYNC_OPERATOR_EXPERIENCE_FIX_NO_PRODUCTION_MUTATION
POPUP_ROOT_CAUSE=CONFIRMED_LITE.LLM_REMOTE_OBJECTIVE_SYNC_TASK_INTERACTIVE_POWERSHELL_ACTION
SYNC_TASK_NAME=LITE.LLM Remote Objective Sync
SYNC_TASK_BACKGROUND_MODE=PASS_WSCRIPT_HIDDEN_LAUNCHER
SYNC_INTERVAL=PT1M
SYNC_LAST_RUN_RESULT=0
VISIBLE_CONSOLE_FLASH=NO
REMOTE_OBJECTIVE_SYNC=PASS
REMOTE_OBJECTIVE_EXACT_MATCH=PASS
REMOTE_OBJECTIVE_CHANNEL_VERSION_3=PASS
DUPLICATE_OBJECTIVE_GUARD=PASS
REDUNDANT_TASKS_FOUND=NO
REDUNDANT_TASKS_REMOVED_OR_DISABLED=NONE
SCHEDULED_TASK=PASS_RUNNING
EXECUTION_TRIGGER=PASS_ACTIVE
PRD_READONLY_VERIFICATION=BOUNDED_HUMAN_GATE_UNCHANGED
CHANGES_MADE=CHANNEL_V3_SUPPORT;HIDDEN_WSCRIPT_LAUNCHER;TASK_REINSTALL;REGRESSION_TESTS;STATUS_PUBLISHER_NATIVE_STDERR_HARDENING;REPORTING
CURRENT_PRODUCTION_STATE=UNCHANGED
DEPLOYMENT_STATE=REMOTE_OBJECTIVE_BRIDGE_RECONCILED_NO_APPLICATION_DEPLOYMENT
VALIDATION_RESULT=PASS_THREE_SCHEDULED_SYNCS_ZERO_VISIBLE_WINDOWS
ROLLBACK_USED=NO
ROLLBACK_RESULT=NOT_REQUIRED
PRODUCTION_MUTATION=NONE
BLOCKERS=NONE
PREEXISTING_PROJECT_GATE=EXISTING_ENROLLED_PRD_READONLY_PRIVATE_KEY_RESTORATION_REQUIRED
RISKS_OR_WARNINGS=PRD_INTERNAL_STATE_REMAINS_UNVERIFIED_UNTIL_EXISTING_AUDIT_KEY_IS_RESTORED
CURRENT_WORK=NONE
NEXT_STEP=RESTORE_EXISTING_ENROLLED_PRD_READONLY_PRIVATE_KEY_FROM_APPROVED_SECURE_BACKUP
ETA_OR_COMPLETION_STATE=COMPLETE
SECRET_SCAN=PASS
PUBLIC_REPORT_URL=https://status.itech3s.com/lite-llm/archive/20260904-165000-FIX-HIDDEN-REMOTE-SYNC-0003-PASS.md
```

## Root Cause

The canonical one-minute Scheduled Task directly launched `powershell.exe` under an interactive token. The task action did not originally request hidden execution, and adding `-WindowStyle Hidden` plus the task-level hidden flag still produced a visible top-level PowerShell window at each scheduled boundary. Process-window observation detected the flash at two consecutive task runs, confirming the reported popup came from the LITE.LLM remote-objective sync task rather than an unrelated scheduled task.

## Repair

- Extended the bridge's explicit supported channel set from version 2 only to versions 2 and 3 while preserving fail-closed rejection of unsupported versions.
- Added a local VBScript launcher that invokes the existing PowerShell sync script through `WScript.Shell.Run` with window style `0`, waits for completion, and returns the PowerShell exit code to Task Scheduler.
- Reconfigured the canonical task action to use `wscript.exe`; retained the one-minute interval, sign-in trigger, interactive-token identity, least-privilege run level, network requirement, overlap guard, atomic sync, and duplicate objective protection.
- Marked the task itself hidden and retained hidden PowerShell arguments as defense in depth.
- Reinstalled the bridge into the existing workspace-local durable `.remote-objective` state directory without resetting claim or execution history.
- Hardened the existing public-status publisher so harmless native Git stderr is evaluated by Git's exit code instead of becoming a terminating Windows PowerShell error.

## Acceptance Evidence

```text
REMOTE_OBJECTIVE_TESTS=PASS
UTF8_PRESERVATION=PASS
INVALID_CONTENT_FAIL_CLOSED=PASS
CHANNEL_VERSION_3=PASS
UNSUPPORTED_CHANNEL_FAIL_CLOSED=PASS
IDEMPOTENCY=PASS
DUPLICATE_OBJECTIVE_GUARD=PASS
SCHEDULED_TASK_BACKGROUND_MODE=PASS
PUBLISH_STATUS_TESTS=PASS
NATIVE_GIT_STDERR_HANDLING=PASS
REMOTE_OBJECTIVE_SHA256=15b3dd5893a6a42275f7a9c3ea714391be02c6fccfbfb34d16bbf828dbe3192f
REMOTE_SOURCE_LOCAL_EXACT_MATCH=PASS
OBSERVATION_WINDOW=2026-09-04T16:45:35+07:00_TO_2026-09-04T16:48:00+07:00
OBSERVATION_SAMPLES=547
NEW_SUCCESSFUL_SCHEDULED_SYNCS=3
VISIBLE_WINDOW_DETECTIONS=0
TASK_LAST_RUN_RESULT=0
TASK_STATE=ENABLED_READY
```

The scheduled task inventory contains one canonical task with two intended triggers: sign-in and the daily repeating one-minute trigger. The two query rows are not duplicate tasks; they share one task name and one action. No redundant task was removed or disabled.

The synchronized version-3 objective exactly matches the HTTPS source. Its active claim remained intact through the task reinstall and all scheduled cycles, while the last executed bootstrap objective remained recorded separately. Recent operational logs contain only timestamp, event, objective ID, and PASS/FAIL-style result fields.

## Boundaries

No PRD, QAS, GitLab protected main, GitHub production backup, Cloudflare, LiteLLM, 9Router, Portal, PostgreSQL, routing, provider, or Gateway Admin state changed. No new runtime or execution platform was introduced.

The earlier PRD read-only identity recovery gate is independent of this completed operator-experience repair. The existing enrolled private key still requires restoration from its approved secure backup before internal PRD state verification can resume.
