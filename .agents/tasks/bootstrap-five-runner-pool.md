---
title: Bootstrap Five-Runner Organization Pool
document_id: AGENT-TASK-ADMIN-INFRA-001
version: 1.0
status: validated-published
language: en-US
last_updated: 2026-08-29
scope:
  - self-hosted runner provisioning adaptation
  - five-slot organization runner pool bootstrap
  - concurrency verification and assertion
  - .github/workflows/provision-self-hosted-runner.yml
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

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
Repository Planner / Madeena Admin Infra

## Delivery context

The `admin-infra` repository maintains GitHub Actions automation for server administration and deployment.

Currently, one organization-level self-hosted runner (`simama-production-server`, ID: 25) is active and online in the `madeena-devops` runner group on the target Linux production host (`[self-hosted, linux, x64, production]`).

The existing provisioning workflow `.github/workflows/provision-self-hosted-runner.yml` was originally authored to run on GitHub-hosted `ubuntu-latest` and connect via SSH to the target host to install runner services. However, because Slot 1 is already operational on the server, running the provisioning workflow directly on the existing self-hosted runner (`runs-on: [self-hosted, linux, x64, production]`) removes external SSH dependencies, eliminates network/firewall connection barriers, and enables direct local bootstrapping of four additional runner instances (Slots 2, 3, 4, and 5).

This task defines the contract for adapting `.github/workflows/provision-self-hosted-runner.yml` to perform local bootstrap on the existing self-hosted runner, bringing the total active runner pool to five concurrent runners in the `madeena-devops` group.

## Baseline and task revision

**Implementation baseline:**  
`4eea91539493308be4a9a680c4daab5f88fa09ce`

**Task revision:**  
Resolved upon publication commit to `main` (`.agents/tasks/bootstrap-five-runner-pool.md` @ published Git commit SHA).

## Objective

Adapt `.github/workflows/provision-self-hosted-runner.yml` so that the `provision` job executes on the existing Linux self-hosted runner (`runs-on: [self-hosted, linux, x64, production]`), downloads the latest actions-runner archive (or uses cached archive in `/tmp`), requests four fresh organization runner registration tokens via GitHub API using `SELF_HOSTED_CREDENTIAL`, configures and installs systemd services for runner slots 2 through 5 in dedicated local directories (`~/actions-runner-madeena-devops-2` through `5`) under runner group `madeena-devops` with labels `self-hosted,linux,x64,production`, preserves the existing Slot 1 (`simama-production-server`) instance, and validates five-way concurrent execution across the pool via the 5-job matrix verification and assertion steps.

## Authoritative inputs

### Governing authority

- Repository AI Delivery Contract: `.agents/AGENTS.md`
- Software Delivery Protocol: `.agents/software-workflow.md`
- Repository Context: `.agents/context/project.md`
- Existing Workflow Implementation: `.github/workflows/provision-self-hosted-runner.yml`
- Server Permission & Sudo Patterns: `.github/workflows/server-setup-deploy.yml`
- Verified Planning Evidence:
  - 1 active organization-level runner online (`simama-production-server`, ID 25, group `madeena-devops`, labels `self-hosted`, `linux`, `x64`, `production`).
  - GitHub secret `SELF_HOSTED_CREDENTIAL` configured with organization actions runner registration permissions.
  - GitHub secret `SUDO_PASSWORD` configured with sudo authorization on the runner host.

### Requirement traceability

