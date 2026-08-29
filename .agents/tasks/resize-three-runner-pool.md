---
title: Resize Madeena Organization Runner Pool from 5 to 3
document_id: AGENT-TASK-ADMIN-INFRA-002
version: 1.0
status: validated-published
language: en-US
last_updated: 2026-08-29
scope:
  - self-hosted runner pool resizing from 5 to 3
  - safe decommission of Slots 4 and 5
  - preservation of Slots 1, 2, and 3
  - .github/workflows/provision-self-hosted-runner.yml
  - robust 3-way concurrency proof and shared-host barrier
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Supersession note

**Supersedes:** [`.agents/tasks/bootstrap-five-runner-pool.md`](file:///var/www/admin-infra/.agents/tasks/bootstrap-five-runner-pool.md) @ `e3ed95dd273e7f98cd0032d0754f1ebda23a95fe`  
**Reason for supersession:**  
Five runners were successfully provisioned and demonstrated that they can accept GitHub Actions jobs (Workflow run `33201240003`). However, operational verification evidence (Workflow run `33245122569` at revision `89b81176df08aa227b945dff6178bc59ab5595d1`) confirmed that the production host's shared cellular/mobile-card/TP-Link outbound connection creates severe network contention during parallel action and dependency downloads (hitting 100-second HttpClient timeouts on `codeload.github.com`). Reducing the pool from 5 to 3 concurrent runners right-sizes the system to provide adequate concurrent worker capacity while reducing network, CPU, RAM, and I/O contention and operational maintenance overhead.

## Task identity

**Task title:**  
Resize Madeena Organization Runner Pool from 5 to 3

**Task path:**  
`.agents/tasks/resize-three-runner-pool.md`

**Task contract state:**  
`Validated/Published`

**Delivery objective / Work Package / MVP:**  
Work Package 02 — Three-Way Concurrent Organization Runner Pool & Safe Decommission

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

The `admin-infra` repository maintains GitHub Actions infrastructure automation for Madeena server administration and DevOps.

Previously, a five-runner organization pool was provisioned on the Linux production host across Slots 1 through 5. While all 5 runner services were successfully installed and active, verification evidence revealed that the host's shared cellular network connection suffers from download bottlenecks when multiple jobs download actions concurrently.

The designated operational decision is to resize the production runner pool from FIVE to THREE total runners on the same host:
- **Retain:**
  - Slot 1: `simama-production-server` in `/home/madeena/actions-runner-madeena-devops`
  - Slot 2: `simama-production-server-2` in `/home/madeena/actions-runner-madeena-devops-2`
  - Slot 3: `simama-production-server-3` in `/home/madeena/actions-runner-madeena-devops-3`
- **Safely Decommission & Retire:**
  - Slot 4: `simama-production-server-4` in `/home/madeena/actions-runner-madeena-devops-4`
  - Slot 5: `simama-production-server-5` in `/home/madeena/actions-runner-madeena-devops-5`

All retained runners remain in organization runner group `madeena-devops` with labels `[self-hosted, linux, x64, production]`. The existing runner architecture continues to use the local runner as the control plane; inbound SSH is strictly prohibited as a control mechanism.

## Baseline and task revision

**Implementation baseline:**  
`89b81176df08aa227b945dff6178bc59ab5595d1`

**Task revision:**  
Resolved upon publication commit to `main` (`.agents/tasks/resize-three-runner-pool.md` @ published Git commit SHA).

### Baseline and diff semantics

- The behavioral implementation baseline is `89b81176df08aa227b945dff6178bc59ab5595d1`.
- The Executor working-tree implementation changes MUST be strictly bounded to `.github/workflows/provision-self-hosted-runner.yml`.
- Review evaluates the implementation against implementation baseline `89b81176df08aa227b945dff6178bc59ab5595d1`, governing task revision, and the resulting implementation revision.

## Objective

Adapt `.github/workflows/provision-self-hosted-runner.yml` and provide the bounded operational procedure to:
1. Target exactly THREE TOTAL organization runner processes in runner group `madeena-devops` (`simama-production-server`, `simama-production-server-2`, `simama-production-server-3`).
2. Update all five-runner assumptions, constants (`TOTAL_RUNNERS=3`), matrix definitions (`[1, 2, 3]`), comments, and assertion logic.
3. Ensure the normal provisioning path preserves valid Slots 1–3, provisions only missing slots within the 3-runner target, never recreates Slots 4 or 5, and stops safely on ambiguous state.
4. Implement a robust 3-runner concurrency proof that avoids unnecessary network-dependent action downloads on the self-hosted path (e.g., via a host-local synchronization barrier across the shared-host runner processes) and sizes timeouts to tolerate cellular latency.
5. Provide explicit, bounded authority and execution steps for safely decommissioning Slots 4 and 5 (service stop, service uninstall, GitHub unregistration via temporary removal token, and confirmed directory cleanup) without touching retained Slots 1–3.

## Authoritative inputs

### Governing authority

- **Designated Human Directive / Approved Planning Decision:** Authorizing the pool resize from 5 to 3 runners on the production host, establishing the target topology (retaining Slots 1–3, retiring Slots 4–5), mandating safe decommission procedures, and prohibiting inbound SSH.
- **Repository AI Delivery Contract:** [`.agents/AGENTS.md`](file:///var/www/admin-infra/.agents/AGENTS.md)
- **Software Delivery Protocol:** [`.agents/software-workflow.md`](file:///var/www/admin-infra/.agents/software-workflow.md)
- **Historical Validated Task (Superseded):** [`.agents/tasks/bootstrap-five-runner-pool.md`](file:///var/www/admin-infra/.agents/tasks/bootstrap-five-runner-pool.md) @ `e3ed95dd273e7f98cd0032d0754f1ebda23a95fe`

### Observed implementation & operational evidence (Supporting reality, NOT requirement authority)

- [`.github/workflows/provision-self-hosted-runner.yml`](file:///var/www/admin-infra/.github/workflows/provision-self-hosted-runner.yml) @ `89b81176df08aa227b945dff6178bc59ab5595d1`: Current five-runner provisioning workflow with local bootstrap, curl-based API calls, sudo handling, and matrix verification.
- **Workflow Run 33201240003:** Successful provisioning of 5 runners on the production host.
- **Workflow Run 33245122569:** Verification run showing 4 runners succeeding, but 1 job timing out due to slow `actions/upload-artifact@v4` action download from `codeload.github.com` over cellular connection.
- **Observed Runner Group Configuration:** Runner group `madeena-devops` (`id: 3`, `visibility: selected`).
- **Configured Secrets:** `SELF_HOSTED_CREDENTIAL`, `SUDO_PASSWORD`, `GITHUB_TOKEN`.

### Requirement traceability

- `REQ-RESIZE-001` (Three-Runner Target Architecture) → Designated Human Directive: The target state is exactly THREE active organization runner processes in runner group `madeena-devops` (`simama-production-server`, `simama-production-server-2`, `simama-production-server-3`).
- `REQ-RESIZE-002` (Preservation of Slots 1–3) → Designated Human Directive: Retained Slots 1, 2, and 3 MUST NOT be deleted, re-registered, or destructively recreated. Valid configurations and services MUST be preserved.
- `REQ-RESIZE-003` (Prohibition of Slot 4/5 Re-provisioning) → Designated Human Directive: The provisioning logic MUST NOT provision, configure, or start Slots 4 or 5 under any normal circumstance.
- `REQ-RESIZE-004` (Safe Decommission of Slots 4 and 5) → Designated Human Directive & GitHub REST API: Slots 4 and 5 MUST be decommissioned only after strict identity validation, stopping and uninstalling systemd services via `svc.sh`, and unregistering via `config.sh remove --token <REMOVE_TOKEN>` using a freshly requested organization removal token (`POST /orgs/Madeena-software/actions/runners/remove-token`).
- `REQ-RESIZE-005` (Removal Token Protection) → Repository Security Policy: Organization runner removal tokens MUST be requested on-demand, masked immediately with `::add-mask::`, never logged, never persisted to disk, and never exported.
- `REQ-RESIZE-006` (Robust Concurrency Verification) → Designated Human Directive & Quality Audit: Concurrency proof MUST verify exactly 3 distinct runner names executing in parallel with positive time overlap (`overlap > 0`), designed to be resilient against cellular network latency (using shared-host synchronization or adequate timeout buffers and minimizing network action dependencies).
- `REQ-RESIZE-007` (Local Control Plane Invariant) → Designated Human Directive: Provisioning and verification MUST continue to run on the local self-hosted runner control channel (`runs-on: [self-hosted, linux, x64, production]`); inbound SSH is strictly prohibited.

## Scope

### In scope

- Modifying `.github/workflows/provision-self-hosted-runner.yml` to:
  1. Set `TOTAL_RUNNERS=3`.
  2. Update slot iteration and verification loop to range from Slot 2 to 3 (Slot 1 preserved).
  3. Ensure provisioning logic strictly stops at Slot 3 and does not touch or recreate Slots 4 and 5.
  4. Update verification matrix `job_index: [1, 2, 3]`.
  5. Update assertion job and script to expect exactly 3 runner proof records, 3 distinct runner names, and positive concurrency overlap (`overlap > 0`).
  6. Optimize the concurrency verification step for cellular connections (e.g., shared-host barrier in `/tmp/runner-concurrency-${GITHUB_RUN_ID}` and/or increased job timeouts).
  7. Update all job descriptions, summaries, messages, and comments from 5 runners to 3 runners.
- Defining the precise, bounded decommission authority and steps for retiring Slots 4 and 5 during Phase B live migration.

### Out of scope

- Modifying any other workflow file in `.github/workflows/`.
- Retiring, unregistering, or reconfiguring Slots 1, 2, or 3.
- Modifying runner group `madeena-devops` settings or labels.
- Using force-delete API endpoints (`DELETE /orgs/{org}/actions/runners/{runner_id}`) while runner host is accessible.
- Modifying repository secrets or exposing tokens.
- Introducing inbound SSH or external network control ports.

### Preserved behavior

- Slot 1 (`simama-production-server`), Slot 2 (`simama-production-server-2`), and Slot 3 (`simama-production-server-3`) remain running and intact.
- Runner group `madeena-devops` and labels `[self-hosted, linux, x64, production]` remain exact.
- Local execution via self-hosted runner without inbound SSH is preserved.
- Non-interactive sudo execution via `_sudo` helper function is preserved.
- SHA-256 validation of any downloaded runner package is preserved.
- Credential masking and non-persistence invariants remain strictly enforced.

## Dependencies and assumptions

### Dependencies

- Slots 1, 2, and 3 remain online and capable of accepting jobs for label `production`.
- Secret `SELF_HOSTED_CREDENTIAL` is valid for requesting registration and removal tokens for `Madeena-software`.
- Secret `SUDO_PASSWORD` is valid for executing `sudo ./svc.sh` commands on the host.

### Approved assumptions

- The production host is accessible via the self-hosted runner control channel.
- Slots 4 and 5 are located in `/home/madeena/actions-runner-madeena-devops-4` and `/home/madeena/actions-runner-madeena-devops-5`.
- Cellular internet connection has limited bandwidth; local inter-process communication on `/tmp` is fast and reliable for barrier synchronization.

### Remaining approval requirements

- Code review and verification of Stage A static implementation evidence by Repository Reviewer.
- Separate explicit human operational authorization before executing Phase B (live decommissioning of Slots 4 and 5) and Phase C (workflow dispatch for verification).

## Required capabilities

- Repository read and write for `.github/workflows/provision-self-hosted-runner.yml`.
- Local syntax and structure verification tools (YAML parser, Python syntax checker / AST compiler, bash syntax check).

## Execution constraints

### Constraints

- `TOTAL_RUNNERS` MUST be exactly 3.
- Under NO circumstance shall the workflow create, configure, or start runner instances for slot index > 3.
- **Decommission Bounded Authority:**
  - Decommission authority applies ONLY to:
    - `simama-production-server-4` (`/home/madeena/actions-runner-madeena-devops-4`)
    - `simama-production-server-5` (`/home/madeena/actions-runner-madeena-devops-5`)
  - Before removal, validate:
    - directory exists and is NOT a symlink
    - `.runner` contains exact expected `agentName`
    - `gitHubUrl` matches `https://github.com/Madeena-software`
    - `poolName` matches `madeena-devops`
  - Removal sequence:
    1. `cd "$RUNNER_DIR"`
    2. `_sudo ./svc.sh stop`
    3. `_sudo ./svc.sh uninstall`
    4. Request removal token via GitHub API: `POST /orgs/Madeena-software/actions/runners/remove-token`
    5. Mask token immediately: `echo "::add-mask::$REMOVE_TOKEN"`
    6. Run `./config.sh remove --token "$REMOVE_TOKEN"` as runner user (not root).
    7. Remove directory `rm -rf "$RUNNER_DIR"` ONLY after successful unregistration and identity confirmation.
- **Credential Protection Invariant:** `SELF_HOSTED_CREDENTIAL`, registration tokens, and removal tokens MUST NEVER be written to disk, logged, or exported.
- **Control Flow & Verification:**
  - `provision` is skipped when `inputs.verify_only == true`.
  - `verify-self-hosted` matrix has exactly 3 items: `[1, 2, 3]`.
  - Concurrency proof asserts exactly 3 distinct runner names with positive overlap.

## Acceptance criteria

- [ ] `.github/workflows/provision-self-hosted-runner.yml` is updated to target exactly 3 runners (`TOTAL_RUNNERS=3`).
- [ ] Stale five-runner references, comments, and matrix definitions are completely replaced with 3-runner semantics.
- [ ] Provisioning logic checks Slots 1–3, preserves valid slots, configures missing slots 2–3, and never touches Slots 4 or 5.
- [ ] Concurrency verification matrix runs 3 jobs on `[self-hosted, linux, x64, production]`.
- [ ] Assertion job expects exactly 3 proof records and verifies 3 distinct runner names with positive overlap (`overlap > 0`).
- [ ] Concurrency verification is resilient against cellular download delays (e.g. via shared-host barrier and adjusted timeouts).
- [ ] Workflow YAML and embedded Python scripts pass all static syntax and lint validations.
- [ ] Safe decommission procedure for Slots 4 and 5 is fully documented with strict identity checks and token protection.
- [ ] Phase B operational decommissioning removes only Slots 4 and 5 from systemd and GitHub organization topology.
- [ ] Final organization runner topology confirms `total_count = 3` with runners `simama-production-server`, `simama-production-server-2`, and `simama-production-server-3` all online.
- [ ] Idempotent subsequent execution of the 3-runner provisioning workflow preserves Slots 1–3 without recreating Slots 4 or 5.

## Verification requirements

### Stage A — Implementation / Static Verification (Authorized during implementation)

The Executor MUST perform and report:
1. **YAML Syntax & Structure Validation:** Validate `.github/workflows/provision-self-hosted-runner.yml` using `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/provision-self-hosted-runner.yml'))"` or equivalent.
2. **Bash Syntax Validation:** Validate all bash script blocks using `bash -n`.
3. **Python Syntax Validation:** Validate embedded Python code using `python3 -m py_compile` or `ast.parse()`.
4. **Stale Assumption Search:** Search for any remaining occurrences of "5", "five", "slot 4", "slot 5", "devops-4", "devops-5" in active provisioning or assertion logic.
5. **Diff Scope Check:** Verify that only `.github/workflows/provision-self-hosted-runner.yml` is modified.
6. **Stage A Terminal Outcome:** Report `REVIEW REQUIRED — STATIC IMPLEMENTATION EVIDENCE`.

### Stage B — Operational Verification (Requires explicit human authorization)

Upon receiving explicit human authorization to perform live migration and dispatch:
1. Execute safe decommission of Slot 4 and Slot 5 on the host.
2. Query GitHub API for organization runner list to verify Slots 4 and 5 are removed and Slots 1, 2, 3 remain online (`total_count == 3`).
3. Dispatch workflow with `verify_only: true` (or normal mode) to verify 3-job concurrency.
4. Verify all 3 jobs run on 3 distinct runner names with positive overlap.
5. Dispatch a second normal provisioning run to prove idempotency (Slots 1–3 preserved, no new slots created).

## Stop conditions

The Executor MUST stop implementation and return to planning if:
1. Any of Slots 1, 2, or 3 is offline or fails identity validation.
2. Slot 4 or 5 directory exists but contains unexpected metadata or symlinks.
3. Organization removal token cannot be obtained via API.
4. `svc.sh uninstall` or `config.sh remove` fails during decommission.
5. Organization runner topology becomes ambiguous or contains unexpected runners.
6. Execution would require inbound SSH or destructive changes to Slots 1–3.
7. Implementation baseline changes on `main`.

## Side-effect authorization

### Explicitly authorized side effects

- Modification of `.github/workflows/provision-self-hosted-runner.yml` within the repository workspace.
- Local static validation checks (YAML, bash, Python parsing).

All live operational actions (decommissioning services on the production host, GitHub API unregistration, workflow dispatch) require separate explicit human authorization under Stage B.

## Expected terminal outcome

### Review Required

Upon completing Stage A static verification:
- Working tree contains clean modifications to `.github/workflows/provision-self-hosted-runner.yml`.
- All static validation checks pass with zero errors.
- Text is prepared for Reviewer review and subsequent Executor Stage B execution.
