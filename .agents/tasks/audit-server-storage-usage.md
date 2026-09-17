---
title: Audit Madeena Production Storage Topology and Utilization
document_id: AGENT-TASK-ADMIN-INFRA-004
version: 1.2
status: Validated/Published
language: en-US
last_updated: 2026-09-17
scope:
  - read-only mapping of production storage topology and disk utilization
  - reusable workflow architecture invoked via existing server-debug.yml control plane
  - root filesystem (/) capacity, LVM backing, and major consumer category breakdown
  - additional storage mount (/media/nextcloud-data) capacity and mapping
  - effective DATA_PARTITIONS monitored mount targets identification
  - Nextcloud and MinIO storage filesystem mapping and co-location determination
  - evidence-based classification of future cleanup candidates without mutation
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Task identity

**Task title:**  
Audit Madeena Production Storage Topology and Utilization

**Task path:**  
`.agents/tasks/audit-server-storage-usage.md`

**Task contract state:**  
`Validated/Published`

The task file is the executable delivery contract.

Execution and review lifecycle states such as `In Execution`, `Review Required`, `Remediation Required`, and `Accepted` SHOULD normally be tracked by orchestration, review records, repository metadata, or another mechanism that preserves the exact governing task revision.

A lifecycle-status update MUST NOT silently replace the immutable task revision that governed an execution attempt.

When remediation materially changes this executable contract, edit the same stable task path, return it to Draft as needed, and republish it as a new immutable governing task revision before renewed execution.

**Delivery objective / Work Package / MVP:**  
Work Package 04 — Production Storage Topology, Utilization Audit & Cleanup Candidate Identification

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

In preceding production operations (Run ID `35054440176`), the critical disk-usage alert threshold on the Madeena production server was unified and raised to 80% for both the root filesystem (`/`) and configured `DATA_PARTITIONS`.

Observed production evidence established that the root filesystem (`/dev/mapper/ubuntu--vg-ubuntu--lv`) is currently at 70% utilization, occupying approximately 157.6 GB (150.3 GiB) out of 237.7 GB (226.7 GiB) total capacity, leaving approximately 69.3 GB (66.1 GiB) available. In addition, an additional storage mount `/media/nextcloud-data` and an active `DATA_PARTITIONS` definition are present on the production runner environment.

To plan future storage management and remediation safely without service disruption or data loss, a comprehensive, read-only diagnostic audit is required. The audit must establish the physical and logical storage topology from reliable evidence, determine where Nextcloud and MinIO object/file data reside, analyze what constitutes the ~158 GB root filesystem consumption, and identify evidence-based cleanup candidates.

The audit itself MUST remain strictly read-only and fail-closed: it must not delete, prune, move, truncate, compact, rotate, or modify any production data, container, or configuration.

### Established Production Facts (Discovered & Verified)
- **Observed production runner:** `simama-production-server-2`
- **Runner group:** `madeena-devops`
- **Machine name:** `server`
- **Root Filesystem Device:** `/dev/mapper/ubuntu--vg-ubuntu--lv`.
- **Observed Baseline Root Usage (Run 35054440176):** Total ~237,708,024 KiB, Used ~157,657,264 KiB, Free ~69,281,200 KiB, Usage: 70%.
- **Known Additional Storage Mount:** `/media/nextcloud-data`.
- **Monitored Service:** `madeena-monitor.service` configured with exactly one active, non-empty `DATA_PARTITIONS` definition.
- **Access Control Plane:** Production access is mediated strictly via GitHub Actions self-hosted runner labeled `[self-hosted, linux, x64, production]`.

### Execution & Environment Boundary Clarification
- **Antigravity Local Shell ≠ Madeena Production Server:** The local shell environment contains repository code and development tools, but has no direct access to production. Production evidence must come solely from authorized GitHub Actions workflow runs.
- **Separate Human Approval for Workflow Dispatch:** Authoring, validating, and publishing this task does NOT authorize dispatching any production workflow. Every workflow run against production requires separate, explicit human authorization.
- **Dispatch Control Plane & Default Branch Constraint:** GitHub Actions `workflow_dispatch` requires the dispatched workflow file to exist on the default branch (`main`). Direct modification of `main` is outside current authority. Therefore, `.github/workflows/server-debug.yml` (which already exists on the default branch) serves as the manual dispatch entrypoint, and calls the dedicated `.github/workflows/server-storage-audit.yml` as a local reusable workflow (`workflow_call`), strictly gated to execute only on `task/server-storage-audit`.

