# GitHub CI Ansible Lint Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a GitHub Actions workflow that runs ansible-lint and `ansible-playbook --syntax-check` on every push and pull request.

**Architecture:** One workflow file `.github/workflows/ci.yml` with a single `lint` job on `ubuntu-latest`. Official `ansible/ansible-lint` action performs lint; a follow-up shell step runs syntax-check against all four playbooks using `hosts.example` (because `hosts` is gitignored).

**Tech Stack:** GitHub Actions, `actions/checkout@v4`, `ansible/ansible-lint@v26.8.0`, Ansible CLI (`ansible-playbook`)

## Global Constraints

- CI only — no deploy, no SSH, no live host `--check`
- Triggers: `push` and `pull_request` on all branches
- Inventory for syntax-check must be `-i hosts.example`
- No `continue-on-error`; lint or syntax failure fails the job
- No `.ansible-lint` config unless a later follow-up is needed for noise
- No Docker image build in CI
- Do not commit `.idea/` or real `hosts`

## File Structure

| File | Responsibility |
|------|----------------|
| `.github/workflows/ci.yml` | Define triggers, job, lint + syntax-check steps |
| `docs/superpowers/specs/2026-08-21-github-ci-lint-design.md` | Approved design (read-only reference) |
| `hosts.example` | Inventory used by CI syntax-check (already exists) |
| Playbooks listed below | Lint/syntax targets (already exist) |

Playbooks to syntax-check:

- `playbook-debian-12-to-13.yml`
- `playbook-openconnect.yml`
- `playbook-radius.yml`
- `playbooks/playbook-telegram.yml`

---

### Task 1: Add CI workflow

**Files:**
- Create: `.github/workflows/ci.yml`
- Test: validate YAML locally; after push, GitHub Actions run for the branch

**Interfaces:**
- Consumes: existing playbooks + `hosts.example`
- Produces: workflow named `CI` with job `lint` that exits non-zero on lint/syntax failure

- [ ] **Step 1: Create the workflow file**

Create `.github/workflows/ci.yml` with exactly this content:

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  lint:
    name: Ansible Lint
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run ansible-lint
        uses: ansible/ansible-lint@v26.8.0

      - name: Syntax-check playbooks
        run: |
          set -euo pipefail
          command -v ansible-playbook
          for playbook in \
            playbook-debian-12-to-13.yml \
            playbook-openconnect.yml \
            playbook-radius.yml \
            playbooks/playbook-telegram.yml
          do
            echo "==> syntax-check $playbook"
            ansible-playbook -i hosts.example --syntax-check "$playbook"
          done
```

Notes for the implementer:

- `ansible/ansible-lint@v26.8.0` installs `ansible-lint` and exposes `ansible-core` executables (including `ansible-playbook`) on the job PATH via `uv tool install`.
- If Step 3 locally or on GitHub shows `ansible-playbook: command not found`, insert this step **between** lint and syntax-check (do not change triggers or inventory):

```yaml
      - name: Ensure ansible-playbook is available
        run: |
          if ! command -v ansible-playbook >/dev/null 2>&1; then
            pip install --user "ansible-core>=2.16,<2.21"
            echo "$HOME/.local/bin" >> "$GITHUB_PATH"
          fi
          command -v ansible-playbook
```

- [ ] **Step 2: Validate workflow YAML parses**

Run:

```bash
python3 -c "import pathlib, yaml; yaml.safe_load(pathlib.Path('.github/workflows/ci.yml').read_text()); print('ok')"
```

Expected: `ok`

If `yaml` is missing:

```bash
python3 -c "import pathlib; import json; from pathlib import Path; p=Path('.github/workflows/ci.yml'); assert p.exists() and p.stat().st_size>0; print('ok-exists')"
```

Expected: `ok-exists` (fallback only; prefer PyYAML when available)

- [ ] **Step 3: Dry-run checks locally in the project Docker image (optional but preferred)**

From repo root:

```bash
docker compose run --rm ansible sh -c 'ansible-playbook -i hosts.example --syntax-check playbook-debian-12-to-13.yml && ansible-playbook -i hosts.example --syntax-check playbook-openconnect.yml && ansible-playbook -i hosts.example --syntax-check playbook-radius.yml && ansible-playbook -i hosts.example --syntax-check playbooks/playbook-telegram.yml'
```

Expected: each playbook prints a play recap / syntax OK and exit code `0`.

If Docker is unavailable, skip this step and rely on GitHub Actions after push.

- [ ] **Step 4: Commit the workflow**

```bash
git add .github/workflows/ci.yml
git commit -m "$(cat <<'EOF'
Add GitHub Actions CI for ansible-lint and syntax-check.

EOF
)"
```

- [ ] **Step 5: Push and confirm the Actions run**

```bash
git push -u origin HEAD
```

Then open the branch Actions tab (or `gh run list --limit 5` / `gh run watch`) and confirm job `Ansible Lint` completes.

Expected success: green check, both ansible-lint and syntax-check steps passed.

If ansible-lint fails with rule violations in existing playbooks: fix the playbook issues (preferred) or, only if the team agrees noise is unreasonable, add a minimal `.ansible-lint` skip list in a **separate follow-up commit** — not part of the happy-path first commit unless CI is blocked.

If `ansible-playbook` is missing on the runner: apply the fallback step from Step 1 notes, commit, and push again.

---

## Spec coverage (self-review)

| Spec requirement | Task |
|------------------|------|
| Workflow at `.github/workflows/ci.yml` | Task 1 |
| `push` + `pull_request` all branches | Task 1 workflow `on:` |
| `ubuntu-latest`, single `lint` job | Task 1 |
| `actions/checkout` | Task 1 |
| Official `ansible/ansible-lint` | Task 1 `@v26.8.0` |
| Syntax-check all four playbooks | Task 1 loop |
| `-i hosts.example` | Task 1 |
| Fail on lint/syntax errors | default job behavior, no continue-on-error |
| No CD / Docker build / secrets | omitted from workflow |

No placeholders remain. Version pins are concrete (`checkout@v4`, `ansible-lint@v26.8.0`).
