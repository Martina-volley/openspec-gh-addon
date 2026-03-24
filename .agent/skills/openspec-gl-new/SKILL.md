---
name: openspec-gl-new
description: Start a new OpenSpec change with GitLab integration. Includes preflight checks, base branch confirmation, branch creation, and artifact requirement guidance by branch type.
license: MIT
compatibility: Requires openspec CLI, git CLI, and glab CLI (GitLab CLI).
metadata:
  author: custom
  version: "1.0"
  basedOn: openspec-gh-new
---

Start a new change with automatic Git branch creation and GitLab push.
This is a **non-destructive extension** of the standard `/opsx-new` workflow.

**Input**: The user's request should include a change name (kebab-case) OR a description of what they want to build.
Optional flags: `--explore`, `--feat`, `--fix`, `--hotfix`, `--refactor`

---

## Steps

### Step 1: Understand what to build

If no clear input provided, use the **AskUserQuestion tool**:
> "What change do you want to work on? Describe what you want to build or fix."

Derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).

**Do NOT proceed without understanding what the user wants to build.**

---

### Step 2: Preflight checks

**2a. Check git status**
```bash
git status
```
- If there are **uncommitted changes** → warn:
  > "⚠️ 有未提交的變更，建議先 `git stash` 或 `git commit` 後再建新分支，避免混入。"
  - Ask: "繼續建立新分支，還是先處理這些變更？"
  - If user chooses to continue → proceed with warning noted

**2b. Check for similar existing branches**
```bash
git branch -a | grep -E "feat/|fix/|hotfix/|refactor/|explore/"
```
- If a branch with the same or very similar name exists → show it and ask:
  > "找到相似的分支：`<branch-name>`。要繼續那個分支，還是建立新的？"

**2c. Check if it's a new or extension feature**
Use **AskUserQuestion**:
> "這個 change 是全新的專案功能，還是基於現有功能的延伸？"
- New feature → proceed normally
- Extension of existing → run `openspec list` to show related changes, suggest reviewing them first

---

### Step 3: Determine the branch type

If a flag was provided (`--explore`, `--feat`, `--fix`, `--hotfix`, `--refactor`), use it directly.

Otherwise, ask the user using the **AskUserQuestion tool**:

| 類型 | 前綴 | 說明 | TDD |
|-----|------|------|-----|
| Feature（新功能） | `feat/` | 正式功能開發 | 建議 |
| Explore（快速驗證） | `explore/` | 本地 spike，可能會丟棄 | 不強制 |
| Bug Fix（修 bug） | `fix/` | 修復已知問題 | 建議 |
| Hotfix（緊急修正） | `hotfix/` | Production 緊急修正 | 非必要 |
| Refactor（重構） | `refactor/` | 重構，不加新功能 | 建議 |

---

### Step 4: Confirm base branch

**Determine default base branch:**
```bash
git branch -a | grep -E "^(\*| ) (main|develop|master)$"
```
Default priority: `main` → `develop` → `master`

**Display and confirm with user:**
```
即將建立分支：<prefix>/<name>
Base branch：<base>（目前 HEAD: <short-sha>）
```

Use **AskUserQuestion**:
> "確認 base branch 是 `<base>` 嗎？"
- 確認 → proceed
- 更改 → ask for correct base branch name

**Sync the base branch before branching:**
```bash
git fetch origin
git checkout <base>
git pull
```
If `git pull` fails or there are conflicts, warn the user and stop.

---

### Step 5: Determine the workflow schema

Use the default schema (omit `--schema`) unless the user explicitly requests a different workflow.
- User mentions a schema name → use `--schema <name>`
- User asks "show workflows" → run `openspec schemas --json` and let them choose

---

### Step 6: Create the change directory

```bash
openspec new change "<name>"
```
Add `--schema <name>` only if the user requested a specific workflow.

---

### Step 7: Create the Git branch

```bash
git checkout -b <prefix>/<name>
```

---

### Step 8: Handle push based on branch type

- **`explore/` branches**: Ask:
  > "要不要 push 到 GitLab？（explore 分支通常用於本地快速驗證，可以之後再 push）"
  - Yes → `git push -u origin explore/<name>`
  - No → keep local only, show: `git push -u origin explore/<name>` for manual use later

- **All other branches** (`feat/`, `fix/`, `hotfix/`, `refactor/`): Always push:
  ```bash
  git push -u origin <prefix>/<name>
  ```

**If `git push` fails**: warn but do NOT block. Suggest `git remote add origin <url>`.

---

### Step 9: Show artifact requirements for this branch type

Based on the branch type, display the **minimum artifact requirements**:

| 分支類型 | 最低 Artifact 要求 |
|---------|-------------------|
| `feat/` | `proposal.md` + `design.md` + `tasks.md` |
| `refactor/` | `proposal.md` + `design.md` + `tasks.md` |
| `fix/` | `tasks.md` + simplified `design.md`（design note 即可） |
| `explore/` | `note.md`（hypotheses / test log / observations） |
| `hotfix/` | `incident.md`（事件描述）+ `rollback.md`（回滾步驟） |

Display as a reminder:
```
依 <branch-type>/ 分支類型，最低 artifact 要求：
  - <artifact 1>
  - <artifact 2>
建議完成這些 artifacts 後再執行 /opsx-gl-apply。
```

**`explore/` note.md 建議結構：**
```markdown
## Hypothesis
## Test Log
## Observations
## Conclusion
```

**`hotfix/` incident.md 建議結構：**
```markdown
## Incident Summary
## Root Cause
## Fix Applied
```

**`hotfix/` rollback.md 建議結構：**
```markdown
## Rollback Steps
## Rollback Trigger Condition
```

---

### Step 10: Show artifact status and first artifact instructions

```bash
openspec status --change "<name>"
openspec instructions <first-artifact-id> --change "<name>"
```

---

### Step 11: STOP and wait for user direction

---

## Output Summary

After completing the steps, summarize:
- Change name and location
- **Git branch name** and type
- **Base branch** used
- Whether pushed to GitLab
- **Minimum artifact requirements** for this branch type
- Schema/workflow and artifact sequence
- Current status (0/N artifacts complete)
- First artifact template
- Prompt: "Ready to create the first artifact? Just describe what this change is about and I'll draft it."

---

## Guardrails

- Do NOT create any artifacts yet — just show instructions
- Do NOT advance beyond showing the first artifact template
- Always run preflight checks before branching
- Always confirm base branch with user before executing `git checkout -b`
- Always fetch + pull base branch before branching
- If the change name already exists, suggest continuing that change instead
- For `explore/` branches, always ask before pushing
- Never force-push
- If git is not initialized, run `git init` first and warn the user
- If `glab` not installed: `winget install Git.GitLabCLI` (Windows) / `brew install glab` (macOS)
- If `glab` not authenticated: `glab auth login`
