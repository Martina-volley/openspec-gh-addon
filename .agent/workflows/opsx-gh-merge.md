---
description: Merge an approved GitHub PR (blocks explore/ direct merge, validates approval/CI, strategy by branch type)
---

Merge an approved PR, clean up branches, and prompt to archive. Includes explore/ merge protection.

**Prerequisites**: `gh` CLI installed and authenticated. PR must already exist.
**Input**: Optionally specify change name or PR number. If omitted, infer from current branch.

---

**Steps**

1. **Identify the PR**
   ```bash
   gh pr view --json number,title,state,headRefName,baseRefName,isDraft,reviews,statusCheckRollup,mergeable
   ```
   No PR exists → error, suggest `/opsx-gh-pr` first.

2. **Block explore/ direct merge**

   If branch starts with `explore/`:
   ```
   🚫 explore/ 分支不允許直接 merge。
   請回到 /opsx-gh-apply 選擇升格流程（選項 B 或 D）。
   ```
   **Stop here.** Exception: `--force-explore-merge` flag + explicit `YES` confirmation.

3. **Validate merge readiness**

   | Branch | Approval required | CI |
   |--------|------------------|----|
   | `feat/fix/refactor` | ≥ 1 approved review | Warn if failing |
   | `hotfix/` | CI pass OR 1 approval | Either sufficient |

   - `mergeable: CONFLICTING` → stop, show conflict files
   - Missing conditions → show summary, ask confirmation before bypass (hotfix only)

4. **Determine merge strategy**

   | Branch | Strategy |
   |--------|---------|
   | `feat/`, `fix/`, `refactor/` | Squash merge |
   | `hotfix/` | Merge commit |
   | `explore/` (exception) | Squash merge |

   Show strategy before executing.

5. **Execute merge**

   Squash:
   ```bash
   gh pr merge --squash --delete-branch
   ```
   Merge commit (hotfix):
   ```bash
   gh pr merge --merge --delete-branch
   ```

6. **Clean up local branch**
   ```bash
   git checkout main
   git pull
   git branch -d <branch-name>
   ```
   If `-d` fails, ask user before using `-D`.

7. **Hotfix only: offer to create a tag**
   Ask: "要建立 hotfix tag 嗎？（如 v1.2.1）"
   ```bash
   git tag <tag-name> && git push origin <tag-name>
   ```

8. **Show summary and suggest next steps**
   → Run `/opsx-archive <change-name>` to archive.

---

**Guardrails**
- `explore/` branches are blocked — must go through promotion flow first
- `feat/fix/refactor` require at least 1 approval
- `hotfix/` allows CI pass OR 1 approval; warn if neither
- Never force-merge; if conflicts exist, stop
- Only use `git branch -D` with explicit user confirmation
- Always pull main after merge
