---
title: Plan Server Storage Cleanup Readiness and Candidate Classification
document_id: AGENT-TASK-ADMIN-INFRA-005
version: 1.1
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
  - workflow implementation of read-only diagnostic caller and reusable workflow under dedicated branch
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
- **Historical Root-Wide Timeout Evidence:**
  - `du -x --max-depth=1 /` reached bounded timeout: 60 seconds -> TIMEOUT.
  - `find / -xdev -type f -size +1G` reached bounded timeout: 60 seconds -> TIMEOUT.
- **Specific Targeted Directories Measured:**
  - `/home`: ~19 GiB
  - `/var/log`: ~2.9 GiB
  - `/var/www`: ~2.5 GiB
  - `/var/cache`: ~152 MiB
  - systemd journal: ~2.5 GiB
- **Docker Aggregate Utilization (via `docker system df`):**
  - Images: 117.8 GB logical (reclaimable 45.26 GB, 38%)
  - Containers: 539.5 MB logical (reclaimable 474.7 MB, 87%)
  - Local Volumes: 107.2 MB logical (reclaimable 10.41 MB, 9%)
  - Build Cache: 71.37 GB logical (reclaimable 17.41 GB, 24%)
- **Docker Directory Measurement:**
  - `du /var/lib/docker`: ~1.4G

### Critical Accounting & Safety Clarifications
1. **Docker Accounting Rule:** Docker reports a large logical storage footprint while the root filesystem is 70% utilized; the physical/unique contribution of Docker to root usage has not yet been reconciled and is a primary objective of this task. Docker category sizes MUST NOT be blindly summed and treated as physical filesystem usage because shared/image-layer accounting may overlap across images, containers, and build cache.
2. **Reclaimable Rule:** Docker `RECLAIMABLE` values reported by `docker system df` are cleanup candidates only, NOT deletion authorization.
3. **Probe Timeout Policy Distinction:**
   - Historical baseline timeout = 60s (probes timed out root-wide).
   - Future targeted probe policy = <= 15s per command (strictly scoped to candidate subdirectories to guarantee deterministic workflow execution).
4. **Strict Non-Destructive Boundary:** This readiness planning task is strictly read-only. Absolutely NO deletion, pruning, container removal, journal vacuuming, or configuration modification is authorized.

## Problem statement

While Run `35181615898` established initial hardware topology and disk capacity, several key operational questions remain unresolved:
1. Exact location of Nextcloud user and application data (SSD root vs HDD `/media/nextcloud-data`).
2. Exact MinIO storage path and whether MinIO objects reside on SSD root or HDD.
3. Docker physical backing location (`Docker Root Dir`), physical vs logical size reconciliation between `du /var/lib/docker` (~1.4G) and logical `docker system df` (~189 GB aggregate), and granular status of unreferenced vs in-use images, containers, and build cache.
4. Detailed footprint of `/home` (~19 GiB), `/var/snap/nextcloud`, `/var/log`, and snap old revisions.
5. Realistic categorization of potential reclaimable space divided into Low-Risk, Medium-Risk, and High-Risk (prohibited) tiers.

Before any destructive cleanup, deletion, or pruning can be authorized by human operators, an evidence-complete cleanup readiness plan and targeted probe must establish the exact operational impact, dependencies, and risk classification.

## Implementation baseline

- Baseline commit: `f3b1f90d7322049b015da2aa29f1fb6ac5466aef`
- Baseline branch: `task/server-storage-cleanup-readiness`
- Established audit workflow: `.github/workflows/server-storage-audit.yml`
- Control-plane caller: `.github/workflows/server-debug.yml`

## In-scope work

Following review and approval of this task contract, the Executor is authorized to implement the repository diagnostic workflows on branch `task/server-storage-cleanup-readiness` to close all unresolved findings:

### 1. Workflow Architecture & Dispatch Model
- Modify `.github/workflows/server-debug.yml`:
  - Add branch-gated caller job `cleanup-readiness` running strictly when `github.ref_name == 'task/server-storage-cleanup-readiness'`.
  - Invoke local reusable workflow `./.github/workflows/server-storage-cleanup-readiness.yml` via `workflow_call`.
  - Maintain job isolation:
    - `monitor-threshold-change`: skipped (`if: github.ref_name == 'task/server-disk-alert-threshold-80'`);
    - `storage-audit`: skipped (`if: github.ref_name == 'task/server-storage-audit'`);
    - `full-audit`: skipped on this branch (`github.ref_name != 'task/server-storage-cleanup-readiness'`).
- Create reusable workflow `.github/workflows/server-storage-cleanup-readiness.yml`:
  - Defined with `on: workflow_call`.
  - Permissions strictly limited to `permissions: contents: read`.
  - Runner target: `runs-on: [self-hosted, linux, x64, production]`.
  - Job timeout: <= 10 minutes (individual probe steps <= 15 seconds).
  - No secret inheritance (`secrets: inherit` strictly forbidden).
  - Use `sudo -n` only where available; fail-closed if permissions are insufficient.

