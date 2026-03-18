---
description: Implement tasks from an OpenSpec change with GitHub integration (includes explore/ lifecycle)
---

Implement tasks with automatic Git commits, push on completion/pause, branch-aware TDD, and explore/ exit management.

**Input**: Optionally specify change name. Optional flags: `--scope <custom>`, `--co-author "Name <email>"`

---

**Steps**

1. **Select the change** — If ambiguous, run `openspec list --json` and let user select.

2. **Detect branch and determine mode**
   ```bash
   git branch --show-current
   ```
   | Branch | Mode | TDD | Commit type |
   |--------|------|-----|-------------|
   | `explore/` | Exploratory | Not required | `wip(scope):` |
   | `feat/` | Feature | Recommended | `feat(scope):` |
   | `fix/` | Bug Fix | Recommended | `fix(scope):` |
   | `hotfix/` | Hotfix | Optional | `fix(scope)!:` |
   | `refactor/` | Refactor | Recommended | `refactor(scope):` |

3. **Check status and get apply instructions**
   ```bash
   openspec status --change "<name>" --json
   openspec instructions apply --change "<name>" --json
   ```
   - `state: "all_done"` → skip to completion step

4. **Read context files and show progress** — branch, mode, scope, co-author, N/M tasks.

5. **Implement tasks (loop — no push mid-loop)**

   For each task: implement → mark `[x]` → commit:
   ```bash
   git add -A
   git commit -m "<type>(<scope>): <short desc>

   Co-Authored-By: Name <email>"
   ```
   Omit Co-Authored-By if not set. **Never push between tasks.**

6. **On completion or pause:**

   - **Non-explore branches** → push and suggest `/opsx-gh-pr`:
     ```bash
     git push
     ```

   - **explore/ branch** → show exit choice (AskUserQuestion):

     | 選項 | 動作 |
     |------|------|
     | A. 關閉 / 歸檔 | `git branch -d explore/<name>` + archive change |
     | B. 轉為正式分支 | 建 `feat/` 或 `fix/` 分支，重新整理後走正式流程 |
     | C. 保留為 Draft PR | push + 建 Draft PR（不得 merge） |
     | D. 升格為正式功能 | cherry-pick 到新正式分支，archive explore/，走完整 PR 流程 |

---

**Guardrails**
- Commit after each task, **push only on completion or pause**
- **explore/ must go through exit choice — never auto-push or auto-PR**
- explore/ Draft PR (option C) cannot be merged; `opsx-gh-merge` will block it
- TDD is recommended in feat/fix/refactor, not forced; not required in explore/hotfix
