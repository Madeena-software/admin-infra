---
title: Update Madeena Server Root-Filesystem Disk-Alert Threshold to 80%
document_id: AGENT-TASK-ADMIN-INFRA-003
version: 1.0
status: Draft
language: en-US
last_updated: 2026-09-15
scope:
  - root filesystem disk-usage critical alert threshold configuration
  - identification and update of alert threshold from 50% to 80%
  - verification via authorized GitHub Actions diagnostic workflow
  - boundary classification between local execution environment and Madeena production host
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Task identity

**Task title:**  
Update Madeena Server Root-Filesystem Disk-Alert Threshold to 80%

**Task path:**  
`.agents/tasks/update-server-disk-alert-threshold.md`

**Task contract state:**  
`Draft`

Execution and review lifecycle states such as `In Execution`, `Review Required`, `Remediation Required`, and `Accepted` SHOULD normally be tracked by orchestration, review records, repository metadata, or another mechanism that preserves the exact governing task revision.

A lifecycle-status update MUST NOT silently replace the immutable task revision that governed an execution attempt.

When remediation materially changes this executable contract, edit the same stable task path, return it to Draft as needed, and republish it as a new immutable governing task revision before renewed execution.

**Delivery objective / Work Package / MVP:**  
Work Package 03 — Server Monitoring & Root-Filesystem Disk Alert Threshold Adjustment (50% -> 80%)

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

Operational monitoring on the Madeena server currently triggers critical alerts when the root filesystem (`/`) disk usage reaches 50%.

The designated operational authority has requested raising this critical-alert threshold from the observed 50% to 80% to avoid premature and noisy alerts, while maintaining appropriate warning headroom for server operations.

Repository inspection across the `admin-infra` Git repository (`305f223281040f3c3dbec3833b596314d2dd0d99`) confirms that no disk monitoring alert scripts, cron jobs, or threshold definitions are tracked in this Git repository. Workflows in `.github/workflows/` (such as `server-debug.yml`) inspect disk usage and check for an 80% threshold for audit reports, but do not emit recurring push/webhook alerts.

### Execution & Environment Boundary Clarification
- **Antigravity Local Shell ≠ Madeena Production Server:** The Madeena production server is not directly accessible through the local Antigravity execution shell. The shell environment is local execution-environment evidence only. The absence of cron jobs or monitoring scripts in the local shell does not imply absence on the production server.
- **Production Access Control Plane:** Production access is mediated strictly through repository-authorized GitHub Actions workflows running on `[self-hosted, linux, x64, production]`.
- **Diagnostic Discovery Requirement:** Determining the exact production monitoring implementation (e.g., cron job, systemd service/timer, external monitoring agent, or external repository) requires running an authorized non-destructive diagnostic workflow via GitHub Actions or inspecting production via designated runner jobs.

## Baseline and task revision

**Implementation baseline:**  
`305f223281040f3c3dbec3833b596314d2dd0d99`

**Task revision:**  
`resolved when published`

`resolved when published` is a Draft placeholder. It is not sufficient for T5.

Before this task is treated as `Validated/Published` or handed to an Executor, the exact immutable governing task revision MUST be resolvable.

For Git repositories, the preferred published task identity is:

```text
.agents/tasks/update-server-disk-alert-threshold.md @ <full Git commit SHA containing the governing task content>
```

## Objective

Locate the source configuration/script for the Madeena server root-filesystem disk-usage alert (via authorized non-destructive GitHub Actions diagnostic execution on the production runner), update the threshold value from 50% to 80%, and verify that the 80% threshold is active without causing false alerts or disrupting existing server workloads.

## Authoritative inputs

### Governing authority

- User Operational Request / Instruction: "Change the Madeena server root-filesystem disk-usage critical-alert threshold from the currently observed 50% to 80%."
- Execution Clarification: Production access boundary and background task timer / timeout policy.
- Repository AI Delivery Contract: `.agents/AGENTS.md`
- Normative Software Delivery Protocol: `.agents/software-workflow.md`
- Repository Orientation Map: `.agents/context/project.md`

### Requirement traceability

- `REQ-INFRA-ALERT-001` (Threshold update 50% -> 80%) → Human Operational Directive
- `REQ-INFRA-RUNNER-BOUNDARY` (Production mediation via GitHub Actions) → Execution Clarification
- `REQ-INFRA-EXEC-TIMEOUT` (Background task timeout policy: 5 min diagnostic / 15 min build) → Execution Clarification

## Scope

### In scope

