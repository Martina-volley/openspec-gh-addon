---
name: openspec-gh-pr
description: Create a GitHub Pull Request from an OpenSpec change. Includes diff check before push, base branch confirmation, bilingual PR body, and explore/ Draft PR protection.
license: MIT
compatibility: Requires openspec CLI, git CLI, and gh CLI (GitHub CLI).
metadata:
  author: custom
  version: "1.2"
---

Create a GitHub Pull Request using OpenSpec artifacts as the PR body.
The PR body is **bilingual**: full Chinese version first, followed by an English summary.

**Prerequisites**:
- `gh` CLI must be installed and authenticated (`gh auth login`)
- The current branch should have been pushed to a remote

**Input**: Optionally specify a change name. If omitted, infer from the current branch name or conversation context.

---

## Steps

### Step 1: Identify the change

If a name is provided, use it. Otherwise:
- Parse the current branch name: `feat/<name>`, `fix/<name>`, `hotfix/<name>`, `refactor/<name>`, `explore/<name>`
- If still unclear, run `openspec list --json` and let the user select

Announce: "Creating PR for change: <name>"

---

### Step 2: Detect branch type and set PR mode

| Branch prefix | PR type | Title prefix |
|--------------|---------|--------------|
| `explore/` | **Draft PR** (`--draft`) | `[Draft]` |
| `feat/` | Regular PR | none |
| `fix/` | Regular PR | none |
| `hotfix/` | Regular PR | `[HOTFIX]` |
| `refactor/` | Regular PR | none |

**explore/ protection**: Remind the user:
> "⚠️ explore/ 分支建立的是 **Draft PR**，不得直接 merge 到 main。這個 PR 僅供分享和討論。若要正式合併，請先執行升格流程（`/opsx-gh-apply` 選項 D）。"

---

### Step 3: Confirm base branch

Check current branch and its tracking info:
```bash
git rev-parse --abbrev-ref HEAD
git rev-parse --abbrev-ref @{upstream} 2>/dev/null
```

Detect default base (`main` → `develop` → `master`).

**Display and confirm with user:**
```
PR 目標：<branch> → <base>
```

Use **AskUserQuestion**:
> "PR 的 base branch 是 `<base>`，確認嗎？"
- 確認 → proceed
- 更改 → ask for correct base branch name

---

### Step 4: Diff check before push

**Check if there are any changes to push:**
```bash
git status --porcelain
git log origin/<current-branch>..HEAD --oneline 2>/dev/null
```

| Situation | Action |
|-----------|--------|
| Uncommitted changes exist | `git add -A && git commit -m "chore(<name>): final changes before PR"` then push |
| Local commits not yet pushed | `git push` |
| Local and remote are identical | **Skip push entirely** |

**If PR already exists, check if body needs updating:**
```bash
gh pr view --json body,number 2>/dev/null
```
- If PR exists and new body is **identical** to existing → skip PR update, show existing URL
- If PR exists and body **differs** → use `gh pr edit --body "<new body>"` to update
- If no PR exists → create new PR (Step 7)

---

### Step 5: Build the bilingual PR description

Read from `openspec/changes/<name>/` if files exist:
- `proposal.md` → 概述
- `specs/` directory → 規格
- `design.md` → 設計決策
- `tasks.md` → 任務進度
- `incident.md` → 事件摘要（hotfix only）
- `rollback.md` → 回滾步驟（hotfix only）
- `note.md` → 探索筆記（explore only）

**PR body structure:**

```markdown
## 概述
<!-- from proposal.md (or note.md for explore) -->
<Chinese content>

---

## 規格
<!-- from specs/ -->
<Chinese content>

---

## 設計決策
<!-- from design.md -->
<Chinese content>

---

## 任務進度
<!-- from tasks.md -->
- [x] Task 1
- [x] Task 2

---
*Generated from OpenSpec change: `<name>`*

---

## English Summary

### Overview
<AI-translated key points from 概述>

### Key Specifications
<AI-translated key points from 規格>

### Design Notes
<AI-translated key points from 設計決策>
```

**For `hotfix/` branches**, insert before English Summary:
```markdown
## 事件摘要 / Incident Summary
<!-- from incident.md -->
<content>

## 回滾步驟 / Rollback Steps
<!-- from rollback.md -->
<content>
```

**Rules:**
- Chinese sections: copy directly from artifacts (no transformation)
- English sections: AI-generated translation summary (key points, not word-for-word)
- If artifact is already in English: Chinese and English sections contain same content
- Missing artifact → omit both Chinese and English sections for it
- `tasks.md` checklist appears in Chinese section only (no English needed)

---

### Step 6: Determine the PR title

- `hotfix/` → `[HOTFIX] <title>`
- `explore/` → `[Draft] <title>`
- Others → derive from change name or first line of `proposal.md`

---

### Step 7: Create or update the Pull Request

**Create new PR:**

For `feat/`, `fix/`, `refactor/`:
```bash
gh pr create --title "<PR Title>" --body "<composed body>" --base <confirmed-base>
```

For `hotfix/`:
```bash
gh pr create --title "[HOTFIX] <PR Title>" --body "<composed body>" --base <confirmed-base>
```

For `explore/`:
```bash
gh pr create --title "[Draft] <PR Title>" --body "<composed body>" --base <confirmed-base> --draft
```

**Update existing PR** (if body changed):
```bash
gh pr edit <number> --body "<new body>"
```

---

### Step 8: Display the result

Show:
- PR URL
- PR type (Draft / Regular)
- Base branch (confirmed)
- Sections included in PR body
- For `explore/`: reminder that Draft PR cannot be merged directly

---

## Output On Success

```
## Pull Request Created 🎉

**PR:** <URL>
**Type:** Regular PR (or Draft PR)
**Branch:** feat/<name> → main
**Title:** <PR title>

### PR Body Includes:
- ✅ 概述 / Overview
- ✅ 規格 / Specification
- ✅ 設計決策 / Design Decisions
- ✅ 任務進度
- ✅ English Summary

→ View and review at the link above.
→ Run `/opsx-gh-merge` when the PR is approved.
```

---

## Guardrails

- Always confirm base branch before creating the PR
- Always check diff before pushing — skip push if nothing changed
- If PR exists and body is unchanged, skip update and show existing URL
- Use `--draft` for `explore/` branches
- Add `[HOTFIX]` prefix for `hotfix/` branches
- If `gh` not installed: `winget install GitHub.cli` (Windows) / `brew install gh` (macOS)
- If `gh` not authenticated: `gh auth login`
- Never force-push
- English Summary is always AI-generated — never leave empty if Chinese content exists
