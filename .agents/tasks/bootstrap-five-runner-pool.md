---
title: Bootstrap Five-Runner Organization Pool
document_id: AGENT-TASK-ADMIN-INFRA-001
version: 1.1
status: validated-published
language: en-US
last_updated: 2026-08-29
scope:
  - self-hosted runner provisioning adaptation
  - five-slot organization runner pool bootstrap
  - two-stage verification and concurrency assertion
  - .github/workflows/provision-self-hosted-runner.yml
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Remediation note

**Review basis:** `4bb3954a08210f5b587e7dc786ecab1cb49d7e28`  
**Verdict:** `REMEDIATION REQUIRED`  
**Remediation focus:**
- Reverted unauthorized project context edits and removed machine-local link.
- Reclassified intended authority vs observed implementation evidence.
- Explicitly required inspection and repair of workflow verification dependencies and control-flow logic.
- Structured a two-stage verification model separating static implementation validation (Stage A) from human-authorized operational proof (Stage B).
- Expanded stop conditions to cover topology divergence, runner offline state, host capacity limits, and unauthorized dispatch.
- Refined idempotency, credential persistence invariants, and baseline/diff semantics.

## Task identity

**Task title:**  
Bootstrap Five-Runner Organization Pool via Existing Self-Hosted Runner

**Task path:**  
`.agents/tasks/bootstrap-five-runner-pool.md`

**Task contract state:**  
`Validated/Published`

**Delivery objective / Work Package / MVP:**  
Work Package 01 — Five-Way Concurrent Organization Runner Pool

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

The `admin-infra` repository maintains GitHub Actions automation for server administration and deployment.

Currently, one organization-level self-hosted runner (`simama-production-server`, ID: 25) is active and online in the `madeena-devops` runner group on the target Linux production host (`[self-hosted, linux, x64, production]`).

The existing provisioning workflow `.github/workflows/provision-self-hosted-runner.yml` was originally authored to run on GitHub-hosted `ubuntu-latest` and connect via SSH to the target host to install runner services. However, inbound SSH introduces external network dependencies, authentication barriers, and unnecessary attack surface. Because Slot 1 is already operating locally on the server, adapting the provisioning workflow to run directly on the existing self-hosted runner (`runs-on: [self-hosted, linux, x64, production]`) allows the existing runner to serve as the local bootstrap/control channel for reaching FIVE TOTAL organization runner processes, preserving Slot 1 and installing only missing Slots 2–5, without relying on inbound SSH.

This umbrella task governs the workflow adaptation, verification dependency repair, static verification, and the operational criteria required to prove 5-way concurrent execution across the pool.

## Baseline and task revision

**Implementation baseline:**  
`4eea91539493308be4a9a680c4daab5f88fa09ce`

**Task revision:**  
Resolved upon publication commit to `main` (`.agents/tasks/bootstrap-five-runner-pool.md` @ published Git commit SHA).

### Baseline and diff semantics

- The behavioral implementation baseline is `4eea91539493308be4a9a680c4daab5f88fa09ce`.
- Commits containing task publication or planning remediation are governance artifacts and do not constitute implementation changes.
- The Executor working-tree implementation change MUST be strictly bounded to `.github/workflows/provision-self-hosted-runner.yml`.
- Review evaluates the implementation against implementation baseline `4eea91539493308be4a9a680c4daab5f88fa09ce`, governing task revision, and the resulting implementation revision.

## Objective

