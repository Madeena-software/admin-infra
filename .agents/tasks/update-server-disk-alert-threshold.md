---
title: Update Madeena Server Root-Filesystem Disk-Alert Threshold to 80%
document_id: AGENT-TASK-ADMIN-INFRA-003
version: 1.1
status: Validated/Published
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
`Validated/Published`

The task file is the executable delivery contract.

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
- **Antigravity Local Shell ≠ Madeena Production Server:** The Madeena production server is not directly accessible through the local Antigravity execution shell. The shell environment is local execution-environment evidence only. The absence of cron jobs or monitoring scripts in the local shell does not imply absence on the production server. Do not use ordinary local-shell `df`, `systemctl`, `crontab`, `ps`, or `find` results as production evidence.
- **Production Access Control Plane:** Production access is mediated strictly through repository-authorized GitHub Actions workflows running on `[self-hosted, linux, x64, production]`. A workflow job running on `[self-hosted, linux, x64, production]` provides truthful production evidence only when the actual dispatched job is verified to execute on the intended production runner.
- **Diagnostic Discovery Requirement:** Determining the exact production monitoring implementation requires running an authorized non-destructive diagnostic workflow via GitHub Actions.

## Baseline and task revision

**Implementation baseline:**  
`305f223281040f3c3dbec3833b596314d2dd0d99`

**Task revision:**  
`established upon commit and reported externally`

Before this task is treated as `Validated/Published` or handed to an Executor, the exact immutable governing task revision MUST be resolvable.

For Git repositories, the published task identity is:

```text
.agents/tasks/update-server-disk-alert-threshold.md @ <full Git commit SHA containing the governing task content>
```

The immutable revision is supplied externally by version-control history and reported to Planner/Reviewer orchestration. The task body does not embed a self-referential commit SHA.

## Objective

Change the Madeena production root-filesystem (`/`) critical disk-alert threshold from the currently observed 50% to 80% (locating the source configuration/script via authorized non-destructive GitHub Actions diagnostic execution on the production runner, updating the threshold value from 50% to 80%, and verifying that the 80% threshold is active without disrupting existing server workloads).

## Authoritative inputs

### Governing authority

- User Operational Request / Instruction: "Change the Madeena server root-filesystem disk-usage critical-alert threshold from the currently observed 50% to 80%."
- Execution Clarification: Production access boundary, diagnostic workflow gating, production mutation gating, and background task timeout policy.
- Repository AI Delivery Contract: `.agents/AGENTS.md`
- Normative Software Delivery Protocol: `.agents/software-workflow.md`
- Repository Orientation Map: `.agents/context/project.md`

### Requirement traceability

- `REQ-INFRA-ALERT-001` (Threshold update 50% -> 80%) → Human Operational Directive
- `REQ-INFRA-RUNNER-BOUNDARY` (Production mediation via GitHub Actions) → Execution Clarification
- `REQ-INFRA-EXEC-TIMEOUT` (Background task timeout policy: 5 min diagnostic / 15 min build) → Execution Clarification

## Scope

### In scope

- Identification of the active disk-usage alert mechanism on the Madeena production host using authorized non-destructive GitHub Actions workflow execution on `[self-hosted, linux, x64, production]`.
- Updating the critical alert threshold configuration or script from 50% to 80% for the root filesystem (`/`) following explicit production-action authorization.
- Automated non-destructive verification that the threshold is configured to 80% and that alert evaluation logic behaves as expected.
- Documenting the exact monitoring mechanism, file path, and operational configuration discovered.

### Out of scope

- Direct production shell access from the local environment bypassing GitHub Actions.
- Inbound SSH access or alternative backdoor access to the production host.
- Altering thresholds for other filesystems or mounts (e.g., `/media/nextcloud-data` HDD mount).
- Modifying runner pool configurations, application containers, or unrelated workflows.
- Workflow dispatch without explicit human authorization.
- Production modification without explicit review and approval following discovery.

### Preserved behavior

- Inbound SSH remains disabled / prohibited as a control mechanism.
- Docker Swarm stacks (`simama`, `madeena_cp`) and runner services (`actions-runner-madeena-devops*`) must not be disrupted.
- Preserve the existing production alert delivery channel(s), recipients, message semantics, and scheduling behavior actually discovered during production inspection, unless a directly necessary change is explicitly authorized by the governing task.

## Dependencies and assumptions

### Dependencies

- Execution of diagnostic and modification steps requires workflow execution on self-hosted runners labelled `[self-hosted, linux, x64, production]`.
- GitHub Actions workflow dispatch authorization from the repository owner / human authority.

### Approved assumptions

