---
title: Update Madeena Server Disk-Alert Threshold to 80% (Unified Root and Data Partitions)
document_id: AGENT-TASK-ADMIN-INFRA-003
version: 2.0
status: Validated/Published
language: en-US
last_updated: 2026-09-16
scope:
  - disk-usage critical alert threshold configuration update from 50% to 80%
  - unified threshold applicable to root filesystem (/) and configured DATA_PARTITIONS via shared ROOT_DISK_THRESHOLD
  - verification via authorized GitHub Actions diagnostic/mutation workflow
  - boundary classification between local execution environment and Madeena production host
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Task identity

**Task title:**  
Update Madeena Server Disk-Alert Threshold to 80% (Unified Root and Data Partitions)

**Task path:**  
`.agents/tasks/update-server-disk-alert-threshold.md`

**Task contract state:**  
`Validated/Published`

The task file is the executable delivery contract.

Execution and review lifecycle states such as `In Execution`, `Review Required`, `Remediation Required`, and `Accepted` SHOULD normally be tracked by orchestration, review records, repository metadata, or another mechanism that preserves the exact governing task revision.

A lifecycle-status update MUST NOT silently replace the immutable task revision that governed an execution attempt.

When remediation materially changes this executable contract, edit the same stable task path, return it to Draft as needed, and republish it as a new immutable governing task revision before renewed execution.

**Delivery objective / Work Package / MVP:**  
Work Package 03 — Server Monitoring & Unified Disk Alert Threshold Adjustment (50% -> 80%)

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

Operational monitoring on the Madeena server currently triggers critical alerts when disk usage reaches 50%.

The designated human authority has clarified and approved that the existing shared Madeena disk-alert threshold shall become 80% for BOTH:
1. the root filesystem (`/`); and
2. all configured `DATA_PARTITIONS` monitored by `madeena-monitor.service`.

This supersedes the earlier requirement that non-root filesystem thresholds must remain unchanged. No separate `DATA_DISK_THRESHOLD` is required. The active monitor binary (`Madeena-software/madeena-server-monitor`) uses `RootDiskThreshold` for both root `/` and configured `DATA_PARTITIONS`, evaluating critical alerts with `>=`. Thus, updating `ROOT_DISK_THRESHOLD=50` to `ROOT_DISK_THRESHOLD=80` in the authoritative environment file achieves the desired unified behavior without modifying or recompiling the monitor binary.

### Established Production Facts (Discovered & Verified)
- **Active Monitor Service:** `madeena-monitor.service`
- **Executable:** `/var/www/madeena-server-monitor/monitor`
- **Working Directory:** `/var/www/madeena-server-monitor`
- **EnvironmentFile Authority:** `/var/www/madeena-server-monitor/.env`
- **Current Effective Disk Threshold:** `ROOT_DISK_THRESHOLD=50`
- **Configured Partitions:** Production has non-empty `DATA_PARTITIONS` both in `.env` and in the active process environment (`/proc/<pid>/environ`).
- **Binary Provenance:** Go build metadata verifies module `github.com/Madeena-software/madeena-server-monitor` at commit `a9be2681f07e9f5cea45de6b51f39b20f929dc1f` with `vcs.modified=false`.
- **Comparator Semantics:** Active monitor evaluates critical condition as `usage >= threshold`.

### Recorded Production Run 35046824964
Production run `35046824964` was an authorized production workflow dispatch on revision `dcbd564f3c3659412673f280ff9a51edf7cb9598`. It executed exactly once, verified service authority, confirmed binary provenance, detected effective non-empty `DATA_PARTITIONS`, and intentionally failed closed in Preflight 3 according to the earlier root-only contract. It performed zero configuration mutations, zero service restarts, and required no rollback because no transaction began. Run `35046824964` MUST NOT be retried; any subsequent production workflow dispatch requires new explicit human approval.

### Execution & Environment Boundary Clarification
- **Antigravity Local Shell ≠ Madeena Production Server:** The Madeena production server is not directly accessible through the local Antigravity execution shell. The shell environment provides local execution-environment evidence only.
- **Production Access Control Plane:** Production access is mediated strictly through repository-authorized GitHub Actions workflows running on `[self-hosted, linux, x64, production]`.
- **Diagnostic & Mutation Gating:** Workflow dispatch and production mutation require explicit designated human authorization.

## Baseline and task revision

**Implementation baseline:**  
`dcbd564f3c3659412673f280ff9a51edf7cb9598`

**Task revision:**  
`established upon commit and reported externally`

Before this task is treated as `Validated/Published` or handed to an Executor, the exact immutable governing task revision MUST be resolvable.

For Git repositories, the published task identity is:

```text
.agents/tasks/update-server-disk-alert-threshold.md @ <full Git commit SHA containing the governing task content>
```

The immutable revision is supplied externally by version-control history and reported to Planner/Reviewer orchestration. The task body does not embed a self-referential commit SHA.

## Objective

Change the critical disk-usage alert threshold from 50% to 80% for every filesystem currently monitored through the shared `ROOT_DISK_THRESHOLD` mechanism of `madeena-monitor.service`, including the root filesystem (`/`) and configured `DATA_PARTITIONS`, while preserving all unrelated monitoring and server behavior.