## Baseline and task revision

**Implementation baseline:**  
`8175e0212cddeb34233af2fea18eeb242e0c0962`

**Task revision:**  
`established upon commit and reported externally`

Before this task is treated as `Validated/Published` or handed to an Executor, the exact immutable governing task revision MUST be resolvable.

For Git repositories, the published task identity is:

```text
.agents/tasks/audit-server-storage-usage.md @ <full Git commit SHA containing the governing task content>
```

The immutable revision is supplied externally by version-control history and reported to Planner/Reviewer orchestration. The task body does not embed a self-referential commit SHA.

## Objective

Produce a verified, read-only map of Madeena production storage topology and utilization, including root storage / root filesystem / LVM-backed root, additional storage mount /media/nextcloud-data, monitored data partitions, Nextcloud and MinIO storage placement, major root-filesystem consumers, and evidence-based cleanup candidates, without modifying production state. Execution is mediated via a reusable workflow architecture called by the existing manual-dispatch control-plane workflow on the isolated branch.

## Authoritative inputs

### Governing authority

- User Operational Directive & Task Handoff: "Produce a verified, read-only map of Madeena production storage topology and utilization, including root storage / root filesystem / LVM-backed root, additional storage mount /media/nextcloud-data, monitored data partitions, Nextcloud and MinIO storage placement, major root-filesystem consumers, and evidence-based cleanup candidates, without modifying production state."
- User Architectural Remediation Directive: Storage Audit Dispatch Architecture (reusable workflow called by existing `server-debug.yml` on branch gate `github.ref_name == 'task/server-storage-audit'`).
- Verified Predecessor Production Evidence: Workflow Run `35054440176` on commit `8175e0212cddeb34233af2fea18eeb242e0c0962`.
- Repository AI Delivery Contract: `.agents/AGENTS.md`
- Normative Software Delivery Protocol: `.agents/software-workflow.md`
- Repository Orientation Map: `.agents/context/project.md`
- Task Template: `.agents/tasks/_template.md`

### Requirement traceability

- `REQ-STORAGE-AUDIT-DISPATCH-ARCH` (Reusable workflow architecture via existing `server-debug.yml` without modifying default branch) → User Architecture Directive
- `REQ-STORAGE-AUDIT-TOPOLOGY` (Block-device, filesystem, and LVM topology mapping with explicit allowlisted columns) → User Directive
- `REQ-STORAGE-AUDIT-DATA-PARTITIONS` (Safe identification of monitored DATA_PARTITIONS mount targets) → User Directive
- `REQ-STORAGE-AUDIT-APP-PLACEMENT` (Nextcloud & MinIO storage filesystem mapping and co-location determination) → User Directive
- `REQ-STORAGE-AUDIT-ROOT-CONSUMERS` (Bounded root filesystem categorization and large-file inventory) → User Directive
- `REQ-STORAGE-AUDIT-CLEANUP-CANDIDATES` (Evidence-based classification of cleanup candidates) → User Directive
- `REQ-STORAGE-AUDIT-READONLY-SAFETY` (Strict read-only safety, zero mutation, timeout enforcement) → User Directive & Repository Policy
- `REQ-STORAGE-AUDIT-SECRET-ISOLATION` (Strict isolation of secrets, no secret inheritance to reusable audit workflow) → User Directive & Security Policy

## Scope

### In scope

