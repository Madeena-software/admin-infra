---
title: Plan Server Storage Cleanup Readiness and Candidate Classification
document_id: AGENT-TASK-ADMIN-INFRA-005
version: 1.0
status: Validated/Published
language: en-US
last_updated: 2026-09-17
scope:
  - read-only targeted investigation closing unresolved findings from run 35181615898
  - exact Nextcloud Snap storage path resolution and mount mapping
  - exact MinIO service storage path resolution and SSD/HDD co-location mapping
  - Docker Root Dir mapping and physical vs logical size reconciliation
  - Docker detailed candidate inventory (stopped containers, unreferenced images, build cache, volumes)
  - targeted bounded root filesystem breakdown and targeted large-file scanning (avoiding unbounded timeouts)
  - Snap retained disabled revisions inventory and footprint estimation
  - systemd journal and log footprint assessment without vacuuming or rotation alteration
  - evidence-complete storage cleanup readiness matrix (categories, risk tiers, operational impact, approval requirements)
  - strictly read-only diagnostics with zero pruning, zero deletion, zero service restarts, and zero mutation
authority_note: A published validated task authorizes only the bounded implementation scope explicitly defined by the task and applicable approved repository authority. Observed repository evidence governs claims about current implementation reality but does not silently redefine the task or its intended authority.
---

# Executable Task

This file defines a bounded software-delivery contract for implementation.

A validated task MUST provide enough authority, scope, acceptance, verification, and stop-condition information for an Executor to proceed without inventing material product, requirement, architecture, scope, or approval decisions.

A task is not a generic coding recipe. Implementation technique remains the Executor's responsibility within the constraints established here.

## Task identity

**Task title:**  
Plan Server Storage Cleanup Readiness and Candidate Classification

**Task path:**  
`.agents/tasks/plan-server-storage-cleanup.md`

**Task contract state:**  
`Validated/Published`

The task file is the executable delivery contract.

Execution and review lifecycle states such as `In Execution`, `Review Required`, `Remediation Required`, and `Accepted` SHOULD normally be tracked by orchestration, review records, repository metadata, or another mechanism that preserves the exact governing task revision.

A lifecycle-status update MUST NOT silently replace the immutable task revision that governed an execution attempt.

When remediation materially changes this executable contract, edit the same stable task path, return it to Draft as needed, and republish it as a new immutable governing task revision before renewed execution.

**Delivery objective / Work Package / MVP:**  
Work Package 05 — Production Storage Cleanup Readiness, Targeted Investigation & Evidence-Complete Reclaim Matrix

**Owner / designated planning authority:**  
Designated Human Authority / Repository Planner

## Delivery context

In preceding storage diagnostic work (Governing Task: `.agents/tasks/audit-server-storage-usage.md @ 414982906529e220011f947373570c6422e3eb43`, Run ID `35181615898`, Commit `f3b1f90d7322049b015da2aa29f1fb6ac5466aef`), initial storage topology and utilization baseline were captured.

### Established Production Evidence (from Run 35181615898)
- **Root Physical Disk:** Samsung SSD 850 family, `sda`, non-rotational (`ROTA=0`).
- **Root Logical Volume:** `/dev/sda3 -> ubuntu-vg -> ubuntu-lv -> /`, ext4 filesystem.
- **Root Utilization:** 70% utilization, approximately 149.9 GiB used out of ~226.7 GiB total, leaving ~66 GiB free.
- **HDD Storage:** Toshiba `sdb`, rotational (`ROTA=1`).
- **Secondary Mount:** `/media/nextcloud-data -> /dev/sdb1`, ext4, 3% utilization (~84.1 GiB used of ~3.58 TiB).
- **Monitored Targets:** `DATA_PARTITIONS=/media/nextcloud-data`.
- **Nextcloud Snap Presence:** Snap loop devices mounted under `/snap/nextcloud/...`. However, exact Nextcloud `datadirectory` and actual storage path remain unresolved.
- **MinIO Service Presence:** `minio.service` exists, but its exact storage path and whether data resides on SSD or HDD remain unresolved.
- **Unbounded Root Scan Timeout:** Root-wide `du` and root-wide large-file find operations timed out after the default 30-second sub-step limits, leaving overall root breakdown partially incomplete.
- **Specific Targeted Directories Measured:**
  - `/home`: ~19 GiB
  - `/var/log`: ~2.9 GiB
  - `/var/www`: ~2.5 GiB
  - `/var/cache`: ~152 MiB
  - systemd journal: ~2.5 GiB