- `REQ-RUNNER-001` (Self-Hosted Local Execution) → `.agents/context/project.md`: The provisioning job MUST run on the existing self-hosted runner environment (`runs-on: [self-hosted, linux, x64, production]`), eliminating external SSH requirements.
- `REQ-RUNNER-002` (Slot Isolation & Preservation) → `.github/workflows/provision-self-hosted-runner.yml`: Slot 1 (`simama-production-server` in `$HOME/actions-runner-madeena-devops`) MUST remain preserved and active; Slots 2 through 5 MUST be installed into distinct directories (`$HOME/actions-runner-madeena-devops-2` .. `-5`) with distinct runner names (`simama-production-${HOST_NAME}-2` .. `-5`) and separate systemd services.
- `REQ-RUNNER-003` (Registration Token Generation) → `.github/workflows/provision-self-hosted-runner.yml`: The workflow MUST generate fresh registration tokens for slots 2..5 (or all slots where registration is needed) via `gh api --method POST orgs/Madeena-software/actions/runners/registration-token` using `SELF_HOSTED_CREDENTIAL` (or `GH_TOKEN`).
- `REQ-RUNNER-004` (Group and Label Consistency) → `.agents/context/project.md`: All five runners MUST belong to runner group `madeena-devops` and advertise labels `self-hosted,linux,x64,production`.
- `REQ-RUNNER-005` (Five-Way Concurrency Verification) → `.github/workflows/provision-self-hosted-runner.yml`: The workflow MUST retain the 5-job verification matrix (`verify-self-hosted`) and concurrency assertion (`assert-runner-pool`) confirming 5 distinct runner processes execute in parallel with positive time overlap.

## Scope

### In scope

- Modifying `.github/workflows/provision-self-hosted-runner.yml` to:
  1. Change the `provision` job `runs-on` target from `ubuntu-latest` to `[self-hosted, linux, x64, production]`.
  2. Remove external SSH connectivity steps, SSH retry loops, and SSH key management from the provisioning job, replacing them with direct local shell execution on the host.
  3. Generate four fresh registration tokens for slots 2, 3, 4, and 5 using `SELF_HOSTED_CREDENTIAL` via the GitHub Organization API.
  4. Ensure the actions-runner tarball is cached/verified locally in `/tmp` against its published sha256 checksum.
  5. Configure runner instances 2 through 5 in `$HOME/actions-runner-madeena-devops-2` through `$HOME/actions-runner-madeena-devops-5`.
  6. Install and start systemd services for slots 2 through 5 using `SUDO_PASSWORD` (`echo "$SUDO_PASSWORD" | sudo -S -p '' ...` or helper function `_sudo`).
  7. Preserve the active configuration and service of Slot 1 (`$HOME/actions-runner-madeena-devops`).
  8. Retain the `verify-self-hosted` matrix job (`matrix.job_index: [1, 2, 3, 4, 5]`) and the `assert-runner-pool` assertion step.
  9. Retain the `verify_only` workflow dispatch boolean input.

### Out of scope

- Modifying any other workflow file in `.github/workflows/`.
- Altering or rotating GitHub repository secrets.
- Dispatching or executing live GitHub Actions workflow runs (reserved for operational execution after review/acceptance).
- Direct mutation of server state or runner files outside of workflow automation.
- Modifying repository branch protection or access control settings.

### Preserved behavior

- Slot 1 (`simama-production-server`) continues running uninterrupted.
- Runner group `madeena-devops` and runner labels `[self-hosted, linux, x64, production]` remain exact.
- Artifact upload/download schema for concurrency verification (`proof/runner-job-*.json`) remains unchanged.
- Concurrency verification threshold requiring exactly 5 distinct runner names with positive overlap (`overlap > 0`) remains intact.

## Dependencies and assumptions

### Dependencies

- Existing organization runner `simama-production-server` (ID 25) remains online and capable of accepting jobs for label `production`.
- Secret `SELF_HOSTED_CREDENTIAL` is valid and authorized to generate organization runner registration tokens for `Madeena-software`.
- Secret `SUDO_PASSWORD` is valid for executing `sudo ./svc.sh install` and `sudo ./svc.sh start` on the runner host.

### Approved assumptions

- The target server has sufficient memory, CPU, and process capacity to run 5 GitHub Actions runner listener processes concurrently.
- The `actions-runner` binary package supports multiple service instances on a single systemd host when installed in separate directories.
- Local command execution on the self-hosted runner runs under the runner user account with `$HOME` set to the user directory.

### Remaining approval requirements

