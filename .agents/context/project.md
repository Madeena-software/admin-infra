---
title: Madeena Admin Infra Repository Context
document_id: AGENT-CONTEXT-ADMIN-INFRA-001
version: 1.0
status: approved-template
language: en-US
scope:
  - repository-level AI orientation
  - GitHub Actions infrastructure automation
authority_note: This context is supporting repository knowledge. Approved repository authority and observed implementation evidence remain controlling.
---

# Repository Context

## Repository identity

**Name:** `admin-infra`

**Repository type:** Infrastructure automation repository

**Primary responsibility:** Maintain GitHub Actions workflows used to administer
Madeena servers and related infrastructure.

## Purpose

This repository separates server administration and DevOps automation from the
Simama Laravel application. It contains workflow definitions and the repository
delivery contract required to maintain them safely.

## Current repository state

Initial extraction from the Simama repository. The workflows are operational
automation and may affect production systems; changes require review and the
required operational authorization before dispatch.

## Intended authority map

- Repository AI delivery contract: `.agents/AGENTS.md`
- Delivery protocol: `.agents/software-workflow.md`
- Planning and review procedure: `.agents/prompts/plan-create-task.md`
- Validated tasks: `.agents/tasks/`
- Operational authorization: the applicable human decision, change record, or
  approved operational procedure

## Observed implementation evidence map

- Workflow implementation: `.github/workflows/`
- Version control and review evidence: Git history and GitHub pull requests
- Runtime evidence: GitHub Actions run history and server-side verification

## Top-level architecture and boundaries

- GitHub Actions is the execution control plane.
- Workflows use configured GitHub secrets and runner permissions at runtime.
- Server-side commands execute through the configured self-hosted runner or SSH
  boundary defined by each workflow.
- This repository does not contain the Laravel application source code.
- Secrets, private keys, passwords, and generated credentials are out of scope
  for version control.

## Delivery state

### Current delivery objective

Maintain a public, reviewable source repository for Madeena infrastructure
automation.

### Quality-gate state

The initial extraction requires review of workflow scope, public-disclosure
risk, syntax, and operational authorization before production use.

### Active tasks

- [bootstrap-five-runner-pool.md](file:///var/www/admin-infra/.agents/tasks/bootstrap-five-runner-pool.md) — Bootstrap Five-Runner Organization Pool via Existing Self-Hosted Runner (Validated/Published)

### Blocking items

- Configure repository secrets, environments, runner access, and branch
  protections in GitHub before using state-changing workflows.
- Review public disclosure of infrastructure topology and operational commands.

## Accepted baseline

`4eea91539493308be4a9a680c4daab5f88fa09ce`