- The active monitoring implementation has not yet been identified. Possible implementation mechanisms (cron job, systemd timer/service, containerized monitor, external webhook, etc.) may be investigated, but none is authoritative until verified through production-runner evidence.
- The intended critical alert threshold for the root filesystem is 80% as authorized by human directive. Whether the discovered implementation can safely realize that value must be verified during execution.
- Workflows verified to execute on `runs-on: [self-hosted, linux, x64, production]` provide truthful production-server evidence.

### Remaining approval requirements

- **Designated Human Approval required before diagnostic workflow dispatch:** Production diagnostic execution is required, but workflow dispatch is NOT authorized by default. Before any dispatch:
  1. Identify the exact workflow file.
  2. Identify the exact job and steps intended to execute.
  3. Demonstrate that the steps are strictly non-destructive.
  4. Identify any secrets/permissions used (without exposing secret values).
  5. Specify the exact production evidence to be collected.
  6. Obtain explicit human approval before triggering dispatch.
- **Designated Human Approval required before production modification:** Discovery of the monitor does not authorize changing it. After discovery, return:
  1. Actual monitoring mechanism.
  2. Exact production file, configuration, or service involved.
  3. Current threshold representation.
  4. Proposed smallest change to 80%.
  5. Verification approach.
  6. Operational risk.
  7. Rollback/recovery method.
  Planner/Reviewer will determine whether the task authorizes the change or whether explicit production-action authorization is required before applying the mutation.

## Required capabilities

- Repository read and write (for tracking task and workflow files).
- Local git inspection and shell execution within bounded timeouts.
- GitHub Actions workflow inspection and dispatch (strictly subject to human approval).

## Execution constraints

### Constraints

- Strict separation of environments: Local Antigravity shell outputs are local-environment evidence, not production evidence.
- No direct external network mutation or SSH access from local shell.
- Background process timeout policy:
  - Exploration / diagnostic: default maximum 5 minutes.
  - Tests / builds / validation: default maximum 15 minutes.
  - A longer timeout must be technically justified and explicitly recorded.
  - Track task/process ID, purpose, start time, timeout deadline, and terminal result.
  - On timeout, terminate/cancel only Executor-owned processes safely, preserve partial evidence, report the timeout truthfully, and leave no orphaned processes.
- Reuse existing diagnostic mechanisms (such as non-destructive audit jobs in `server-debug.yml` or safe workflow steps) rather than inventing unmanaged tools.

## Acceptance criteria

- [ ] The exact production mechanism emitting the 50% disk alert is identified and verified via production runner evidence.
- [ ] The root-filesystem disk-usage critical alert threshold is updated from 50% to 80% following required approvals.
- [ ] Non-destructive verification confirms that the alert script or service reads 80% as the critical threshold.
- [ ] A representative/synthetic disk-usage value below 80% does not satisfy the disk critical condition, and a representative value meeting the configured critical boundary does satisfy it, according to the monitor's existing comparison semantics (evaluated safely via deterministic dry-run/unit test without manipulating production disk space or triggering alert storms).
- [ ] Server services, Docker Swarm workloads, and runner pool remain undisturbed.

## Verification requirements

### Required checks

- Non-destructive diagnostic check on production runner to inspect crontabs, systemd units, container definitions, and custom monitoring scripts.
- Post-change inspection of the identified file/configuration to confirm the 80% threshold string/logic.
- Safe deterministic evaluation (dry-run or synthetic parameter test) of the alert logic verifying boundary behavior without triggering real alerts.

### Required evidence

The Executor MUST report:
- Specific GitHub Actions workflow run ID, URL, and job output establishing production state.
- Exact file path and line content of the threshold definition.
- Diff or content change demonstrating the update to 80%.
- Bounded execution log showing timeouts respected and no orphaned processes.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- Workflow dispatch cannot be authorized by human authority.
- The threshold is managed by a third-party managed service outside host control requiring external console authority.
- Production host runner is offline or unreachable.
- Production diagnostic steps yield ambiguous or contradictory findings.
- Any unexpected production error or service failure occurs.

## Side-effect authorization

### Explicitly authorized side effects

- Authoring, committing, and pushing this validated task document to the isolated branch `task/server-disk-alert-threshold-80`.
- Local git status, diff, and branch inspections.
- No direct commit to `main`, no PR creation, no production workflow dispatch, and no production modification is authorized by this task publication.

## Expected terminal outcome

### Review Required

- When production diagnostic findings and proposed change are returned for review before modification.
- When implementation and non-destructive verification are completed with full evidence.

### Planning Required

- If workflow dispatch is denied, production runner is unreachable, or monitor is externally managed.