- **Docker Aggregate Utilization (via `docker system df`):**
  - Images: 117.8 GB (reclaimable 45.26 GB, 38%)
  - Containers: 539.5 MB (reclaimable 474.7 MB, 87%)
  - Local Volumes: 107.2 MB (reclaimable 10.41 MB, 9%)
  - Build Cache: 71.37 GB (reclaimable 17.41 GB, 24%)

### Critical Accounting & Safety Clarifications
1. **Docker Accounting Rule:** Docker category sizes MUST NOT be blindly summed and treated as physical filesystem usage because shared/image-layer accounting may overlap across images, containers, and build cache.
2. **Reclaimable Rule:** Docker `RECLAIMABLE` values reported by `docker system df` are cleanup candidates only, NOT deletion authorization.
3. **Strict Non-Destructive Boundary:** This readiness planning task is strictly read-only. Absolutely NO deletion, pruning, container removal, journal vacuuming, or configuration modification is authorized.

## Problem statement

While Run `35181615898` revealed that root SSD space is significantly occupied by Docker and other components, several key operational questions remain unresolved:
1. Exact location of Nextcloud user and application data (SSD root vs HDD `/media/nextcloud-data`).
2. Exact MinIO storage path and whether MinIO objects reside on SSD root or HDD.
3. Docker physical backing location (`Docker Root Dir`), exact correlation between `du /var/lib/docker` and logical `docker system df`, and granular status of unreferenced vs in-use images, containers, and build cache.
4. Detailed footprint of `/home` (~19 GiB), `/var/snap/nextcloud`, `/var/log`, and snap old revisions.
5. Realistic categorization of potential reclaimable space divided into Low-Risk, Medium-Risk, and High-Risk (prohibited) tiers.

Before any destructive cleanup, deletion, or pruning can be authorized by human operators, an evidence-complete cleanup readiness plan and targeted probe must establish the exact operational impact, dependencies, and risk classification.

## Implementation baseline

- Baseline commit: `f3b1f90d7322049b015da2aa29f1fb6ac5466aef`
- Baseline branch: `task/server-storage-cleanup-readiness` (branched from `f3b1f90d7322049b015da2aa29f1fb6ac5466aef`)
- Established audit workflow: `.github/workflows/server-storage-audit.yml`
- Control-plane caller: `.github/workflows/server-debug.yml`

## In-scope work

The Executor is authorized to perform targeted read-only implementation and diagnostics to close all unresolved findings:

### 1. Resolve Nextcloud Snap Storage
- Inspect safe Nextcloud Snap metadata:
  - `snap list nextcloud`
  - `snap services nextcloud`
  - `snap connections nextcloud` where useful (inspecting plug/slot attachments such as removable-media)
- Check existence of `/var/snap/nextcloud/current/nextcloud/config/config.php`.
- Extract ONLY the `datadirectory` key from `config.php` (e.g. using `sed`/`awk`/`grep` targeting `datadirectory`).
- **Redaction Rule:** Strictly prohibit printing any other configuration values (e.g., `passwordsalt`, `secret`, `dbpassword`, database credentials, admin credentials).
- Map the resolved data path to its underlying mount using `findmnt -T` and `df -P`.
- Measure bounded directory usage for:
  - `/var/snap/nextcloud`
  - configuration, log, and database subdirectories where safely distinguishable
  - data directory ONLY IF it resides on the root filesystem (`/`).
- **Exclusion Rule:** If the Nextcloud data directory resolves to `/media/nextcloud-data` or another secondary HDD mount, do NOT recursively scan user files.

### 2. Resolve MinIO Storage
- Inspect `minio.service` in read-only mode using `systemctl show minio.service -p ...`.
- Allowlisted systemd properties ONLY:
  - `ActiveState`
  - `FragmentPath`
  - `ExecStart`
  - `EnvironmentFiles`
  - `User`
  - `WorkingDirectory`