- Implementation of a dedicated reusable GitHub Actions workflow (`.github/workflows/server-storage-audit.yml`) defining `workflow_call`, targeting `[self-hosted, linux, x64, production]`, with least-privilege `permissions: contents: read`.
- Addition of a storage-audit caller job in `.github/workflows/server-debug.yml` on branch `task/server-storage-audit` invoking `./.github/workflows/server-storage-audit.yml` (resolving from the same commit), strictly gated by `if: github.ref_name == 'task/server-storage-audit'`.
- Adjustment of the legacy `full-audit` job condition in `server-debug.yml` so that `full-audit` is skipped on `task/server-storage-audit` (runs only when `ref_name` is neither `task/server-disk-alert-threshold-80` nor `task/server-storage-audit`).
- Preservation of the existing `monitor-threshold-change` job in `server-debug.yml` (runs strictly on `task/server-disk-alert-threshold-80`).
- Strict secret isolation: the storage audit caller does NOT pass secrets (`secrets: inherit` is forbidden; no database, Nextcloud, MinIO, or host credentials passed).
- Runtime probe for tool availability prior to execution (`command -v <tool>` for diagnostic utilities, failing gracefully if optional tools are absent).
- Bounded, read-only inspection of block-device topology using `lsblk` with explicit allowlisted output columns (`NAME,TYPE,SIZE,FSTYPE,MOUNTPOINTS,MODEL,ROTA`), determining physical backing if reliable evidence exists, otherwise reporting `UNRESOLVED`. Serial numbers and WWN are excluded.
- Bounded, read-only inspection of filesystem and mount topology (`findmnt`, `df -P` for `/`, `/media/nextcloud-data`, monitored `DATA_PARTITIONS`, Nextcloud root, MinIO root).
- Mapping LVM hierarchy for `/dev/mapper/ubuntu--vg-ubuntu--lv` (`pvdisplay`/`pvs`, `vgdisplay`/`vgs`, `lvdisplay`/`lvs` or equivalent read-only queries if LVM tools are available; report UNRESOLVED if unavailable).
- Safe extraction of the configured `DATA_PARTITIONS` mount path(s) from `/var/www/madeena-server-monitor/.env` without dumping unrestricted configuration or secret variables.
- Determination of Nextcloud storage filesystem and path via allowlisted systemd, Docker container, or Docker Swarm service mount inspection (source, destination, mount type), redacting any secret or credential fields.
- Determination of MinIO storage filesystem and path via allowlisted systemd, Docker container, or Docker Swarm service inspection, without dumping environment variables (access keys, secret keys).
- Clear answer on whether Nextcloud and MinIO share the same underlying filesystem or use distinct filesystems (`SAME FILESYSTEM`, `DIFFERENT FILESYSTEMS`, or `UNRESOLVED` with explicit reason).
- Bounded root-filesystem depth analysis (e.g., `du -x --max-depth=1 /` with strict timeout) and high-level categorization (Docker, logs, databases, `/var/lib`, `/var/www`, `/home`, `/opt`, backups, caches).
- Bounded inventory of unusually large files on the root filesystem (e.g., files >= 1 GiB, top 50, path and size only, remaining strictly on `/`).
- High-level Docker aggregate disk usage (`docker system df` if Docker is present; report `NOT PRESENT / UNAVAILABLE` if absent or inaccessible), distinguishing images, containers, local volumes, and build cache.
- Systemd journal aggregate disk usage (`journalctl --disk-usage` if `journalctl` is available; report `UNRESOLVED` if unavailable).
- Evidence-based classification of storage findings into:
  1. Potentially reclaimable;
  2. Requires application-specific review;
  3. Must not touch without dedicated plan.
- Emission of a structured, concise Markdown audit report summarizing topology, consumption, and classification.

### Out of scope

- Direct modification or push to `main` (default branch).
- Any file, volume, container, image, or cache deletion or pruning (no `rm`, `truncate`, `vacuum`, `prune`).
- Any modification to `.env`, service units, or production configuration.
- Any service restart, container restart, or daemon reload.
- Any LVM or filesystem modification (`lvextend`, `lvreduce`, `vgextend`, `resize2fs`, etc.).
- Modification of monitor thresholds or alert intervals.
- Package installation or host dependency updates.
- Passing repository or host secrets to the storage audit workflow.
- Recursive scans of the additional storage mount (`/media/nextcloud-data`).
- Content inspection of user files, databases, or object storage.
- Exposure of disk serial numbers, WWN, passwords, tokens, private keys, or application environment dumps.
- Direct production SSH access or local execution bypass.