- Identification of the active disk-usage alert mechanism on the Madeena production host using authorized GitHub Actions workflows running on `[self-hosted, linux, x64, production]`.
- Updating the critical alert threshold configuration or script from 50% to 80% for the root filesystem (`/`).
- Automated non-destructive verification that the threshold is configured to 80%.
- Documenting the exact monitoring mechanism, file path, and operational configuration discovered.

### Out of scope

- Direct production shell access from the local environment bypassing GitHub Actions.
- Inbound SSH access or alternative backdoor access to the production host.
- Altering thresholds for other filesystems or mounts (e.g., `/media/nextcloud-data` HDD mount).
- Modifying runner pool configurations, application containers, or unrelated workflows.
- Workflow dispatch without explicit human authorization.

### Preserved behavior

- Inbound SSH remains disabled / prohibited as a control mechanism.
- Docker Swarm stacks (`simama`, `madeena_cp`) and runner services (`actions-runner-madeena-devops*`) must not be disrupted.
- Existing alert channels (webhook, email, Telegram, or notification service) and alert message formats must remain preserved.

## Dependencies and assumptions

### Dependencies

- Execution of diagnostic and modification steps requires workflow execution on self-hosted runners labelled `[self-hosted, linux, x64, production]`.
- GitHub Actions workflow dispatch authorization from the repository owner / human authority.

### Approved assumptions

- The alert observed by the human originates either from a host cron job, systemd service, containerized monitor, or external repository webhook on the Madeena host.
- The 50% threshold was an intentional or default setting in that monitor and can be modified to 80% without breaking alerting logic.
- Workflows running on `runs-on: [self-hosted, linux, x64, production]` provide truthful production-server evidence.

### Remaining approval requirements

- **Designated Human Approval required before workflow dispatch:** Any GitHub Actions workflow dispatch reaching production runners (`runs-on: [self-hosted, linux, x64, production]`) must be explicitly approved by human authority prior to triggering.
- **Designated Human Approval before modifying production monitor:** Once the file/config is discovered, the proposed modification to 80% must be reviewed before committing or applying to production.

## Required capabilities

- Repository read and write (for tracking task and any workflow adjustments).
- Local git inspection and shell execution within bounded timeouts (max 5 minutes for diagnostic commands).
- GitHub Actions workflow inspection and dispatch (subject to human approval).

## Execution constraints

### Constraints

- Strict separation of environments: Local Antigravity shell outputs must be classified as local-environment evidence, not production evidence.
- No direct external network mutation or SSH access from local shell.
- Any background command must have an explicit timeout (max 5 minutes for exploratory/diagnostic; max 15 minutes for tests/builds). No orphaned background processes.
- Reuse existing diagnostic mechanisms (such as non-destructive audit jobs in `server-debug.yml` or dedicated diagnostic workflow steps) rather than inventing unmanaged tools.

## Acceptance criteria

- [ ] The exact production mechanism emitting the 50% disk alert (cron, script, timer, or container) is identified and verified via production runner evidence.
- [ ] The root-filesystem disk-usage critical alert threshold is updated from 50% to 80%.
- [ ] Non-destructive verification confirms that the alert script or service reads 80% as the critical threshold.
- [ ] No alert is fired when root filesystem usage is between 50% and 79%.
- [ ] Server services, Docker Swarm workloads, and runner pool remain undisturbed.

## Verification requirements

### Required checks

- Non-destructive diagnostic check on production runner to inspect existing crontabs (`/etc/cron*`, user crontabs), systemd services, and custom monitoring scripts in `/opt`, `/var/www`, or `/usr/local/bin`.
- Post-change inspection of the identified file/configuration to confirm 80% threshold string/logic.
- Simulation or dry-run test of the alert script showing alert suppression below 80% and alert trigger at or above 80%.

### Required evidence

The Executor MUST report:
- Specific GitHub Actions workflow run ID and job output establishing production state.
- Exact file path and line content of the threshold definition.
- Diff or content change demonstrating the update to 80%.
- Bounded execution log showing timeouts respected and no orphaned processes.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- Workflow dispatch cannot be authorized by human authority.
- The threshold is managed by a third-party managed service outside host control requiring external console authority.
- Production host runner is offline or unreachable.
- Any unexpected production error or service failure occurs.

## Side-effect authorization

### Explicitly authorized side effects

- Authoring and committing this task document to the isolated branch `task/server-disk-alert-threshold-80`.
- Local git status, diff, and branch inspections.
- No remote push, PR creation, workflow dispatch, or production modification is authorized during task authoring.

## Expected terminal outcome

### Planning Required / Review Required

- Task authoring completed as `Draft`.
- Returned to Planner/Reviewer to review task readiness (T5) and authorize publication or diagnostic workflow dispatch.