- **Redaction Rule:** Do NOT dump the full service environment.
- For each authoritative EnvironmentFile discovered:
  - Extract ONLY known storage-location keys (such as `MINIO_VOLUMES` or another clearly storage-specific MinIO path variable).
  - Explicitly prohibit printing: `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`, or any credential tokens.
- Map candidate path(s) using `findmnt -T` and `df -P`.
- Determine definitively whether MinIO data resides on SSD root or HDD.

### 3. Determine Docker Physical Backing & Reconcile Accounting
- Collect `docker info` allowlisting only:
  - `Docker Root Dir`
  - `Storage Driver`
  - Container and image counts
- Map `Docker Root Dir` using `findmnt -T`, `df -P`, and bounded `du -shx`.
- Reconcile why `du /var/lib/docker` and `docker system df` differ (e.g., overlay2 layer sharing, sparse files, buildkit cache storage under `overlay2` and `buildkit`).
- Explicitly avoid assuming that logical size equals unique physical disk footprint.

### 4. Resolve Docker Cleanup Candidates (Read-Only Inventory)
- Inventory Docker components without pruning or deletion:
  - Running containers (`docker ps --format ...`)
  - Stopped containers (`docker ps -a --filter "status=exited" --filter "status=created" --format ...`)
  - Images referenced by running/stopped containers
  - Current Docker Swarm service images (if Swarm is active, via `docker service ls`)
  - Unreferenced images (images not associated with any container or service)
  - Detailed build cache reclaimable status via supported Docker commands (`docker buildx du` or `docker system df -v`)
  - Volume references (`docker volume ls` and inspecting mount bindings)
- Classify candidates into 3 strict operational categories:
  - **Category A (Likely safe but performance-impacting):** Unreferenced build cache.
  - **Category B (Potentially removable but deployment/rollback sensitive):** Images not referenced by any container/service, and stopped exit-code-0 non-persisted containers.
  - **Category C (Prohibited without application review):** Docker volumes, database data directories, images required for active service rollback, and currently referenced images.
- An image must NOT be characterized as safe merely because `ACTIVE=0` in an aggregate summary.

### 5. Targeted Bounded Root Usage & Large Files
- Avoid repeating unbounded root-wide probes that timed out in run `35181615898`.
- Execute targeted, bounded scans (depth <= 2, per-command timeouts <= 15s) strictly on candidate locations:
  - `/home` (subdirectories and user caches)
  - `/var/snap` and `/var/snap/nextcloud`
  - `/var/log`
  - `/var/www`
  - Docker Root Dir
- Search for large files (e.g. >= 500 MiB) ONLY within targeted candidate directories rather than across `/`.
- Return path and size only; do not read or output file contents.

### 6. Snap Retained Revisions
- Inspect all installed snaps and revisions: `snap list --all`.
- Identify disabled / old revisions as potential cleanup candidates.
- Estimate physical footprint of `/var/lib/snapd/snaps/` old revision squashfs files.
- Prohibit removing any snap revision during this phase.

### 7. Journal and Application Log Retention
- Record systemd journal footprint (`journalctl --disk-usage`) and check active retention policy in `/etc/systemd/journald.conf` (e.g. `SystemMaxUse`, `SystemKeepFree`).
- Do NOT vacuum journals (`--vacuum-size` / `--vacuum-time` prohibited).
- Do NOT alter `logrotate` configs.

### 8. Cleanup Readiness Table Output
- Compile all findings into a structured Cleanup Readiness Matrix containing:
  - Category
  - Current observed usage
  - Estimated reclaimable amount
  - Evidence source
  - Operational risk (Low / Medium / High)
  - Dependency / rollback impact
  - Proposed future cleanup action
  - Requires separate approval? (Yes/No)

## Preserved behavior

- Zero deletion of files, directories, volumes, or snaps.
- Zero pruning of Docker build cache, images, containers, or volumes.
- Zero vacuuming or truncation of systemd journal or logs.
- Zero restart or state change of `nextcloud`, `minio`, `docker`, or system services.
- Zero modification to filesystems, partitions, LVM volumes, or mount points.
- Zero leakage or logging of sensitive keys, passwords, or configuration secrets.
- Full preservation of control-plane caller security boundaries in `server-debug.yml`.

