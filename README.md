# Madeena Admin Infra

Public repository for Madeena server administration and DevOps automation.

The repository contains GitHub Actions workflows for provisioning, deployment,
diagnostics, maintenance, and service operations. Secrets must be configured in
GitHub Actions and must never be committed here.

## Scope

- `.github/workflows/` — server and infrastructure workflows
- `.agents/` — repository delivery contract and agent guidance
- `AGENTS.md` — Codex runtime adapter

Production workflows require the appropriate GitHub environment, repository
secrets, runner permissions, and separate operational authorization.