### 2. Fail-Closed UNRESOLVED Outcome Requirement
For every diagnostic probe, valid outcomes MUST strictly be:
- `RESOLVED with evidence`, OR
- `UNRESOLVED (<explicit reason>)`.
Access MUST NEVER be escalated or broadened to force resolution.

### 3. Resolve Nextcloud Snap Storage
- Inspect safe Nextcloud Snap metadata:
  - `snap list nextcloud`
  - `snap services nextcloud`
  - `snap connections nextcloud` where useful (e.g. plug/slot attachments such as removable-media)
- Check existence of `/var/snap/nextcloud/current/nextcloud/config/config.php`.
- Extract ONLY the `datadirectory` key from `config.php` (e.g. using `sed`/`awk`/`grep` targeting `datadirectory`).
- **Redaction Rule:** Strictly prohibit printing any other configuration values (e.g., `passwordsalt`, `secret`, `dbpassword`, database credentials, admin credentials).
- Map the resolved data path to its underlying mount using `findmnt -T` and `df -P`.
- Measure bounded directory usage for:
  - `/var/snap/nextcloud`
  - configuration, log, and database subdirectories where safely distinguishable
  - data directory ONLY IF it resides on the root filesystem (`/`).
- **Exclusion Rule:** If the Nextcloud data directory resolves to `/media/nextcloud-data` or another secondary HDD mount, do NOT recursively scan user files.
- If data directory cannot be safely determined: report `UNRESOLVED (<reason>)`.

### 4. Resolve MinIO Storage
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
  - Extract ONLY known storage-location keys (primarily `MINIO_VOLUMES` or another clearly documented storage-location variable established from local service configuration).
  - Explicitly prohibit printing: `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`, or any credential tokens.
- Resolve the effective MinIO storage filesystem when safely observable using `findmnt -T` and `df -P`; otherwise report `UNRESOLVED (<reason>)` with the exact limiting evidence.

### 5. Determine Docker Physical Backing & Reconcile Accounting
- Collect `docker info` allowlisting only:
  - `Docker Root Dir`
  - `Storage Driver`
  - Container and image counts
- Map `Docker Root Dir` using `findmnt -T`, `df -P`, and bounded `du -shx`.
- Investigate read-only evidence relevant to the observed discrepancy between `docker system df` (~189 GB aggregate) and filesystem `du /var/lib/docker` (~1.4G).
- Check evidence-discovered Docker/container runtime paths (e.g. BuildKit storage, containerd backing directories, overlay2 directories under the confirmed Docker Root Dir).
- Distinguish the four non-interchangeable values:
  1. logical Docker accounting;
  2. unique/physical filesystem usage;
  3. reclaimable according to Docker;
  4. operationally approved for future removal.

### 6. Resolve Docker Cleanup Candidates (Read-Only Inventory)
- Inventory Docker components without pruning or deletion:
  - Running containers (`docker ps --format ...`)
  - Stopped containers (`docker ps -a --filter "status=exited" --filter "status=created" --format ...`)
  - Images referenced by running/stopped containers
  - Current Docker Swarm service images (if Swarm is active, via `docker service ls`)
  - Unreferenced images (images not associated with any container or service)
  - Detailed build cache reclaimable status via supported Docker commands (`docker buildx du` or `docker system df -v`)
  - Volume references (`docker volume ls` and inspecting mount bindings)
- Classify candidates into 3 strict operational categories:
  - **Category A (Lower operational risk):** Demonstrably unreferenced build cache (still performance-impacting on future builds).
  - **Category B (Deployment/rollback sensitive):** Images not referenced by current containers/services, and stopped containers after state/dependency review (do NOT label as inherently "safe").
  - **Category C (Protected / Prohibited without dedicated application review):** Docker volumes, database data directories, current running-service images, established rollback-critical artifacts, and persistent application state.
- Do not infer rollback irrelevance solely because an image is not currently active.

### 7. Targeted Bounded Root Usage & Large Files
- Avoid repeating unbounded root-wide probes that timed out in run `35181615898`.
- Execute targeted, bounded scans (depth <= 2, per-command timeouts <= 15s) strictly on candidate locations:
  - `/home` (subdirectories and user caches)
  - `/var/snap` and `/var/snap/nextcloud`
  - `/var/log`
  - `/var/www`
  - Docker Root Dir
- Search for large files (e.g. >= 500 MiB) ONLY within targeted candidate directories rather than across `/`.
- Return path and size only; do not read or output file contents.

### 8. Snap Retained Revisions
- Inspect all installed snaps and revisions: `snap list --all`.
- Identify disabled / old revisions as potential cleanup candidates.
- Estimate physical footprint of `/var/lib/snapd/snaps/` old revision squashfs files where practical.
- Prohibit removing any snap revision.

