---
name: openspec-gl-mr
description: Create a GitLab Merge Request from an OpenSpec change. Includes diff check before push, base branch confirmation, bilingual MR body, and explore/ WIP MR protection.
license: MIT
compatibility: Requires openspec CLI, git CLI, and glab CLI (GitLab CLI).
metadata:
  author: custom
  version: "1.0"
  basedOn: openspec-gh-pr
---

Create a GitLab Merge Request using OpenSpec artifacts as the MR description.
The MR description is **bilingual**: full Chinese version first, followed by an English summary.

**Prerequisites**:
- `glab` CLI must be installed and authenticated (`glab auth login`)
- The current branch should have been pushed to a remote
- 建議先執行 `/opsx-gl-verify` 驗收實作，確認無誤再開 MR

**Input**: Optionally specify a change name. If omitted, infer from the current branch name or conversation context.

---

## Steps

### Step 1: Identify the change

If a name is provided, use it. Otherwise:
- Parse the current branch name: `feat/<name>`, `fix/<name>`, `hotfix/<name>`, `refactor/<name>`, `explore/<name>`
- If still unclear, run `openspec list --json` and let the user select

Announce: "Creating MR for change: <name>"

---

### Step 2: Detect branch type and set MR mode

| Branch prefix | MR type | Title prefix |
|--------------|---------|--------------|
| `explore/` | **WIP MR** (`--wip`) | `WIP:` |
| `feat/` | Regular MR | none |
| `fix/` | Regular MR | none |
| `hotfix/` | Regular MR | `[HOTFIX]` |
| `refactor/` | Regular MR | none |

**explore/ protection**: Remind the user:
> "⚠️ explore/ 分支建立的是 **WIP Merge Request**，不得直接 merge 到 main。這個 MR 僅供分享和討論。若要正式合併，請先執行升格流程（`/opsx-gl-apply` 選項 D）。"

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
MR 目標：<branch> → <base>
```

Use **AskUserQuestion**:
> "MR 的 target branch 是 `<base>`，確認嗎？"
- 確認 → proceed
- 更改 → ask for correct target branch name

---

### Step 4: Diff check before push

**Check if there are any changes to push:**
```bash
git status --porcelain
git log origin/<current-branch>..HEAD --oneline 2>/dev/null
```

| Situation | Action |
|-----------|--------|
| Uncommitted changes exist | `git add -A && git commit -m "chore(<name>): final changes before MR"` then push |
| Local commits not yet pushed | `git push` |
| Local and remote are identical | **Skip push entirely** |

**If MR already exists, check if description needs updating:**
```bash
glab mr view 2>/dev/null
```
- If MR exists and new description is **identical** to existing → skip MR update, show existing URL
- If MR exists and description **differs** → use `glab mr update <id> --description "<new body>"` to update
- If no MR exists → create new MR (Step 7)

---

### Step 5: Build the bilingual MR description

Read from `openspec/changes/<name>/` if files exist:
- `proposal.md` → 概述
- `specs/` directory → 規格
- `design.md` → 設計決策
- `tasks.md` → 任務進度
- `incident.md` → 事件摘要（hotfix only）
- `rollback.md` → 回滾步驟（hotfix only）
- `note.md` → 探索筆記（explore only）

**MR description structure:**

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

### Step 6: Determine the MR title

- `hotfix/` → `[HOTFIX] <title>`
- `explore/` → `WIP: <title>`
- Others → derive from change name or first line of `proposal.md`

---

### Step 7: Create or update the Merge Request

**Create new MR:**

For `feat/`, `fix/`, `refactor/`:
```bash
glab mr create --title "<MR Title>" --description "<composed body>" --target-branch <confirmed-base>
```

For `hotfix/`:
```bash
glab mr create --title "[HOTFIX] <MR Title>" --description "<composed body>" --target-branch <confirmed-base>
```

For `explore/`:
```bash
glab mr create --title "WIP: <MR Title>" --description "<composed body>" --target-branch <confirmed-base> --wip
```

**Update existing MR** (if description changed):
```bash
glab mr update <id> --description "<new body>"
```

---

### Step 8: Display the result

Show:
- MR URL
- MR type (WIP / Regular)
- Target branch (confirmed)
- Sections included in MR description
- For `explore/`: reminder that WIP MR cannot be merged directly

---

## Output On Success

```
## Merge Request Created

**MR:** <URL>
**Type:** Regular MR (or WIP MR)
**Branch:** feat/<name> -> main
**Title:** <MR title>

### MR Description Includes:
- [OK] 概述 / Overview
- [OK] 規格 / Specification
- [OK] 設計決策 / Design Decisions
- [OK] 任務進度
- [OK] English Summary

-> View and review at the link above.
-> Run `/opsx-gl-merge` when the MR is approved.
```

---

## Guardrails

- Always confirm target branch before creating the MR
- Always check diff before pushing — skip push if nothing changed
- If MR exists and description is unchanged, skip update and show existing URL
- Use `--wip` for `explore/` branches
- Add `[HOTFIX]` prefix for `hotfix/` branches
- If `glab` not installed: `winget install Git.GitLabCLI` (Windows) / `brew install glab` (macOS)
- If `glab` not authenticated: `glab auth login`
- Never force-push
- English Summary is always AI-generated — never leave empty if Chinese content exists