### Preserved behavior

- Zero production mutation: all commands must be strictly passive/read-only.
- The existing `monitor-threshold-change` job logic and conditions are strictly preserved.
- The legacy `full-audit` job logic is strictly preserved (only its execution condition is updated to skip the storage-audit branch).
- All running services, containers, and Swarm tasks remain undisturbed.
- Host and container process environments remain private and unexposed.
- Storage mount state, permissions, ownership, and volume mappings remain unchanged.

## Dependencies and assumptions

### Dependencies

- Manual dispatch entrypoint via `.github/workflows/server-debug.yml` on `[self-hosted, linux, x64, production]`.
- Reusable workflow invocation via `./.github/workflows/server-storage-audit.yml` resolving from the same commit on branch `task/server-storage-audit`.
- Explicit human approval required before any GitHub Actions workflow dispatch.
- Reusable audit workflow restricted to `permissions: contents: read` with no inherited secrets.

### Approved assumptions

- GitHub Actions supports same-repository local reusable workflow calls (`./.github/workflows/server-storage-audit.yml`) executed on the checked-out ref.
- The production runner environment's available diagnostic tools will be probed at runtime (`command -v`), and probes will adapt gracefully without failing the entire audit.
- The root filesystem `/dev/mapper/ubuntu--vg-ubuntu--lv` can be inspected locally with `du -x` without traversing into attached mounts.
- Diagnostic operations can safely complete within a bounded 5-minute total job timeout, with individual commands bounded by strict sub-timeouts (e.g., 30–60s).

### Remaining approval requirements

- **Designated Human Approval required before production workflow dispatch:** Task authoring and implementation do NOT grant dispatch authority; execution against production requires explicit, separate human authorization following review of both workflow files.
- **Audit Plan and Implementation Review:** The workflow implementations (`server-debug.yml` caller and `server-storage-audit.yml` reusable) must be reviewed and accepted prior to dispatch.

## Required capabilities

- Repository read and write (for authoring `.github/workflows/server-storage-audit.yml`, updating caller in `server-debug.yml`, and task documentation).
- Local Git inspection and shell execution within bounded timeouts.
- GitHub Actions workflow inspection and dispatch capability (contingent upon human authorization).

## Execution constraints

### Constraints

- **Dispatch Architecture:** `server-debug.yml` remains the manual `workflow_dispatch` entrypoint. On branch `task/server-storage-audit`, a caller job invokes `./.github/workflows/server-storage-audit.yml`. No modification of `main` is permitted.
- **Exact Branch Gating:**
  - Storage audit caller job in `server-debug.yml` runs only when `github.ref_name == 'task/server-storage-audit'`.
  - Legacy `full-audit` job in `server-debug.yml` is skipped on `task/server-storage-audit` and `task/server-disk-alert-threshold-80`.
  - Legacy threshold change job in `server-debug.yml` runs only on `task/server-disk-alert-threshold-80`.
- **Secret Isolation:** The storage-audit caller MUST NOT pass repository secrets (`secrets: inherit` is prohibited; no database, Nextcloud, MinIO, or host credentials). If a secret-derived path is genuinely needed to resolve topology, STOP and return to Planner/Reviewer rather than passing secrets.
- **Token Permissions:** The reusable audit workflow uses least-privilege `permissions: contents: read`. No write permissions are granted.
- **Strict Read-Only Execution:** Every diagnostic probe must be non-destructive and read-only.
- **Runtime Tool Probing:** Test tool availability using safe read-only checks (`command -v <tool>`). Do not install missing packages or dependencies. If a tool is missing or inaccessible, report the exact status/limitation (`NOT PRESENT / UNAVAILABLE` or `UNRESOLVED`) and continue.
- **Explicit Allowlisted Output Columns:** `lsblk` must use explicit allowlisted columns (`NAME,TYPE,SIZE,FSTYPE,MOUNTPOINTS,MODEL,ROTA`). Do not request serial or WWN fields.
- **No Unrestricted Environment/Secret Dumps:** Under no circumstances may `/proc/<pid>/environ`, container `.env` files, or MinIO/Nextcloud credentials be dumped to workflow logs or artifacts.
- **Diagnostic Timeout Budget:** Hard job timeout of 5 minutes (`timeout-minutes: 5`) on the reusable audit job; individual expensive probes (e.g. `du`) must be protected by internal timeouts (e.g., `timeout 60s`). If a probe times out, it must fail gracefully, record the timeout as `UNRESOLVED (timeout)`, and allow remaining probes to proceed.
- **Process Safety:** In the event of a probe timeout, only processes spawned by the audit probe itself may be killed.
- **Fail-Closed Reporting:** If an item cannot be inspected safely or times out, report it as `UNRESOLVED` with explicit reason rather than guessing or expanding permissions.
- **Filesystem Isolation:** Root usage analysis must strictly use `-x` (or `--one-file-system`) to prevent descending into `/media/nextcloud-data` or other mounted filesystems.
- **Separation of Filesystem vs. Directory Metrics:** Do not sum percentages across different filesystems; clearly separate total storage capacity from directory-level consumed bytes.