### 9. Effective Journal & Log Retention Configuration
- Record systemd journal footprint (`journalctl --disk-usage`).
- Inspect effective retention configuration using a bounded read-only mechanism that accounts for `/etc/systemd/journald.conf` and relevant configuration drop-ins (`/etc/systemd/journald.conf.d/*.conf`, `/run/systemd/journald.conf.d/*.conf`).
- Report ONLY storage/retention-relevant fields:
  - `SystemMaxUse`
  - `SystemKeepFree`
  - `SystemMaxFileSize`
  - `MaxRetentionSec`
- Do NOT vacuum journals (`--vacuum-size` / `--vacuum-time` prohibited).
- Do NOT alter `logrotate` configs.

### 10. Cleanup Readiness Table Output
The diagnostic execution deliverable MUST produce a structured Cleanup Readiness Matrix containing:
- Category
- Observed physical/logical usage
- Estimated reclaimable amount
- Evidence source
- Resolution status (`RESOLVED` / `UNRESOLVED`)
- Operational risk (Low / Medium / High)
- Dependency / rollback impact
- Proposed future cleanup action
- Separate approval required? (Yes/No)

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
5. **Separation of Implementation vs Dispatch:** Implementing the workflow in the repository does NOT authorize workflow dispatch. Production dispatch requires separate explicit human authorization.

## Acceptance criteria

The executable implementation of this task must satisfy:
- [ ] `.github/workflows/server-debug.yml` is updated with branch-gated caller job `cleanup-readiness` active only on `task/server-storage-cleanup-readiness`.
- [ ] Isolation verified: `monitor-threshold-change`, `storage-audit`, and `full-audit` are skipped on this branch.
- [ ] Dedicated reusable workflow `.github/workflows/server-storage-cleanup-readiness.yml` is created with:
  - `permissions: contents: read`;
  - No secret inheritance or credential exposure;
  - 10-minute workflow timeout and <= 15s per-command timeouts;
  - Safe `sudo -n` fail-closed logic.
- [ ] Diagnostic probes implemented with strict fail-closed `RESOLVED` / `UNRESOLVED` semantics for:
  - Nextcloud Snap `datadirectory` and backing mount (without user file recursion);
  - MinIO service storage path and SSD vs HDD co-location (with strict credential redaction);
  - Docker Root Dir mapping and physical vs logical accounting reconciliation;
  - Docker unreferenced images, stopped containers, build cache, and volume references categorized into Risk Tiers A, B, and C;
  - Bounded candidate root scans and targeted large-file discovery;
  - Old disabled Snap revisions inventory and footprint estimation;
  - Journal disk usage and effective drop-in retention settings.
- [ ] Reusable workflow outputs or formatted step summaries produce the complete Cleanup Readiness Matrix.
- [ ] Safety constraints verified: zero pruning, zero deletion, zero journal vacuuming, zero service restart, zero configuration modification.
- [ ] Workflow passes local YAML syntax and schema validation.
- [ ] Dispatch authorization boundary preserved: no workflow dispatched without separate human approval.

## Verification requirements

### Required checks
- Local YAML and syntax validation of `.github/workflows/server-debug.yml` and `.github/workflows/server-storage-cleanup-readiness.yml`.
- Verification of branch gating logic (`github.ref_name == 'task/server-storage-cleanup-readiness'`).
- Git diff check ensuring no credentials, mutations, or unrelated workflows were touched.
- Clean git status on `task/server-storage-cleanup-readiness`.

### Required evidence
- Branch name: `task/server-storage-cleanup-readiness`.
- Immutable governing task commit SHA.
- Validation logs demonstrating successful static checks.
- Confirmation of zero workflow dispatch, zero production access, and zero cleanup actions.

## Stop conditions

The Executor MUST stop implementation and return the issue to planning when:
- Any proposed diagnostic probe requires write access, container restart, service restart, or configuration mutation.
- A probe cannot determine storage paths without leaking credentials, passwords, or secret tokens.
- Secret passing or secret inheritance is requested or required.
- The Executor is requested to perform actual pruning, deletion, or cleanup.
- A requirement requires direct workflow dispatch before human approval.

## Side-effect authorization

### Explicitly authorized side effects
- Editing `.agents/tasks/plan-server-storage-cleanup.md`.
- Following task contract acceptance, editing `.github/workflows/server-debug.yml` and creating `.github/workflows/server-storage-cleanup-readiness.yml`.
- Committing and pushing changes to `origin/task/server-storage-cleanup-readiness`.

### Explicitly prohibited side effects
- Workflow execution or dispatch without separate human approval.
- Production server access or mutation.
- Merging to `main` or modifying the default branch.
- Any file deletion, container pruning, volume removal, or journal truncation.

## Expected terminal outcome

### Review Required
- When this task document is authored, committed, and pushed to `task/server-storage-cleanup-readiness` for Reviewer approval prior to workflow implementation.
- Following workflow implementation, for Reviewer approval prior to any production dispatch.

### Planning Required
- If new constraints or production access boundaries require redefining the scope or authorization model.
