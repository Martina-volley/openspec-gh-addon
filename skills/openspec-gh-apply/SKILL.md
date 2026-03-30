---
name: openspec-gh-apply
description: Implement tasks from an OpenSpec change with GitHub integration. Commits progress, pushes on completion or pause, and optionally enforces TDD. Includes explore/ branch lifecycle management.
license: MIT
compatibility: Requires openspec CLI, git CLI, and optionally gh CLI.
metadata:
  author: custom
  version: "1.2"
  basedOn: openspec-apply-change
---

Implement tasks from an OpenSpec change with automatic Git commits and GitHub push.
This is a **non-destructive extension** of the standard `/opsx-apply` workflow.

**Input**: Optionally specify a change name. If omitted, infer from conversation context.
Optional flags:
- `--scope <custom>` → override commit scope (default: change name)
- `--co-author "Name <email>"` → add Co-Authored-By to every commit this session

---

## Steps

### Step 1: Select the change

If a name is provided, use it. Otherwise:
- Infer from conversation context
- Auto-select if only one active change exists
- If ambiguous, run `openspec list --json` and use **AskUserQuestion** to let user select

Always announce: "Using change: <name>"

---

### Step 2: Detect branch and determine mode

```bash
git branch --show-current
```

| Branch prefix | Mode | TDD | Commit type |
|--------------|------|-----|-------------|
| `explore/` | Exploratory | Not required | `wip(scope):` |
| `feat/` | Feature | Recommended | `feat(scope):` |
| `fix/` | Bug Fix | Recommended | `fix(scope):` |
| `hotfix/` | Hotfix | Optional | `fix(scope)!:` |
| `refactor/` | Refactor | Recommended | `refactor(scope):` |

Display the detected mode prominently.

---

### Step 3: Check co-author setting

If `--co-author` was provided, store for this session. Otherwise skip.

---

### Step 4: Check status and get apply instructions

```bash
openspec status --change "<name>" --json
openspec instructions apply --change "<name>" --json
```

Handle states:
- `state: "blocked"` → show message, suggest `/opsx-continue`
- `state: "all_done"` → skip to Step 8 (completion)
- Otherwise → proceed to implementation

---

### Step 5: Read context files

Read all files listed in `contextFiles` from the apply instructions output.

---

### Step 6: Show current progress

Display:
- Schema being used
- **Current Git branch and mode**
- Commit scope and co-author (if set)
- Progress: "N/M tasks complete"
- Remaining tasks overview

---

### Step 7: Implement tasks (loop — no push mid-loop)

For each pending task:
- Show which task is being worked on
- **In Feature/Fix/Refactor Mode**: suggest writing a failing test first (user can decline)
- Make the code changes required, keep changes minimal and focused
- Mark task complete: `- [ ]` → `- [x]`
- **Auto-commit after each task:**
  ```bash
  git add <only the files created or modified for this task>
  git commit -m "<type>(<scope>): <short task description>

  Co-Authored-By: Name <email>"
  ```
  Omit Co-Authored-By if not set.
  For `hotfix/`: `fix(<scope>)!: <desc>`

**DO NOT push during task loop.**

**Pause if:**
- Task is unclear → ask for clarification
- Implementation reveals a design issue → suggest updating artifacts
- Error or blocker encountered → report and wait for guidance

---

### Step 8: On completion or pause — push decision

**For `explore/` branches**: Do NOT auto-push. Go to **Step 9 (explore lifecycle)** instead.

**For all other branches**: Push:
```bash
git push
```
Then show completion summary and suggest next steps:
1. `/opsx-gh-verify` — 先驗收實作內容
2. `/opsx-gh-pr` — 驗收通過後再建立 Pull Request

---

### Step 9: explore/ branch — lifecycle exit choice

**Only for `explore/` branches after all tasks are complete or when pausing.**

Display a summary of what was explored and ask the user to choose an exit path using **AskUserQuestion**:

> "explore/ 分支驗證完成，接下來要怎麼處理？"

| 選項 | 說明 | 動作 |
|------|------|------|
| A. 關閉 / 歸檔 | 驗證結果不採用，清理掉 | 歸檔 change，刪除本地分支 |
| B. 轉為正式分支 | 驗證通過，整理後走正式流程 | 建立 `feat/<name>` 或 `fix/<name>` |
| C. 保留為 Draft PR | 分享討論用，但不 merge | push + 建 Draft PR |
| D. 升格為正式功能 | 確認可交付，直接升格 | 走升格流程（見下） |

**執行各選項：**

**選項 A（關閉 / 歸檔）：**
```bash
git checkout main
git branch -d explore/<name>
```
Then run `/opsx-archive <name>`.

**選項 B（轉為正式分支）：**
1. Ask: "轉為 `feat/` 還是 `fix/<name>`？"
2. Ask: "要 cherry-pick 現有 commits，還是重新整理後提交？"
3. Create new branch from base:
   ```bash
   git fetch origin
   git checkout main && git pull
   git checkout -b feat/<name>   # or fix/<name>
   ```
4. Archive the explore/ change, create a new change for the formal branch.
5. Prompt user to continue with `/opsx-gh-apply` on the new branch.

**選項 C（保留為 Draft PR）：**
```bash
git push -u origin explore/<name>
```
Then suggest `/opsx-gh-pr` (which will create a Draft PR).
**Remind**: This Draft PR cannot be merged directly. It is for review and discussion only.

**選項 D（升格為正式功能）：**
1. Ask for confirmation:
   > "確認要升格？這會建立一個新的正式分支並重新整理 commits。"
2. Ask: "升格為 `feat/<name>` 還是 `fix/<name>`？"
3. Create new branch:
   ```bash
   git fetch origin
   git checkout main && git pull
   git checkout -b feat/<name>
   ```
4. Cherry-pick or re-commit work from the explore/ branch.
5. Archive the original explore/ change.
6. Push new branch and proceed to `/opsx-gh-pr`.

---

## Output During Implementation

```
## Implementing: <change-name> (schema: <schema-name>)
🏗️ Feature Mode | Branch: feat/<change-name>
📝 Commit scope: <scope> | Co-author: <name or none>

Working on task 3/7: <task description>
✓ Task complete → committed: feat(<scope>): <desc>
```

## Output On Completion (non-explore)

```
## Implementation Complete

**Change:** <change-name>
**Branch:** feat/<change-name>
**Mode:** Feature (TDD)
**Progress:** 7/7 tasks complete ✓

### Git Summary
- 7 commits (local) → pushing now...
- ✓ Pushed to origin/feat/<change-name>

All tasks complete! 🎉
→ Run `/opsx-gh-verify` to verify implementation before opening PR
→ Then run `/opsx-gh-pr` to create a GitHub Pull Request
```

## Output On Completion (explore)

```
## Exploration Complete

**Change:** <change-name>
**Branch:** explore/<change-name>
**Progress:** N/N tasks complete ✓

📋 Exploration summary: [brief notes from note.md]

Choose your next step: (showing exit options)
```

---

## Guardrails

- Keep going through tasks until done or blocked
- Always read context files before starting
- TDD is **recommended** in feat/fix/refactor branches, not forced
- TDD is **not required** in explore/ or hotfix/ branches
- Commit after each task with the correct type prefix
- **Push only on completion or pause — never push mid-loop**
- **explore/ branches MUST go through exit choice — never auto-push or auto-PR**
- explore/ Draft PR can be created (option C) but cannot be merged — `opsx-gh-merge` will block it
- Keep code changes minimal and scoped to each task