## Acceptance criteria

- [ ] A dedicated reusable read-only workflow (`.github/workflows/server-storage-audit.yml`) defining `workflow_call` is authored, committed to `task/server-storage-audit`, and validated.
- [ ] `.github/workflows/server-debug.yml` remains the manual `workflow_dispatch` entrypoint and is updated on `task/server-storage-audit` with a caller job referencing `./.github/workflows/server-storage-audit.yml`.
- [ ] Storage audit caller job in `server-debug.yml` runs strictly when `github.ref_name == 'task/server-storage-audit'`.
- [ ] Legacy `full-audit` job in `server-debug.yml` is skipped on `task/server-storage-audit` (runs only when `ref_name` is neither `task/server-disk-alert-threshold-80` nor `task/server-storage-audit`).
- [ ] Existing `monitor-threshold-change` job in `server-debug.yml` remains unchanged and skipped on `task/server-storage-audit`.
- [ ] Reusable workflow call resolves from the same running commit without requiring changes to the repository default branch (`main`).
- [ ] No repository secrets are passed to the reusable audit workflow (`secrets: inherit` is not used).
- [ ] Reusable audit workflow has only `permissions: contents: read` and targets `[self-hosted, linux, x64, production]`.
- [ ] Hard timeout of 5 minutes (`timeout-minutes: 5`) is enforced on the audit job.
- [ ] Block-device topology is captured with explicit allowlisted fields (`NAME,TYPE,SIZE,FSTYPE,MOUNTPOINTS,MODEL,ROTA`). Physical backing (rotational/non-rotational/SSD/HDD) is reported as RESOLVED with evidence or UNRESOLVED with explicit reason; no premature inference.
- [ ] Serial numbers, WWN, and unnecessary hardware identifiers are explicitly excluded.
- [ ] Filesystem and mount topology maps `/`, `/media/nextcloud-data`, and monitored `DATA_PARTITIONS` targets with capacity, used, free, and percentage utilization (or UNRESOLVED with explicit reason if inaccessible).
- [ ] LVM hierarchy is mapped for `/dev/mapper/ubuntu--vg-ubuntu--lv` (PV -> VG -> LV -> Filesystem) if LVM diagnostic tools are available; reported as UNRESOLVED with explicit reason if tooling or permissions are insufficient.
- [ ] Configured `DATA_PARTITIONS` mount target(s) are safely identified from monitor configuration without exposing secrets or dumping full environment files (or UNRESOLVED with explicit reason if inaccessible).
- [ ] Nextcloud effective storage filesystem and path are identified via allowlisted runtime inspection without exposing credentials (or UNRESOLVED with explicit reason if inaccessible/not running).
- [ ] MinIO effective storage filesystem and path are identified via allowlisted runtime inspection without exposing access/secret keys (or UNRESOLVED with explicit reason if inaccessible/not running).
- [ ] The report explicitly establishes whether Nextcloud and MinIO share the same filesystem or use distinct filesystems, with valid terminal states: `SAME FILESYSTEM`, `DIFFERENT FILESYSTEMS`, or `UNRESOLVED` (with explicit reason).
- [ ] Root filesystem utilization is categorized by major consumers (`/var/lib/docker`, `/var/log`, `/var/lib`, `/var/www`, `/home`, `/opt`, backups, caches) using bounded `du -x` (or UNRESOLVED with explicit reason if timed out/inaccessible).
- [ ] Top large files (>= 1 GiB, up to 50 entries) on the root filesystem are inventoried by path and size only (or UNRESOLVED with explicit reason if timed out).
- [ ] Docker disk usage breakdown (images, containers, local volumes, build cache) is reported via `docker system df` if Docker is active; if Docker is absent or inaccessible, reported as `NOT PRESENT / UNAVAILABLE`.
- [ ] Systemd journal disk usage is reported via `journalctl --disk-usage` if available, or reported as `UNRESOLVED` with explicit reason if unavailable.
- [ ] Findings are categorized into evidence-based cleanup candidates (reclaimable, requires application review, must not touch) without authorizing mutation.
- [ ] Zero unauthorized mutation and zero destructive operations are verified: no files deleted, no containers pruned, no services restarted. All probe failures, unavailable tools, permission denials, and timeouts are explicitly reported with no silent omission or fabricated result.