## Authoritative inputs

### Governing authority

- User Operational Directive & Requirement Clarification: "It is acceptable and intended for the existing shared Madeena disk-alert threshold to become 80% for BOTH root filesystem / and all configured DATA_PARTITIONS monitored by madeena-monitor.service."
- Verified Production Run Evidence: Run `35046824964` establishing active service, executable, EnvironmentFile, binary provenance (`a9be2681f07e9f5cea45de6b51f39b20f929dc1f`), and non-empty `DATA_PARTITIONS`.
- Monitor Source Architecture: `Madeena-software/madeena-server-monitor` uses `RootDiskThreshold` for both root and data partition evaluations.
- Repository AI Delivery Contract: `.agents/AGENTS.md`
- Normative Software Delivery Protocol: `.agents/software-workflow.md`
- Repository Orientation Map: `.agents/context/project.md`

### Requirement traceability

- `REQ-INFRA-ALERT-001` (Unified threshold update 50% -> 80% for root and data partitions) → Human Operational Directive
- `REQ-INFRA-RUNNER-BOUNDARY` (Production mediation strictly via GitHub Actions) → Execution Clarification
- `REQ-INFRA-EXEC-TIMEOUT` (Background task timeout policy: 5 min diagnostic / 15 min build) → Execution Clarification
- `REQ-INFRA-NO-SOURCE-CHANGE` (Preserve monitor binary and source without new abstractions) → Human Clarification & Architecture Authority

## Scope

### In scope

- Updating the critical alert threshold configuration `ROOT_DISK_THRESHOLD=50` to `ROOT_DISK_THRESHOLD=80` in the authoritative production EnvironmentFile (`/var/www/madeena-server-monitor/.env`) via authorized transactional workflow execution.
- Raising the effective alert threshold from 50% to 80% for the root filesystem (`/`).
- Raising the effective alert threshold from 50% to 80% for all configured `DATA_PARTITIONS` through the existing shared threshold mechanism.
- Updating workflow preflight and verification logic in `server-debug.yml` to permit non-empty `DATA_PARTITIONS`, verifying its configuration consistency and immutability via a safe non-secret digest/count fingerprint.
- Restart of only `madeena-monitor.service`.
- Verification that the restarted process uses `ROOT_DISK_THRESHOLD=80`, retains the exact pre-change `DATA_PARTITIONS` fingerprint, and that the service remains active.
- Deterministic synthetic comparator verification (`79.9 < 80` not critical, `80.0 >= 80` critical, `80.1 >= 80` critical).

### Out of scope

- Modifying `Madeena-software/madeena-server-monitor` source code.
- Introducing a separate `DATA_DISK_THRESHOLD` variable.
- Rebuilding or deploying a new monitor binary.
- Changing which `DATA_PARTITIONS` are configured or altering mount paths.
- Changing CPU, RAM, or temperature thresholds.
- Changing alert intervals, cooldown periods, alert recipients, or SMTP configuration.
- Changing alert message semantics or formatting.
- Modifying systemd service unit definitions (`madeena-monitor.service`).
- Docker Swarm workload modifications (`simama`, `madeena_cp`).
- GitHub Actions runner service pool modifications (`actions-runner-madeena-devops*`).
- SSD or HDD storage cleanup, file deletion, log rotation, cache pruning, or disk migration.
- Direct production SSH access or alternative bypasses from the local environment.

### Preserved behavior

- Exact `DATA_PARTITIONS` configuration and content.
- Monitor executable (`/var/www/madeena-server-monitor/monitor`).
- Service WorkingDirectory (`/var/www/madeena-server-monitor`).
- Service EnvironmentFile authority (`/var/www/madeena-server-monitor/.env`).
- Inbound SSH remains disabled as a control mechanism.
- Alert recipients and SMTP configuration.
- CPU, RAM, and temperature alert thresholds.
- Alert evaluation intervals and cooldown behavior.
- Comparator semantics (`usage >= threshold`).
- Docker Swarm workloads and production runner services.
- All unrelated `.env` values.

## Dependencies and assumptions

### Dependencies

- Workflow execution on self-hosted runner labeled `[self-hosted, linux, x64, production]`.
- Explicit human approval required before any GitHub Actions workflow dispatch.

### Approved assumptions

- The active monitor binary (`a9be2681f07e9f5cea45de6b51f39b20f929dc1f`) applies `RootDiskThreshold` to all configured filesystems; changing `ROOT_DISK_THRESHOLD` in `.env` to 80 uniformly adjusts critical evaluation for root `/` and all configured `DATA_PARTITIONS`.
- Non-empty `DATA_PARTITIONS` in production is expected and intended to evaluate against 80%.
- Workflows running on `[self-hosted, linux, x64, production]` provide truthful production-server evidence.

### Remaining approval requirements

- **Designated Human Approval required before production workflow dispatch:** Every production workflow dispatch requires prior explicit human authorization.
- **Production root usage gate:** If current root filesystem usage is already `>= 80%`, the workflow must abort before mutation to prevent activating a critical alert condition during configuration change.

