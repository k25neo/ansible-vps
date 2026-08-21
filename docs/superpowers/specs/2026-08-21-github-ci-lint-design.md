# GitHub CI: Ansible lint + syntax-check

Date: 2026-08-21  
Status: approved design  
Scope: CI only (no CD)

## Goal

On every push and pull request to any branch, fail the GitHub Actions job if Ansible playbooks fail static checks. No deploy to VPS, no SSH, no live `--check` against hosts.

## Triggers

- `push` — all branches
- `pull_request` — all branches

## Workflow

File: `.github/workflows/ci.yml`

- Runner: `ubuntu-latest`
- Single job: `lint`
- No matrix, no secrets, no artifacts

### Steps

1. `actions/checkout@v4` (or current stable)
2. Run `ansible/ansible-lint` (official action, current stable major, e.g. `@v25`) against the repository
3. Ensure Ansible CLI is available (via the lint action environment or an explicit install if needed)
4. Run `ansible-playbook --syntax-check` for each playbook, using inventory that is not gitignored:

   - `playbook-debian-12-to-13.yml`
   - `playbook-openconnect.yml`
   - `playbook-radius.yml`
   - `playbooks/playbook-telegram.yml`

Inventory for syntax-check: `-i hosts.example` because `hosts` is in `.gitignore` and must not be required in CI.

## Failure policy

- Any ansible-lint failure fails the job
- Any playbook syntax-check failure fails the job
- No soft-fail / continue-on-error

## Out of scope

- Continuous deployment / running playbooks on VPS
- Building `docker/Dockerfile` in CI
- Adding `.ansible-lint` skip rules unless noise appears after first runs
- Caching beyond what the official action provides by default

## Rationale

Official `ansible/ansible-lint` action keeps the workflow small and maintained. Syntax-check covers parse/load errors that lint alone may miss. Using `hosts.example` keeps CI secret-free while matching the documented inventory shape.
