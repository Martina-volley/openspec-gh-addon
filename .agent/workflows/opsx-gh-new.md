---
description: Start a new change with GitHub integration (preflight checks, base branch confirmation, artifact requirements)
---

Start a new change with Git branch creation, push, and guided setup.

**Input**: Change name or description. Optional flags: `--explore`, `--feat`, `--fix`, `--hotfix`, `--refactor`

---

**Steps**

1. **Understand what to build**
   If no input, use AskUserQuestion: "What change do you want to work on?"
   Derive kebab-case name from description.

2. **Preflight checks**

   ```bash
   git status
   git branch -a | grep -E "feat/|fix/|hotfix/|refactor/|explore/"
   ```
   - Uncommitted changes → warn, ask if user wants to stash/commit first
   - Similar branch exists → show it, ask if continuing that or making new
   - Ask: "全新功能，還是延伸現有功能？" → if extension, run `openspec list` first

3. **Determine branch type**
   If no flag provided, ask user:

   | 類型 | 前綴 | TDD |
   |-----|------|-----|
   | Feature | `feat/` | 建議 |
   | Explore | `explore/` | 不強制 |
   | Bug Fix | `fix/` | 建議 |
   | Hotfix | `hotfix/` | 非必要 |
   | Refactor | `refactor/` | 建議 |

4. **Confirm base branch**
   Detect default (`main` → `develop` → `master`). Display:
   ```
   即將建立：<prefix>/<name>  Base：<base>（HEAD: <sha>）
   ```
   Ask user to confirm. If different base needed, let user specify.
   Then sync:
   ```bash
   git fetch origin
   git checkout <base>
   git pull
   ```

5. **Determine schema** — use default unless user requests specific.

6. **Create the change directory**
   ```bash
   openspec new change "<name>"
   ```

7. **Create Git branch**
   ```bash
   git checkout -b <prefix>/<name>
   ```

8. **Handle push**
   - `explore/` → ask user if they want to push (optional)
   - All other → always push:
     ```bash
     git push -u origin <prefix>/<name>
     ```
   If push fails, warn but continue.

9. **Show artifact requirements for this branch type**

   | 分支 | 最低要求 |
   |------|---------|
   | `feat/` / `refactor/` | proposal + design + tasks |
   | `fix/` | tasks + simplified design note |
   | `explore/` | note.md（hypotheses / test log） |
   | `hotfix/` | incident.md + rollback.md |

   Display as reminder before artifact step.

10. **Show artifact status and first artifact instructions**
    ```bash
    openspec status --change "<name>"
    openspec instructions <first-artifact-id> --change "<name>"
    ```

11. **STOP and wait for user direction**

---

**Guardrails**
- Always run preflight checks before branching
- Always confirm base branch before `git checkout -b`
- Always fetch + pull base before branching
- For `explore/` branches, ask before pushing
- If change name already exists, suggest continuing instead
- Never force-push
