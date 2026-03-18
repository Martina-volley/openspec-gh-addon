---
description: Create a GitHub Pull Request from an OpenSpec change (diff check, base confirmation, bilingual body, explore/ protection)
---

Create a GitHub PR using OpenSpec artifacts. Includes diff check before push, base branch confirmation, and bilingual body.

**Prerequisites**: `gh` CLI installed and authenticated.
**Input**: Optionally specify change name. If omitted, infer from current branch.

---

**Steps**

1. **Identify the change** — Parse branch name or let user select.

2. **Detect branch type and set PR mode**
   | Branch | PR type | Title prefix |
   |--------|---------|--------------|
   | `explore/` | Draft PR (`--draft`) | `[Draft]` |
   | `feat/` / `fix/` / `refactor/` | Regular PR | none |
   | `hotfix/` | Regular PR | `[HOTFIX]` |

   For `explore/`: warn that Draft PR cannot be merged directly.

3. **Confirm base branch**
   Detect default (`main` → `develop` → `master`). Display:
   ```
   PR 目標：<branch> → <base>
   ```
   Ask user to confirm. Allow changing if needed.

4. **Diff check before push**
   ```bash
   git status --porcelain
   git log origin/<branch>..HEAD --oneline 2>/dev/null
   ```
   | Situation | Action |
   |-----------|--------|
   | Uncommitted changes | commit + push |
   | Unpushed commits | push |
   | No difference | **skip push** |

   If PR already exists:
   ```bash
   gh pr view --json body,number
   ```
   - Body unchanged → show existing URL, skip update
   - Body changed → `gh pr edit --body "<new body>"`

5. **Build bilingual PR description**
   From `openspec/changes/<name>/`:
   - `proposal.md` / `note.md` → 概述
   - `specs/` → 規格
   - `design.md` → 設計決策
   - `tasks.md` → 任務進度（Chinese only）
   - `incident.md` + `rollback.md` → hotfix sections

   Structure: Chinese full content → `---` → English Summary (AI-translated key points).
   Missing artifact → omit both language sections.

6. **Create or update PR**
   ```bash
   gh pr create --title "<Title>" --body "<body>" --base <confirmed-base>
   # add --draft for explore/
   ```
   Update existing: `gh pr edit <number> --body "<new body>"`

7. **Display result** — URL, type, base branch, included sections.

---

**Guardrails**
- Always confirm base branch before creating PR
- Skip push if no diff; skip PR update if body unchanged
- `explore/` → Draft PR only, cannot be merged
- `hotfix/` → `[HOTFIX]` title prefix
- If `gh` not installed, show install instructions
- Never force-push