## Execution constraints

1. **Strictly Read-Only Execution:** All probes must be read-only (`snap list`, `systemctl show`, `findmnt`, `df`, bounded `du -x`, `docker ps`, `docker system df`, `journalctl --disk-usage`). No mutating flags or commands are permitted.
2. **Strict Redaction & Allowlisting:** Full dumps of configuration files or systemd environments are prohibited. Target only specific keys (`datadirectory`, `MINIO_VOLUMES`).
3. **Bounded Probes & Timeouts:** Every filesystem scan must have explicit bounded depth and per-command timeouts (<= 15s) to guarantee completion within the workflow execution budget.
4. **Isolated Branching:** All work must remain on `task/server-storage-cleanup-readiness`. No direct commits to `main`.
5. **No Production Access Without Approval:** Implementation of the task document does NOT authorize workflow dispatch or production execution. Dispatch requires subsequent Reviewer approval.

## Acceptance criteria

- [ ] A dedicated task contract `.agents/tasks/plan-server-storage-cleanup.md` is authored, validated, committed, and pushed on `task/server-storage-cleanup-readiness`.
- [ ] The task accurately incorporates all established baseline evidence from run `35181615898` (SSD vs HDD, utilization, partial directory sizes, Docker aggregate breakdown).
- [ ] Specific targeted investigation procedures are defined for:
  - Nextcloud Snap data directory and underlying mount;
  - MinIO service storage path and SSD vs HDD co-location;
  - Docker Root Dir mapping and physical layer accounting reconciliation;
  - Docker stopped containers, unreferenced images, and build cache breakdown;
  - Bounded targeted root scans and large-file inventory;
  - Snap disabled revisions footprint;
  - Journal footprint and retention configuration.
- [ ] Candidate categorization explicitly separates low-risk, medium-risk, and prohibited/high-risk items with clear rollback and dependency implications.
- [ ] Required final cleanup readiness table format is clearly specified.
- [ ] Safety constraints explicitly forbid `docker prune`, `snap remove`, `journalctl --vacuum`, `rm`, `truncate`, service restarts, or configuration edits.
- [ ] Stop conditions and side-effect authorizations are strictly bounded.

## Verification requirements

### Required checks
- Verification that the task document satisfies `.agents/` task schema, structure, and quality gates.
- Verification that baseline commit `f3b1f90d7322049b015da2aa29f1fb6ac5466aef` is preserved.
- Verification that no workflow files or code changes were added or executed during this task-authoring phase.
- Verification that the working tree is clean and the task is published to the remote tracking branch.

### Required evidence
- Branch name: `task/server-storage-cleanup-readiness`.
- Immutable governing task commit SHA on branch `task/server-storage-cleanup-readiness`.
- Confirmation that no workflow implementation, no workflow dispatch, and no production access occurred.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- Any proposed diagnostic probe requires write access, container restart, service restart, or configuration mutation.
- A probe cannot determine storage paths without leaking credentials, passwords, or secret tokens.
- Secret passing or secret inheritance is requested or required.
- The Executor is requested to perform actual pruning, deletion, or cleanup.
- Unbounded probes that risk hanging or exhausting runner resources are encountered.

## Side-effect authorization

### Explicitly authorized side effects
- Authoring, committing, and pushing `.agents/tasks/plan-server-storage-cleanup.md` to the isolated branch `task/server-storage-cleanup-readiness`.
- Pushing the branch to `origin/task/server-storage-cleanup-readiness`.

### Explicitly prohibited side effects
- Workflow execution or dispatch.
- Production server access or mutation.
- Merging to `main`.
- Any file deletion, container pruning, volume removal, or journal truncation.

## Expected terminal outcome

### Review Required
- When this task document is authored, committed, and pushed to `task/server-storage-cleanup-readiness` for Reviewer approval prior to any workflow implementation or execution.

### Planning Required
- If new constraints or production access boundaries require redefining the scope or authorization model.
