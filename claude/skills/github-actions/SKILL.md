---
name: github-actions
description: >
  This skill should be used when the user asks about CI/CD failures, workflow logs, Actions status,
  pipeline issues, or needs to troubleshoot failed builds. Triggers on "GitHub Actions", "workflow",
  "CI failed", "build failed", "check the logs", or references to a specific workflow run or commit.
model: haiku
allowed-tools:
  - Bash(gh *)
  - Bash(git *)
  - Read
  - Grep
---

# GitHub Actions Troubleshooting

To analyze and troubleshoot GitHub Actions workflows in the current repository.

## Prerequisites

To verify `gh` is available and authenticated:

```bash
gh auth status
gh api user --jq '.login'
```

If not authenticated: `gh auth login`. If wrong account: `gh auth switch`.

## Step 1: Detect GitHub Repository

To check if in a GitHub repo:

```bash
git remote get-url origin 2>/dev/null | grep -q github.com
```

To extract owner/repo:

```bash
git remote get-url origin | sed -E 's#.*github\.com[:/]([^/]+)/([^.]+)(\.git)?#\1/\2#'
```

## Step 2: Find Workflow Runs

**By current commit:**
```bash
COMMIT_SHA=$(git rev-parse HEAD)
gh run list --commit $COMMIT_SHA --json databaseId,status,conclusion,workflowName,headBranch,createdAt --limit 10
```

**By branch:**
```bash
gh run list --branch $(git branch --show-current) --limit 5 --json databaseId,status,conclusion,workflowName,createdAt
```

**Recent failures:**
```bash
gh run list --status failure --json databaseId,status,conclusion,workflowName,headBranch,createdAt --limit 5
```

## Step 3: View Run Details

```bash
gh run view <run-id> --verbose
```

## Step 4: Analyze Logs

**Failed steps only (start here):**
```bash
gh run view <run-id> --log-failed
```

**Full logs:**
```bash
gh run view <run-id> --log
```

**Specific job:**
```bash
gh run view <run-id> --json jobs --jq '.jobs[] | {id: .databaseId, name: .name, conclusion: .conclusion}'
gh run view <run-id> --job <job-id> --log
```

## GHCR / Docker Build Failures

For this repo's CI (push to main → GHCR image build), the relevant step is typically named **"Build and push Docker image"**. Use `--log-failed` first — Docker build errors appear inline in that step's log.

```bash
# Quick pattern for a failed push to main
COMMIT_SHA=$(git rev-parse HEAD)
RUN_ID=$(gh run list --commit $COMMIT_SHA --json databaseId,conclusion --jq 'first(.[] | select(.conclusion == "failure")) | .databaseId')
gh run view $RUN_ID --log-failed
```

## Output Format

Always provide:
1. **Context** — which workflow/job failed, on which commit
2. **Key errors** — extract the relevant error lines from logs
3. **Next steps** — what to fix or investigate
