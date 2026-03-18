---
name: openspec-gh-merge
description: Merge an approved GitHub Pull Request from an OpenSpec change. Validates approval, blocks explore/ direct merge, selects merge strategy by branch type, cleans up, and prompts to archive.
license: MIT
compatibility: Requires git CLI and gh CLI (GitHub CLI).
metadata:
  author: custom
  version: "1.1"
---

Merge an approved Pull Request and clean up the local branch.
This is the final step in the GitHub integration workflow after a PR has been reviewed.

**Prerequisites**:
- `gh` CLI must be installed and authenticated (`gh auth login`)
- A PR must already exist for the current branch (created via `/opsx-gh-pr`)

**Input**: Optionally specify a change name or PR number. If omitted, infer from the current branch name.

---

## Steps

### Step 1: Identify the PR

If a PR number is provided, use it. Otherwise:
```bash
gh pr view --json number,title,state,headRefName,baseRefName,isDraft,reviews,statusCheckRollup,mergeable
```
- Extract branch name and parse change name
- If no PR exists → error, suggest running `/opsx-gh-pr` first

Announce: "Merging PR for change: <name>"

---

### Step 2: Block explore/ direct merge

**If the branch starts with `explore/`:**

Display:
```
🚫 explore/ 分支不允許直接 merge 到 main。

explore/ 分支用於快速驗證，必須先走升格流程再合併。
請回到 /opsx-gh-apply 選擇以下其中一個出口：
  B. 轉為正式分支（建 feat/ 或 fix/ 後走完整流程）
  D. 升格為正式功能（cherry-pick 後重新走 PR 流程）
```

**Stop here. Do NOT proceed with merge.**

The only exception: if the user explicitly types `--force-explore-merge`, show a final confirmation:
> "⚠️ 這會直接將 explore/ 分支 merge 到 main，跳過正式升格流程。確定嗎？（輸入 YES 確認）"
- User types `YES` → proceed with squash merge (treat as explore exception)
- Anything else → abort

---

### Step 3: Validate merge readiness

Check conditions based on branch type:

**Conflict check (all branches):**
- If `mergeable: CONFLICTING` → stop immediately:
  > "❌ 有 merge 衝突，無法繼續。請先解決衝突後再執行 /opsx-gh-merge。"
  - Show list of conflicting files if available

**Approval requirement:**

| Branch | Required |
|--------|----------|
| `feat/` | ≥ 1 approved review |
| `fix/` | ≥ 1 approved review |
| `refactor/` | ≥ 1 approved review |
| `hotfix/` | CI pass **OR** ≥ 1 approval (either sufficient) |

**CI check (if CI is configured):**
- Failing CI → warn and ask if user wants to proceed anyway
- No CI configured → skip this check

**If validation fails for `feat/fix/refactor`:**
- Show clear summary of what's missing
- Do NOT proceed without user confirmation
- Example: "⚠️ 此 PR 尚未通過 code review（0 approvals）。建議請 team member review 後再 merge。"

**For `hotfix/` with neither CI pass nor approval:**
- Show warning
- Ask: "此 hotfix 沒有 approval 也沒有通過 CI，確定要 merge 嗎？"
- Require explicit confirmation before proceeding

---

### Step 4: Determine merge strategy

| Branch prefix | Strategy | Reason |
|--------------|---------|--------|
| `feat/` | Squash merge | Clean main history |
| `fix/` | Squash merge | Clean history |
| `hotfix/` | Merge commit | Preserve full emergency fix context |
| `refactor/` | Squash merge | Clean history |
| `explore/` (exception only) | Squash merge | Exploratory work squashed |

Display the strategy before executing:
```
Merge strategy: Squash merge
feat/add-user-auth → main
```

---

### Step 5: Execute the merge

Squash merge (`feat/`, `fix/`, `refactor/`, `explore/` exception):
```bash
gh pr merge --squash --delete-branch
```

Merge commit (`hotfix/`):
```bash
gh pr merge --merge --delete-branch
```

`--delete-branch` removes the remote branch after merge.

---

### Step 6: Clean up local branch

```bash
git checkout main
git pull
git branch -d <branch-name>
```

If `git branch -d` fails (unmerged commits warning):
- Show the warning
- Ask: "本地分支有未合併的 commits，確定要強制刪除嗎？"
- Only use `git branch -D` with explicit user confirmation

---

### Step 7: Hotfix — offer to create a tag

After a successful `hotfix/` merge, ask:
> "要為這個 hotfix 建立 Git tag 嗎？（例如：v1.2.1）"
- Yes → ask for tag name, then:
  ```bash
  git tag <tag-name>
  git push origin <tag-name>
  ```
- No → skip

---

### Step 8: Show completion summary

Display:
- PR title and URL
- Merge type (squash / merge commit)
- Remote branch deleted
- Local branch deleted
- Tag created (if applicable, hotfix only)

Suggest next steps:
- → Run `/opsx-archive <change-name>` to archive this change locally

---

## Output On Success

```
## PR Merged ✓

**Change:** <change-name>
**PR:** <URL>
**Branch:** feat/<name> → main (deleted)
**Strategy:** Squash merge

### Local cleanup
- ✓ Switched to main
- ✓ Pulled latest
- ✓ Deleted local branch feat/<name>

→ Run `/opsx-archive <change-name>` to archive this change.
```

## Output for Hotfix

```
## Hotfix Merged ✓

**Change:** <change-name>
**PR:** <URL>
**Branch:** hotfix/<name> → main (deleted)
**Strategy:** Merge commit (full context preserved)

Tag created: v1.2.1 → pushed to origin

→ Run `/opsx-archive <change-name>` to archive this change.
```

---

## Guardrails

- **explore/ branches are blocked from direct merge** — must go through promotion flow first
- The only exception to explore/ block requires `--force-explore-merge` flag AND explicit `YES` confirmation
- Never force-push or force-merge into main/master
- `feat/fix/refactor` require at least 1 approved review
- `hotfix/` allows merge with CI pass OR 1 approval; warn if neither
- If `mergeable: CONFLICTING`, stop and show conflict files — do not proceed
- Only use `git branch -D` (force delete) with explicit user confirmation
- Always pull main after merge to sync local state