Adapt `.github/workflows/provision-self-hosted-runner.yml` so that:
1. The `provision` job executes on the existing Linux self-hosted runner (`runs-on: [self-hosted, linux, x64, production]`), serving as the bootstrap channel for reaching FIVE TOTAL organization runner processes in runner group `madeena-devops` without relying on inbound SSH.
2. The workflow inspects each local slot idempotently, preserving Slot 1 (`$HOME/actions-runner-madeena-devops` / `simama-production-server`) and configuring/starting only missing slots among Slots 2 through 5 in `$HOME/actions-runner-madeena-devops-2` through `5`.
3. Registration tokens are requested via GitHub API using `SELF_HOSTED_CREDENTIAL` only for slots that actually require registration.
4. Services for configured slots are installed and started via `svc.sh` using `SUDO_PASSWORD` without interactive prompts.
5. The workflow job dependency graph and conditional execution are corrected so that:
   - In normal provisioning mode, verification (`verify-self-hosted`) and assertion (`assert-runner-pool`) execute cleanly upon successful provisioning.
   - In `verify_only=true` mode, provisioning is skipped while five-runner verification and assertion execute without failure or invalid context references.
6. The workflow satisfies static validation (Stage A) and provides the structure for operational proof (Stage B) asserting five distinct runner names with positive concurrency overlap.

## Authoritative inputs

### Governing authority