- Code review and verification by the Repository Reviewer before merging or triggering production provisioning.
- Operational dispatch authorization before triggering the adapted workflow.

## Required capabilities

- Repository read and write for `.github/workflows/provision-self-hosted-runner.yml`.
- Local syntax and structure verification tools (YAML parser, Python syntax checker).

## Execution constraints

### Constraints

- The adapted workflow MUST use `set -Eeuo pipefail` across all bash steps.
- Sudo operations MUST be non-interactive via `echo "$SUDO_PASSWORD" | sudo -S -p '' <cmd>` to prevent workflow hangs.
- All registration tokens MUST be masked using `echo "::add-mask::$TOKEN"` before export.
- The configuration step for each slot MUST be idempotent: if `$RUNNER_DIR/.runner` exists, skip `./config.sh` and preserve existing registration.
- No secrets, tokens, or credentials may be written to disk outside of the standard `.runner` files managed by `./config.sh`.

## Acceptance criteria

- [ ] `.github/workflows/provision-self-hosted-runner.yml` is adapted so `provision` runs directly on `[self-hosted, linux, x64, production]`.
- [ ] SSH connection logic, key management, and retry loops are removed from the runner provisioning job.
- [ ] The workflow generates registration tokens for slots 2, 3, 4, and 5 using `SELF_HOSTED_CREDENTIAL` against `orgs/Madeena-software/actions/runners/registration-token`.
- [ ] Slots 2 through 5 are configured in `$HOME/actions-runner-madeena-devops-${i}` with names `simama-production-${HOST_NAME}-${i}`, group `madeena-devops`, and labels `self-hosted,linux,x64,production`.
- [ ] Slot 1 (`$HOME/actions-runner-madeena-devops`) is preserved and not reconfigured.
- [ ] Systemd services for slots 2..5 are installed and started via `svc.sh` with sudo.
- [ ] `verify-self-hosted` matrix runs 5 concurrent jobs on `[self-hosted, linux, x64, production]`.
- [ ] `assert-runner-pool` validates 5 distinct runner names with positive overlap.
- [ ] Workflow YAML is valid, properly formatted, and passes YAML syntax validation.

## Verification requirements

### Required checks

1. **YAML Syntax Validation:** Verify `.github/workflows/provision-self-hosted-runner.yml` with a YAML parser / linter.
2. **Python Syntax Validation:** Verify the inline python script in `assert-runner-pool` for valid syntax.
3. **Diff Scope Check:** Verify that only `.github/workflows/provision-self-hosted-runner.yml` is modified in the implementation diff.

### Required evidence

The Executor MUST report:
- Implementation git revision or exact diff.
- YAML and Python syntax validation results.
- Detailed breakdown of how local bootstrap replaces the SSH logic.
- Verification gaps or non-blocking observations.

## Stop conditions

The Executor MUST stop implementation and return to planning if:
- Organization runner registration API schema or requirements differ materially from the task contract.
- The existing self-hosted runner environment requires additional unapproved secrets or permissions.
- The task requires changes to other workflow files or external infrastructure outside the defined scope.
- Sudo execution on the host requires interactive prompts that cannot be resolved via `SUDO_PASSWORD`.

## Side-effect authorization

### Explicitly authorized side effects

- Modifying `.github/workflows/provision-self-hosted-runner.yml` in the local workspace working tree.

### Explicitly unauthorized actions for Executor

- Committing or pushing changes without explicit Planner/Repository authorization.
- Dispatching `workflow_dispatch` for `provision-self-hosted-runner.yml` or any other workflow.
- Connecting to or modifying server state directly via SSH or ad-hoc scripts.
- Modifying, reading, or resetting GitHub Actions repository secrets.

## Expected terminal outcome

### Review Required

The Executor will conclude in `Review Required` upon completing the workflow modification and providing the required validation evidence.

## Review and remediation handling

The Reviewer will evaluate the implementation against this contract, baseline `4eea91539493308be4a9a680c4daab5f88fa09ce`, and observed evidence. If accepted, the resulting revision will become the new accepted baseline.