## Required capabilities

- Repository read and write (for workflow and task definitions).
- Local Git inspection and shell execution within bounded timeouts.
- GitHub Actions workflow inspection and dispatch (strictly subject to human approval).

## Execution constraints

### Constraints

- Separation of environments: Local Antigravity shell outputs are local evidence, not production evidence.
- No direct network mutation or SSH access to production from the local shell.
- Production workflow timeout policy: default maximum 10 minutes.
- Transactional mutation with fail-closed rollback: any failure during preflight, mutation, restart, or verification must abort or revert `.env` to `ROOT_DISK_THRESHOLD=50` and ensure `madeena-monitor.service` is active.
- Confidentiality: Do not log or expose raw partition paths, passwords, or secret values. Fingerprint `DATA_PARTITIONS` using safe metadata/hash/count.
- Storage separation: SSD/HDD cleanup is a separate operational matter; this task authorizes zero file pruning or storage deletion.

## Acceptance criteria

- [ ] Exact production monitor authority remains verified (`madeena-monitor.service`, cwd `/var/www/madeena-server-monitor`, exec `/var/www/madeena-server-monitor/monitor`, EnvironmentFile `/var/www/madeena-server-monitor/.env`).
- [ ] Binary provenance remains verified: module `github.com/Madeena-software/madeena-server-monitor`, revision `a9be2681f07e9f5cea45de6b51f39b20f929dc1f`, `vcs.modified=false`.
- [ ] Before mutation, authoritative EnvironmentFile and active process environment both have `ROOT_DISK_THRESHOLD=50`.
- [ ] Existing `DATA_PARTITIONS` configuration is detected and recorded via safe fingerprint (digest/count) without exposing raw values.
- [ ] Production change modifies strictly `ROOT_DISK_THRESHOLD=50 -> 80` in `/var/www/madeena-server-monitor/.env`.
- [ ] `DATA_PARTITIONS` content is unchanged across the transaction (post-change fingerprint equals pre-change fingerprint).
- [ ] `madeena-monitor.service` restarts cleanly and is in `active` state.
- [ ] New process environment reflects `ROOT_DISK_THRESHOLD=80`.
- [ ] Root `/` evaluates against 80%.
- [ ] Configured `DATA_PARTITIONS` evaluate against the shared 80% threshold.
- [ ] Synthetic comparator verification proves:
  - `79.9 < 80` -> not critical
  - `80.0 >= 80` -> critical
  - `80.1 >= 80` -> critical
- [ ] No real disk filling or test-alert storm is performed.
- [ ] No unrelated production configuration or service is modified.
- [ ] If post-change verification fails, transactional rollback restores `ROOT_DISK_THRESHOLD=50` and restarts `madeena-monitor.service`.

## Verification requirements

### Required checks

- Preflight verification of service authority, binary provenance, and baseline threshold (`50`).
- Safe fingerprint recording of `DATA_PARTITIONS` before mutation.
- Current root filesystem disk usage check (`< 80%`).
- Post-restart inspection of `.env` and `/proc/<new_pid>/environ` confirming `ROOT_DISK_THRESHOLD=80`.
- Post-restart fingerprint verification confirming `DATA_PARTITIONS` identity.
- Synthetic dry-run test of comparator semantics (`79.9`, `80.0`, `80.1`).
- Verification that `madeena-monitor.service` is active and healthy.

### Required evidence

The Executor MUST report:
- Specific GitHub Actions workflow run ID, URL, and execution log establishing production state.
- Pre-mutation baseline values (`ROOT_DISK_THRESHOLD=50`, `DATA_PARTITIONS` presence and fingerprint).
- Atomic diff showing only `ROOT_DISK_THRESHOLD` updated to `80`.
- Post-mutation process inspection showing PID change, active state, and effective `ROOT_DISK_THRESHOLD=80`.
- Unchanged `DATA_PARTITIONS` fingerprint confirmation.
- Synthetic comparator evaluation output.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- Workflow dispatch is not authorized by human authority.
- Production host runner is offline, unreachable, or job is routed to an unexpected runner.
- Active monitor binary provenance does not match `a9be2681f07e9f5cea45de6b51f39b20f929dc1f` (`vcs.modified=false`).
- Initial root filesystem usage is already `>= 80%`.
- EnvironmentFile or active process environment has multiple ambiguous threshold or partition definitions.
- Post-restart verification fails and rollback is executed.
- Any unexpected production error or service failure occurs.

## Side-effect authorization

### Explicitly authorized side effects

- Authoring, committing, and pushing this validated task document to the isolated branch `task/server-disk-alert-threshold-80`.
- Local Git status, diff, and branch inspections.
- No direct commit to `main`, no workflow dispatch, and no production modification is authorized by this task publication alone.

## Expected terminal outcome

### Review Required

- When the updated task document is validated, published, committed, and pushed to the isolated branch `task/server-disk-alert-threshold-80`.
- Following subsequent authorized execution, when production verification evidence is collected and ready for Reviewer evaluation.

### Planning Required

- If workflow dispatch is denied, production preflight fails unexpectedly, or rollback occurs.