- **Designated Human Directive / Approved Planning Decision:** Grounding the delivery objective to achieve a five-runner organization pool using the existing self-hosted runner as the local bootstrap channel and prohibiting inbound SSH requirements.
- **Repository AI Delivery Contract:** [`.agents/AGENTS.md`](file:///var/www/admin-infra/.agents/AGENTS.md)
- **Software Delivery Protocol:** [`.agents/software-workflow.md`](file:///var/www/admin-infra/.agents/software-workflow.md)
- **Repository AI Delivery Bootstrap:** [`.agents/rules/antigravity-code-agent-workflow.md`](file:///var/www/admin-infra/.agents/rules/antigravity-code-agent-workflow.md)

### Observed implementation & operational evidence (Supporting reality, NOT requirement authority)

- [`.github/workflows/provision-self-hosted-runner.yml`](file:///var/www/admin-infra/.github/workflows/provision-self-hosted-runner.yml) @ `4eea91539493308be4a9a680c4daab5f88fa09ce`: Existing SSH-based runner provisioning and 5-job concurrency verification implementation.
- [`.github/workflows/server-setup-deploy.yml`](file:///var/www/admin-infra/.github/workflows/server-setup-deploy.yml) @ `4eea91539493308be4a9a680c4daab5f88fa09ce`: Existing repository patterns for non-interactive sudo execution (`echo "$SUDO_PASSWORD" | sudo -S -p '' ...`), self-hosted job execution, and environment handling.
- **Observed Organization Runner Topology:** 1 active organization runner online (`id: 25`, `name: simama-production-server`, `status: online`, `labels: [self-hosted, Linux, X64, production]`, `runner_group_id: 3`).
- **Observed Runner Group Configuration:** Runner group `madeena-devops` (`id: 3`, `visibility: selected`, `allows_public_repositories: true`).
- **Observed Secret Configuration:** GitHub Actions secrets `SELF_HOSTED_CREDENTIAL`, `SUDO_PASSWORD`, `SSH_HOST`, `SSH_USER` configured for repository `Madeena-software/admin-infra`.

### Requirement traceability

- `REQ-RUNNER-001` (Inbound SSH Elimination & Local Runner Bootstrap) → Designated Human Planning Directive: The provisioning job MUST execute on the existing self-hosted runner (`runs-on: [self-hosted, linux, x64, production]`) and MUST NOT require inbound SSH connections, SSH keys, or network port exposure.
- `REQ-RUNNER-002` (Five Total Runners & Slot 1 Preservation) → Designated Human Planning Directive: The target state is FIVE TOTAL active organization runner processes in runner group `madeena-devops`. Working Slot 1 (`simama-production-server` in `$HOME/actions-runner-madeena-devops`) MUST remain preserved and running; Slots 2 through 5 MUST be installed into distinct directories (`$HOME/actions-runner-madeena-devops-2` .. `-5`) with distinct names (`simama-production-${HOST_NAME}-2` .. `-5`) and separate systemd services.
- `REQ-RUNNER-003` (Registration Token Generation) → Designated Human Planning Directive & GitHub Actions API: Fresh organization registration tokens MUST be requested via `gh api --method POST orgs/Madeena-software/actions/runners/registration-token` using `SELF_HOSTED_CREDENTIAL` (or `GH_TOKEN`) only for slots requiring registration.
- `REQ-RUNNER-004` (Group and Label Consistency) → Designated Human Planning Directive: All five runners MUST belong to runner group `madeena-devops` and advertise labels `self-hosted,linux,x64,production`.
- `REQ-RUNNER-005` (Verification Dependency & Control-Flow Repair) → Planning Quality Audit: The workflow job dependency graph and conditional expressions MUST be corrected so `verify-self-hosted` and `assert-runner-pool` execute cleanly in both normal provisioning mode and `verify_only=true` mode without referencing unavailable or transitive `needs` contexts.
- `REQ-RUNNER-006` (Five-Way Concurrency Assertion) → Designated Human Planning Directive: The 5-job matrix verification (`verify-self-hosted`) and concurrency assertion script (`assert-runner-pool`) MUST verify that 5 distinct runner processes executed in parallel with positive time overlap.
- `REQ-RUNNER-007` (Credential Protection & Non-Persistence) → Repository Security Policy: `SELF_HOSTED_CREDENTIAL` and registration tokens MUST NEVER be explicitly written to disk or logged; only internal credentials created by `config.sh` are permitted; tokens must be masked.

## Scope

### In scope

- Modifying `.github/workflows/provision-self-hosted-runner.yml` to:
  1. Change the `provision` job `runs-on` target from `ubuntu-latest` to `[self-hosted, linux, x64, production]`.
  2. Remove external SSH connectivity steps, SSH retry loops, and SSH key management from the provisioning job, replacing them with direct local execution on the host.
  3. Inspect local slot directories and implement idempotent provisioning for slots 2..5 (preserve valid configured slots, configure missing slots, start inactive services, stop on ambiguous state).
  4. Generate registration tokens using `SELF_HOSTED_CREDENTIAL` via the GitHub API only for slots that require registration.
  5. Ensure the actions-runner tarball is cached/verified locally in `/tmp` against its published sha256 checksum.
  6. Configure missing runner instances in `$HOME/actions-runner-madeena-devops-2` through `$HOME/actions-runner-madeena-devops-5`.
  7. Install and start systemd services for slots 2 through 5 using `SUDO_PASSWORD` (`echo "$SUDO_PASSWORD" | sudo -S -p '' ...` or helper function `_sudo`).
  8. Preserve the active configuration and service of Slot 1 (`$HOME/actions-runner-madeena-devops`).
  9. Repair job dependencies and conditions between `provision`, `verify-self-hosted`, and `assert-runner-pool` to ensure valid execution in both normal and `verify_only=true` modes.
  10. Retain the `verify-self-hosted` matrix job (`matrix.job_index: [1, 2, 3, 4, 5]`) and the `assert-runner-pool` assertion step.
  11. Retain the `verify_only` workflow dispatch boolean input.

### Out of scope

- Modifying any other workflow file in `.github/workflows/` (e.g. `server-setup-deploy.yml`, `deploy-swarm.yml`).
- Altering, creating, or inspecting GitHub repository secret values.
- Dispatching or executing live GitHub Actions workflow runs without explicit human authorization.
- Direct mutation of server state or runner files outside of workflow automation.
- Modifying repository branch protection or access control settings.

### Preserved behavior

- Slot 1 (`simama-production-server`) continues running uninterrupted.
- Runner group `madeena-devops` and runner labels `[self-hosted, linux, x64, production]` remain exact.
- Artifact upload/download schema for concurrency verification (`proof/runner-job-*.json`) remains unchanged.
- Concurrency verification threshold requiring exactly 5 distinct runner names with positive overlap (`overlap > 0`) remains intact.

## Dependencies and preconditions

### Dependencies

- Existing organization runner `simama-production-server` (ID 25) remains online and capable of accepting jobs for label `production`.
- Secret `SELF_HOSTED_CREDENTIAL` is valid and authorized to generate organization runner registration tokens for `Madeena-software`.
- Secret `SUDO_PASSWORD` is valid for executing `sudo ./svc.sh install` and `sudo ./svc.sh start` on the runner host.

### Operational preconditions (To observe/verify, NOT assumed)

- The target server has sufficient memory, CPU, and process capacity to run 5 GitHub Actions runner listener processes concurrently.
- Local command execution on the self-hosted runner runs under the runner user account with write permissions to `$HOME`.
- Systemd supports independent runner service units (`actions.runner.Madeena-software.*`) without naming conflicts.

### Remaining approval requirements

- Code review and verification of Stage A static implementation evidence by Repository Reviewer.
- Separate explicit human operational authorization before dispatching the adapted workflow against live infrastructure.

## Required capabilities

- Repository read and write for `.github/workflows/provision-self-hosted-runner.yml`.
- Local syntax and structure verification tools (YAML parser, Python syntax checker).

## Execution constraints

### Constraints

- The adapted workflow MUST use `set -Eeuo pipefail` across all bash steps.
- Sudo operations MUST be non-interactive via `echo "$SUDO_PASSWORD" | sudo -S -p '' <cmd>` to prevent workflow hangs.
- All registration tokens MUST be masked using `echo "::add-mask::$TOKEN"` immediately upon receipt.
- **Credential Persistence Invariant:** `SELF_HOSTED_CREDENTIAL` / PAT and temporary registration tokens MUST NEVER be explicitly written to disk or recorded in step summaries, logs, artifacts, or commits. Standard internal configuration files created automatically by `config.sh` are permitted.
- **Idempotent Slot Handling:**
  - For each slot 1 through 5:
    - Valid configured slot (`[ -f "$RUNNER_DIR/.runner" ]` and service active) → preserve it.
    - Missing slot directory or config → configure and start service.
    - Configured but service stopped → start service.
    - Ambiguous / stale / partially configured state → STOP and escalate rather than destructively recreating without authority.
  - Working Slot 1 MUST NOT be reconfigured merely for uniformity.
- **Control-Flow Correctness:**
  - `provision` is skipped when `inputs.verify_only == true`.
  - `verify-self-hosted` executes after `provision` in normal mode, or immediately in `verify_only=true` mode.
  - `assert-runner-pool` executes after `verify-self-hosted` completes, and MUST NOT fail due to references to un-needed or skipped jobs.

## Acceptance criteria

- [ ] `.github/workflows/provision-self-hosted-runner.yml` is adapted so `provision` runs directly on `[self-hosted, linux, x64, production]`.
- [ ] Inbound SSH connection logic, SSH keys, and retry loops are removed from the runner provisioning job.
- [ ] The workflow generates registration tokens for missing slots using `SELF_HOSTED_CREDENTIAL` against `orgs/Madeena-software/actions/runners/registration-token`.
- [ ] Slots 2 through 5 are configured in `$HOME/actions-runner-madeena-devops-${i}` with names `simama-production-${HOST_NAME}-${i}`, group `madeena-devops`, and labels `self-hosted,linux,x64,production`.
- [ ] Slot 1 (`$HOME/actions-runner-madeena-devops`) is preserved and not reconfigured.
- [ ] Systemd services for missing/installed slots are installed and started via `svc.sh` with sudo non-interactively.
- [ ] Job dependency graph and conditions are corrected so that:
  - Normal provisioning triggers verification and assertion upon success.
  - `verify_only=true` skips provisioning and runs verification and assertion.
  - No invalid/transitive `needs` expressions exist.
- [ ] `verify-self-hosted` matrix runs 5 concurrent jobs on `[self-hosted, linux, x64, production]`.
- [ ] `assert-runner-pool` validates 5 distinct runner names with positive overlap.
- [ ] Workflow YAML and embedded Python pass syntax validation without errors.

## Verification requirements

### Stage A — Implementation / Static Verification (Authorized during implementation)

The Executor MUST perform and report:
1. **YAML Syntax & Structure Validation:** Validate `.github/workflows/provision-self-hosted-runner.yml` using a YAML parser / linter.
2. **Python Syntax Validation:** Validate the inline python script in `assert-runner-pool` with `python3 -m py_compile` or equivalent.
3. **Control-Flow & Dependency Inspection:** Verify that `needs:` declarations and `if:` conditions correctly support both `verify_only=false` and `verify_only=true` execution paths.
4. **Diff Scope Check:** Verify that only `.github/workflows/provision-self-hosted-runner.yml` is modified in the working tree.
5. **Stage A Terminal Outcome:** Report `REVIEW REQUIRED — STATIC IMPLEMENTATION EVIDENCE`.

### Stage B — Operational Verification (Requires explicit human authorization)

Upon receiving explicit human authorization to dispatch the workflow against live infrastructure:
1. Dispatch `workflow_dispatch` on `provision-self-hosted-runner.yml`.
2. Collect and report:
   - Provisioning workflow run ID and run URL.
   - Provisioning job conclusion and step summary.
   - Five verification job conclusions and execution timestamps.
   - Five distinct observed `RUNNER_NAME` values from proof artifacts.
   - Positive concurrency overlap value (`overlap > 0`) calculated by assertion.
   - Successful conclusion of `assert-runner-pool`.
   - Resulting organization runner count and status (`gh api orgs/Madeena-software/actions/runners --jq '.total_count'` -> 5).
   - Confirmation that Slot 1 remained operational throughout.
   - Partial-success or failure diagnosis if provisioning stops.

Final acceptance of the five-runner pool MUST NOT be claimed without Stage B operational evidence.

## Stop conditions

The Executor MUST stop implementation and return to planning if:
1. Bootstrap Slot 1 runner is unavailable or offline.
2. Runner group `madeena-devops` is missing or incompatible with required repository access.
3. Organization runner registration authorization via `SELF_HOSTED_CREDENTIAL` is unavailable or insufficient.
4. Implementation would require opening or exposing inbound SSH.
5. Implementation would require deleting, re-registering, stopping, or destructively altering the working bootstrap runner without new authority.
6. Actual server/runner topology materially differs from verified planning evidence.
7. Unexpected existing Slots 2–5 or stale/partial installations make preservation/remediation ambiguous.
8. Five TOTAL runners cannot be achieved without materially broader infrastructure changes.
9. Additional secret or permission expansion becomes necessary.
10. Host capacity appears insufficient for five concurrent runner listener/process workloads.
11. Governing implementation baseline becomes materially stale due to intervening repository changes.
12. Operational workflow dispatch would be required before explicit human authorization.
13. Acceptance criteria cannot be met within the current umbrella objective.

## Side-effect authorization

### Explicitly authorized side effects

- Modifying `.github/workflows/provision-self-hosted-runner.yml` in the local workspace working tree.
- Running local static syntax, lint, and parser checks.

### Explicitly unauthorized actions (Without separate explicit human authorization)

- Committing, pushing, or creating branches.
- Pull-request creation.
- Dispatching GitHub Actions workflows (`workflow_dispatch`).
- Creating, removing, or re-registering runners on live servers.
- Mutating server configuration or filesystem directly.
- Reading, modifying, or creating secrets in GitHub Actions.
- Exposing or configuring SSH.
- Deploying or releasing anything.

## Expected terminal outcome

- **Stage A:** `Review Required — Static Implementation Evidence` upon completing workflow modification and static validation.
- **Stage B (Post-Authorization):** `Review Required — Operational Evidence` upon executing the authorized workflow and gathering runtime proof.

## Review and remediation handling

The Reviewer will evaluate the implementation against this contract, baseline `4eea91539493308be4a9a680c4daab5f88fa09ce`, and observed evidence. If accepted, the resulting immutable revision will become the new accepted baseline.
