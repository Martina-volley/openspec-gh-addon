# opsx-gh — OpenSpec × GitHub / GitLab Skill Suite

> AI-driven Git workflow skills for OpenSpec change management
> 結合 OpenSpec 與 GitHub / GitLab 的 AI 工作流程 Skill 套件

**Version**: v1.3 | **License**: MIT

---

## 這是什麼 / What is this?

`opsx-gh` 是一套 AI Coding IDE 的 Skills，將 OpenSpec change management 與 GitHub / GitLab 的完整開發流程串起來。

**完整流程：**
```
/opsx-gh-new  →  建 Artifacts (/opsx-ff)  →  /opsx-gh-apply
  →  /opsx-gh-verify  →  /opsx-gh-pr  →  /opsx-gh-merge  →  /opsx-archive
```

---

## 名詞說明 / Glossary（避免誤解）

| 名詞 | 實際意義 | 常見誤解 |
|------|---------|---------|
| `archive` | 把 OpenSpec 本地的 `changes/<name>/` 資料夾**移動**到 `changes/archive/`，純本地整理 | ❌ 不是 git 操作，不是 push，不是刪除分支 |
| `merge` | 將 GitHub PR / GitLab MR 合併到目標分支 | ❌ 不包含 archive，兩個是獨立步驟 |
| `verify` | 用 openspec 驗收實作內容是否符合 spec | ❌ 不是 code review，是 AI 自動比對 spec |
| `apply` | 逐一實作 tasks.md 裡的任務，自動 commit | ❌ 不會自動 push（完成後才 push） |
| `explore/` 分支 | 本地快速驗證用，**不能直接 merge** | ❌ 不能跳過升格流程直接 merge |

---

## 前置需求 / Prerequisites

| 工具 | 說明 | 確認指令 |
|------|------|---------|
| **AI Coding IDE** | 見下方安裝說明 | — |
| **Git** | [git-scm.com](https://git-scm.com) | `git --version` |
| **GitHub CLI** 或 **GitLab CLI** | `gh` / `glab` | `gh --version` / `glab --version` |
| **OpenSpec CLI** | 依內部文件安裝 | `openspec --version` |

---

## 安裝 Skills / Installation

### Step 1 — 下載 Skill 檔案

```bash
git clone https://github.com/Martina-volley/openspec-gh-addon.git
```

### Step 2 — 依你的 IDE 放到對應位置

#### Claude Code（CLI）
```bash
# 全域（任何專案都能用）
cp -r openspec-gh-addon/.agent/skills/openspec-gh-* ~/.claude/skills/

# 或專案層級
cp -r openspec-gh-addon/.agent/skills/openspec-gh-* your-project/.agent/skills/
```

#### Antigravity IDE
```bash
# 專案層級 skills
cp -r openspec-gh-addon/.agent/skills/openspec-gh-* your-project/.agent/skills/

# 或放到 workflows
cp openspec-gh-addon/.agent/workflows/opsx-gh-*.md your-project/.agent/workflows/
```

#### Cursor
```bash
# 放到專案的 .cursor/skills/ 目錄
cp -r openspec-gh-addon/.agent/skills/openspec-gh-* your-project/.cursor/skills/

# 或 .cursor/commands/
cp -r openspec-gh-addon/.agent/skills/openspec-gh-* your-project/.cursor/commands/
```

#### 驗證安裝成功
在 IDE 的 AI 對話框輸入：
```
What are my available skills?
```
或直接呼叫：
```
/opsx-gh-new
```

---

## 快速開始 / Quick Start

### 開一個新功能

```
1. /opsx-gh-new add-user-auth --feat
   → 建 OpenSpec change + feat/add-user-auth 分支

2. /opsx-ff add-user-auth
   → 快速建立 proposal / design / tasks artifacts

3. /opsx-gh-apply
   → 逐一實作 tasks，每個 task 自動 commit

4. /opsx-gh-verify
   → AI 驗收：確認實作符合 spec，才開 PR

5. /opsx-gh-pr
   → 建立雙語 PR（中文主體 + 英文摘要）

6. /opsx-gh-merge
   → 選目標分支（main / release/x.x.x / 自訂）
   → merge 完詢問是否建立 git tag（例如 v1.2.0）

7. /opsx-archive add-user-auth
   → 將本地 OpenSpec change 資料夾移到 archive/
   → 這步與 git 無關，純本地整理
```

### 緊急修正（Hotfix）

```
1. /opsx-gh-new critical-bug --hotfix
   → 需要 incident.md + rollback.md

2. /opsx-gh-apply
   → commit: fix(scope)!: patch critical bug

3. /opsx-gh-verify

4. /opsx-gh-pr
   → PR 含 [HOTFIX] 標題 + Incident Summary

5. /opsx-gh-merge
   → merge commit + 詢問 git tag（例如 v1.2.1）

6. /opsx-archive critical-bug
```

---

## Skills 總覽

| Skill | 用途 |
|-------|------|
| `/opsx-gh-new` | 建立 OpenSpec change + Git 分支，含 Preflight 檢查 |
| `/opsx-gh-apply` | 實作 tasks，自動 commit，管理 explore/ 出口 |
| `/opsx-gh-verify` | 驗收實作是否符合 spec（開 PR 前） |
| `/opsx-gh-pr` | 建立雙語 PR，含差異比較與 Draft PR 保護 |
| `/opsx-gh-merge` | 合併 PR，可選目標分支，merge 後詢問 tag |
| `/opsx-gl-new` | 同上，GitLab 版 |
| `/opsx-gl-apply` | 同上，GitLab 版 |
| `/opsx-gl-mr` | 建立 GitLab Merge Request（含 WIP 保護） |
| `/opsx-gl-merge` | 合併 MR，可選目標分支，merge 後詢問 tag，GitLab 版 |

---

## 分支類型速查

| 前綴 | 使用情境 | Merge 策略 | PR/MR 類型 |
|------|---------|-----------|-----------|
| `feat/` | 新功能 | Squash | Regular |
| `fix/` | 修 bug | Squash | Regular |
| `hotfix/` | 緊急修正 | Merge commit | `[HOTFIX]` |
| `refactor/` | 重構 | Squash | Regular |
| `explore/` | 快速驗證 | 需升格後才能 merge | Draft / WIP |

> ⚠️ `explore/` 分支**不得直接 merge**，需先走升格流程（`/opsx-gh-apply` 選項 B 或 D）。

---

## 完整說明文件

- [操作指引（中文）](.agent/opsx-gh-guide.md) — 詳細說明、完整流程圖、常見問題
- [流程圖](.agent/opsx-gh-flowchart.md) — Mermaid 圖表說明全流程

---

## 常見錯誤排查

**Skill 在 IDE 裡找不到**
→ Claude Code CLI：確認放在 `~/.claude/skills/<name>/SKILL.md`
→ Antigravity IDE：確認放在專案 `.agent/skills/<name>/SKILL.md`
→ Cursor：確認放在專案 `.cursor/skills/<name>/SKILL.md`

**`gh: command not found`**
→ 安裝 GitHub CLI：`winget install GitHub.cli`（Windows）或 `brew install gh`（macOS）

**`openspec: command not found`**
→ 請參考內部 OpenSpec 安裝文件

**`git push` 失敗（remote 不存在）**
→ 執行 `git remote add origin https://github.com/<org>/<repo>.git`

---

## License

MIT © 2026
