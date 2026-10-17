github notes:
import pypandoc

content = r"""# Git & GitHub Commands for DevOps Engineers
## Practical Notes, Real-Time Syntax, and Common Operations

> **Purpose:** A practical command reference for maintaining application code, Terraform, Kubernetes manifests, CI/CD pipelines, and infrastructure scripts in GitHub repositories.
>
> **Important:** Git commands run locally unless noted. Commands using `git push`, `git pull`, or `gh` interact with a remote service. Replace sample URLs, branch names, repository names, and file paths with your own values.

---

## 1. Git and GitHub: Quick Difference

- **Git** is a distributed version-control system used to track changes in files.
- **GitHub** hosts Git repositories and adds collaboration features such as pull requests, code reviews, issues, Actions, and branch protection.
- **GitHub CLI (`gh`)** is an optional command-line tool for managing GitHub pull requests, issues, and repositories.

### Configure Git for the first time

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main

# Verify configuration
git config --global --list
git --version
```

Use the email address associated with your GitHub account if you want commits linked to that account.

---

## 2. Create or Download a Repository

### Clone an existing GitHub repository

```bash
git clone https://github.com/ORG/REPO.git
cd REPO
```

Clone into a custom directory:

```bash
git clone https://github.com/ORG/REPO.git payment-service
cd payment-service
```

Clone a specific branch:

```bash
git clone --branch develop --single-branch https://github.com/ORG/REPO.git
```

### Initialize a local project and connect it to GitHub

```bash
mkdir devops-demo
cd devops-demo
git init
git branch -M main

echo "# DevOps Demo" > README.md
git add README.md
git commit -m "docs: add project README"

git remote add origin https://github.com/ORG/REPO.git
git push -u origin main
```

Create the GitHub repository first, or use GitHub CLI if installed and authenticated:

```bash
gh auth login
gh repo create devops-demo --private --source=. --remote=origin --push
```

### Check repository remotes

```bash
git remote -v
git remote show origin
git remote add upstream https://github.com/ORIGINAL-ORG/REPO.git
git remote set-url origin https://github.com/ORG/REPO.git
```

`origin` commonly points to your fork or main remote; `upstream` commonly points to the original repository in a fork workflow.

---

## 3. The Daily Git Workflow

Typical workflow for a code or infrastructure change:

```bash
git status
git switch -c feature/add-health-check
# Edit files
git diff
git add deployment.yaml
git diff --cached
git commit -m "feat: add application health check"
git push -u origin feature/add-health-check
# Open a pull request on GitHub and request review
```

The three important areas:

1. **Working tree** — files you have changed.
2. **Staging area** — changes selected for the next commit using `git add`.
3. **Repository history** — saved snapshots created by `git commit`.

---

## 4. Core Git Commands

### `git status` — inspect the current state

```bash
git status
git status --short
```

Shows the current branch, modified files, staged changes, and untracked files.

### `git add` — stage changes

```bash
git add app.py                 # Stage one file
git add deployment.yaml        # Stage one Kubernetes manifest
git add .                      # Stage changes under the current directory
git add -A                     # Stage additions, modifications, and deletions
git add '*.tf'                 # Stage matching Terraform files (shell behavior can vary)
git add -p                     # Interactively stage selected chunks
```

**Best practice:** Review `git status` before `git add .`; avoid staging secrets, generated files, logs, or large artifacts.

### `git commit` — save a snapshot

```bash
git commit -m "fix: correct readiness probe"
git commit -am "fix: update tracked configuration"
git commit --amend -m "fix: correct readiness probe"
```

- `-m` supplies the commit message.
- `-a` stages modifications and deletions of already tracked files, but **does not include new untracked files**.
- `--amend` replaces the most recent commit. Avoid amending a commit already shared with others unless the team agrees and understands the history rewrite.

### `git diff` — review changes

```bash
git diff                  # Unstaged changes
git diff --staged         # Staged changes
git diff main...HEAD      # Changes on current branch since it diverged from main
git diff --stat           # Summary of changed files
git diff --check          # Check whitespace errors
```

### `git log` — inspect history

```bash
git log
git log --oneline --graph --decorate --all
git log -5
git log -- app.py
git show <commit-hash>
git blame app.py
```

`git blame` shows which commit last changed each line; it is an investigation aid, not proof of who is responsible for a defect.

### `git push` — publish commits

```bash
git push
git push -u origin feature/add-health-check
git push origin main
git push --force-with-lease origin feature/my-branch
```

Use `--force-with-lease` only when a history rewrite is necessary and coordinated. Do not force-push shared protected branches such as `main` or `production`.

### `git pull` and `git fetch` — get remote changes

```bash
git fetch origin
git fetch --all --prune
git pull
git pull --rebase origin main
```

- `fetch` downloads remote information without merging it into your current branch.
- `pull` fetches and integrates remote changes.
- `pull --rebase` replays local commits on top of the updated branch. Follow your team's merge/rebase policy.

---

## 5. Branch Management

```bash
git branch                         # List local branches
git branch -a                      # List local and remote-tracking branches
git switch main                    # Switch branches
git switch -c feature/new-api      # Create and switch to a branch
git branch -d feature/new-api      # Delete a merged local branch
git branch -D feature/new-api      # Force-delete a local branch; use cautiously
git push origin --delete feature/new-api  # Delete remote branch
git switch -                       # Return to previous branch
```

Older syntax you may see:

```bash
git checkout main
git checkout -b feature/new-api
```

Prefer `git switch` for switching/creating branches and `git restore` for restoring files because their intent is clearer.

### Example branch naming

```text
feature/add-cloud-monitoring
bugfix/fix-health-check
hotfix/production-timeout
chore/update-terraform-provider
docs/add-deployment-runbook
```

---

## 6. `git reset` vs `git revert` vs `git restore`

These commands are frequently confused. Choose based on whether you want to change local history, create a new undo commit, or discard file changes.

### `git reset` — move the current branch pointer

```bash
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1
```

- `--soft`: undo the latest commit but keep its changes staged.
- Default / `--mixed`: undo the commit and keep changes in the working tree, unstaged.
- `--hard`: reset the commit, staging area, and working tree. **Uncommitted changes can be lost.**

Reset a file from the staging area without resetting commits:

```bash
git reset HEAD -- deployment.yaml
```

This unstages the file while keeping its working-tree changes.

### `git revert` — create a new commit that undoes another commit

```bash
git log --oneline
git revert <commit-hash>
git push origin main
```

For a shared branch or production fix, `git revert` is generally safer than rewriting published history because it preserves the existing commits and records the undo as a new commit.

Revert a merge commit only after determining the correct mainline parent:

```bash
git revert -m 1 <merge-commit-hash>
```

### `git restore` — discard or unstage file changes

```bash
git restore deployment.yaml
git restore --staged deployment.yaml
git restore --source=HEAD~1 deployment.yaml
```

- First command discards unstaged edits in that file.
- Second command unstages the file but keeps its edits.
- Third command restores the file's content from an earlier commit into the working tree.

**Safety rule:** Before destructive commands, run `git status` and consider making a backup or temporary branch.

---

## 7. Merge, Rebase, and Conflict Resolution

### Merge a feature branch

```bash
git switch main
git pull --ff-only origin main
git merge feature/add-health-check
git push origin main
```

In a pull-request workflow, normally open and review a PR rather than pushing directly to `main`.

### Rebase a feature branch

```bash
git fetch origin
git switch feature/add-health-check
git rebase origin/main
```

If conflicts occur:

```bash
git status
# Edit each conflicted file and remove conflict markers
git add <resolved-file>
git rebase --continue
```

Abort if necessary:

```bash
git rebase --abort
```

For a merge conflict:

```bash
git status
# Edit conflicted files
git add <resolved-file>
git commit
```

To cancel an in-progress merge before committing:

```bash
git merge --abort
```

Do not blindly choose `--ours` or `--theirs` in production configuration; verify which version contains the intended infrastructure or application settings.

---

## 8. Undoing Common Mistakes

### Accidentally staged the wrong file

```bash
git restore --staged secret.env
```

This unstages the file; it does not remove it from your disk.

### Discard local edits to a tracked file

```bash
git restore app.py
```

This discards unstaged changes in `app.py`. Check `git diff` first.

### Recover a commit after reset

```bash
git reflog
git switch -c recovery/restore-work <commit-hash>
```

`reflog` can help locate recent local branch positions and commits. Recovery is not guaranteed forever, so act promptly.

### Remove a file from Git tracking but keep it locally

```bash
git rm --cached .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: stop tracking local environment file"
```

If a secret was committed or pushed, removing the file is not enough. **Revoke/rotate the secret immediately**, then follow your organization's process for removing sensitive data from history.

### Revert a production change

```bash
git log --oneline
git revert <bad-commit-hash>
git push origin <release-branch>
```

Use the repository's approved PR, review, testing, and release process.

---

## 9. Tags and Releases

```bash
git tag
git tag -a v1.2.0 -m "Release v1.2.0"
git show v1.2.0
git push origin v1.2.0
git push origin --tags
git tag -d v1.2.0
git push origin --delete v1.2.0
```

Use annotated tags for releases. Confirm the release commit and approval before tagging production.

Example version pattern:

```text
v1.0.0  Initial release
v1.1.0  Backward-compatible feature
v1.1.1  Bug fix
v2.0.0  Breaking change
```

---

## 10. `.gitignore` for DevOps Repositories

Create a `.gitignore` file to exclude generated and local files.

Example:

```gitignore
# Environment files and secrets
.env
.env.*
!.env.example
*.pem
*.key
credentials.json

# Terraform
.terraform/
*.tfstate
*.tfstate.*
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
.terraform.lock.hcl
# Note: Many teams intentionally commit .terraform.lock.hcl
# to pin provider selections; remove the ignore rule if so.

# Kubernetes / local artifacts
*.log
tmp/
.DS_Store
.vscode/
.idea/
```

**Important Terraform note:** Teams commonly commit `.terraform.lock.hcl` so provider versions remain reproducible. Do not ignore it by default if your team's workflow expects it in version control. Never commit state files that may contain secrets. Store remote state in a secured backend with appropriate access controls.

A `.gitignore` rule does not untrack a file already committed. Use `git rm --cached <file>` to stop tracking it.

---

## 11. Pull Requests and GitHub CLI (`gh`)

Install GitHub CLI and authenticate first:

```bash
gh auth login
gh auth status
```

Create a pull request from the current branch:

```bash
gh pr create --base main --head feature/add-health-check \
  --title "Add application health check" \
  --body "Adds readiness and liveness probe configuration."
```

Other useful commands:

```bash
gh repo view
gh pr list
gh pr view 42
gh pr checkout 42
gh pr diff 42
gh pr checks 42
gh pr review 42 --approve
gh pr review 42 --request-changes --body "Please add a rollback note."
gh issue list
gh issue create --title "Investigate deployment failure" --body "Review failed rollout logs."
gh run list
gh run view <run-id>
gh run watch <run-id>
gh run rerun <run-id>
```

Approval and merge permissions depend on repository rules and your GitHub role. Never approve your own change if your organization's policy prohibits it.

---

## 12. GitHub Actions: Common DevOps Operations

GitHub Actions workflows are usually stored under `.github/workflows/`.

Example structure:

```text
repo/
├── app/
├── terraform/
├── k8s/
├── scripts/
├── tests/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .gitignore
└── README.md
```

Typical CI pipeline stages:

1. Checkout source.
2. Set up the required runtime/toolchain.
3. Install dependencies.
4. Run linting and tests.
5. Scan code/dependencies and build artifacts.
6. Publish artifacts or container images after required checks.
7. Deploy through an approved environment with protected secrets and approvals.

Example minimal workflow (`.github/workflows/ci.yml`):

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Show repository files
        run: ls -la

      - name: Placeholder validation
        run: echo "Add project-specific linting and tests here"
```

Use organization-approved action versions and pin third-party actions to a full commit SHA when required by policy. Do not place credentials directly in workflow YAML. Prefer short-lived identity federation/OIDC where supported, and use GitHub Secrets only when appropriate.

---

## 13. DevOps Real-Time Scenarios

### Scenario A: Update a Kubernetes manifest

```bash
git switch -c feature/update-replicas
# Edit k8s/deployment.yaml
git diff -- k8s/deployment.yaml
git add k8s/deployment.yaml
git diff --cached
git commit -m "feat: scale payment deployment"
git push -u origin feature/update-replicas
```

Then open a PR, run validation, get approval, and deploy using the approved CD pipeline. Do not make an unreviewed production change simply because the Git push succeeded.

### Scenario B: Terraform module change

```bash
git switch -c feature/update-vpc-module
# Edit Terraform files
terraform fmt -recursive
terraform validate
git diff
git add terraform/
git diff --cached
git commit -m "feat: update reusable VPC module"
git push -u origin feature/update-vpc-module
```

Run `terraform plan` using the team's configured backend and environment. Review the plan in CI and require approval before applying production infrastructure. Do not commit `.tfstate` files or credentials.

### Scenario C: CI pipeline failed after a commit

```bash
git status
git log -5 --oneline
gh run list
gh run view <run-id>
```

Inspect logs and identify whether the failure comes from code, tests, dependencies, permissions, secrets, runner capacity, or deployment configuration. Fix forward with a reviewed commit, or use the approved rollback/revert process.

### Scenario D: A bad commit reached a shared branch

```bash
git log --oneline
git revert <bad-commit-hash>
git push origin <shared-branch>
```

Run tests and the release pipeline. For an urgent incident, follow the team's incident-management and change-control procedures.

### Scenario E: Resolve a rejected push

```bash
git fetch origin
git status
git pull --rebase origin <branch>
# Resolve conflicts if any
git add <resolved-file>
git rebase --continue
git push
```

If the branch is protected, open a pull request instead of trying to push directly.

### Scenario F: Find who changed a configuration line

```bash
git blame k8s/deployment.yaml
git log --oneline -- k8s/deployment.yaml
git show <commit-hash>
```

Use the commit and PR context to understand why the change was made before modifying it again.

---

## 14. GitHub Repository Best Practices for DevOps

- Protect `main` and release branches; require PR reviews and successful CI checks.
- Use short-lived feature branches and meaningful commit messages.
- Never commit passwords, API tokens, private keys, kubeconfigs, service-account keys, or unencrypted Terraform state.
- Use least-privilege permissions and short-lived credentials where possible.
- Require linting, unit tests, security scans, and infrastructure plan review appropriate to the project.
- Keep reusable Terraform modules, Kubernetes manifests, scripts, and runbooks organized.
- Pin dependencies and tools to approved versions; update them through reviewed changes.
- Keep production deployment approvals and environment protections enabled.
- Use signed commits or verified commits if your organization requires them.
- Document rollback steps and test them.
- Treat Git history as an audit trail, but do not assume Git alone satisfies every compliance requirement.

### Suggested commit-message format

```text
type: short description

Examples:
feat: add Cloud Monitoring alert policy
fix: correct GKE readiness probe
docs: update deployment runbook
chore: upgrade Terraform provider
test: add API health-check tests
refactor: simplify CI validation stage
```

---

## 15. Quick Command Cheat Sheet

| Task | Command |
|---|---|
| Clone repo | `git clone <url>` |
| Check status | `git status` |
| Stage one file | `git add <file>` |
| Stage all changes | `git add -A` |
| Commit | `git commit -m "message"` |
| Push current branch | `git push` |
| Set upstream and push | `git push -u origin <branch>` |
| Fetch remote updates | `git fetch origin` |
| Pull changes | `git pull` |
| Create branch | `git switch -c <branch>` |
| Switch branch | `git switch <branch>` |
| List branches | `git branch -a` |
| Compare changes | `git diff` |
| Review staged changes | `git diff --staged` |
| View history | `git log --oneline --graph --all` |
| Unstage a file | `git restore --staged <file>` |
| Discard local file edits | `git restore <file>` |
| Undo latest local commit, keep staged | `git reset --soft HEAD~1` |
| Undo latest commit, keep edits unstaged | `git reset HEAD~1` |
| Undo a published commit safely | `git revert <commit>` |
| Inspect recovery history | `git reflog` |
| Add release tag | `git tag -a v1.0.0 -m "Release v1.0.0"` |
| Push tag | `git push origin v1.0.0` |
| Create PR with GitHub CLI | `gh pr create` |
| View CI runs | `gh run list` |

---

## 16. A Safe Daily Checklist

```bash
git status
git switch -c feature/<short-description>
# Make and test your changes
git diff
git add <specific-files>
git diff --staged
git commit -m "type: clear description"
git push -u origin feature/<short-description>
# Create PR, wait for required checks/review, then merge using team policy
```

Before any reset, restore, force push, or history rewrite:

1. Run `git status`.
2. Understand whether the change is local or already shared.
3. Back up important work if needed.
4. Choose `revert` for a shared commit in most cases.
5. Follow repository branch protections and incident/change-management policies.

---

## 17. Official Documentation

- Git reference: https://git-scm.com/docs
- Git Book: https://git-scm.com/book/en/v2
- GitHub documentation: https://docs.github.com/
- GitHub CLI manual: https://cli.github.com/manual/
- GitHub Actions documentation: https://docs.github.com/actions
- Terraform CLI documentation: https://developer.hashicorp.com/terraform/cli
"""

output_path = "/mnt/data/notes.md"
pypandoc.convert_text(content, "md", format="md", outputfile=output_path, extra_args=["--standalone"])
print(f"Created {output_path}")