## Verification requirements

### Required checks

- Workflow syntax and structure validation for both `.github/workflows/server-debug.yml` and `.github/workflows/server-storage-audit.yml` (local YAML syntax check, GitHub Actions schema conformance).
- Verification of branch conditions: storage audit caller executes only on `task/server-storage-audit`, `full-audit` is skipped on both task branches, `monitor-threshold-change` runs only on threshold branch.
- Verification that reusable workflow call passes no secrets (`secrets: inherit` absent).
- Verification that all commands in the audit workflow use read-only flags, probe tool availability, and prohibit mutation.
- Verification that timeouts (`timeout-minutes`, command-level `timeout`) are properly configured.
- Verification of redaction and allowlist filtering for storage variables and runtime inspects.

### Required evidence

The Executor MUST report:
- Implementation Git revision on branch `task/server-storage-audit`.
- Static workflow verification output for both workflow files.
- Following authorized execution:
  - Workflow run ID, URL, runner machine, and execution duration.
  - Complete block-device, filesystem, and LVM topology findings (with justified UNRESOLVED entries where applicable).
  - Nextcloud and MinIO storage paths and co-location status (`SAME FILESYSTEM`, `DIFFERENT FILESYSTEMS`, or `UNRESOLVED`).
  - Root filesystem consumer breakdown and large-file inventory.
  - Formulated cleanup candidate classification.
  - Confirmation of:
    - zero unauthorized mutation;
    - zero destructive operations;
    - all probe failures, unavailable tools, permission denials, and timeouts explicitly reported;
    - no silent omission or fabricated result.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- An audit probe requires destructive commands or write access.
- A required observation cannot be completed without dumping passwords, access keys, or secret tokens.
- Secret passing or inheritance to the reusable workflow is demanded.
- Production access outside GitHub Actions self-hosted runners is required.
- Workflow execution cannot be constrained to the 5-minute diagnostic time budget.
- The Executor is asked to perform file deletion, container pruning, or storage cleanup.
- Any unexpected storage or data-integrity risk is encountered.

## Side-effect authorization

### Explicitly authorized side effects

- Authoring, committing, and pushing this validated task document to the isolated branch `task/server-storage-audit`.
- Authoring, committing, and pushing `.github/workflows/server-storage-audit.yml` to `task/server-storage-audit` during execution.
- Updating, committing, and pushing the caller job in `.github/workflows/server-debug.yml` on `task/server-storage-audit` during execution.
- Local syntax and schema validation checks.
- No direct commit to `main`, no workflow dispatch, and no production modification is authorized by this task publication.

## Expected terminal outcome

### Review Required

- When the task document is validated, published, committed, and pushed to the isolated branch `task/server-storage-audit`.
- Following subsequent authorized implementation, when both workflow definitions are ready for Reviewer evaluation prior to dispatch.
- Following subsequent authorized production execution, when diagnostic findings and evidence are ready for Reviewer evaluation.

### Planning Required

- If production access is denied, probes cannot execute safely within constraints, or new authority is required.
