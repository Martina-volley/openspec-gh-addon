# opsx-gh — OpenSpec × GitHub Skill Suite

> AI-driven GitHub workflow skills for OpenSpec change management
> 結合 OpenSpec 與 GitHub 的 AI 工作流程 Skill 套件

**Version**: v1.2 | **License**: MIT | **Requires**: Claude Code + GitHub CLI + OpenSpec CLI

---

## 這是什麼 / What is this?

`opsx-gh` 是一套 [Claude Code](https://claude.ai/claude-code) Skills，將 OpenSpec change management 與 GitHub 的完整開發流程串起來：

```
/opsx-gh-new  →  建 Artifacts  →  /opsx-gh-apply  →  /opsx-gh-pr  →  /opsx-gh-merge
```

4 個 Skill 涵蓋：建立分支、實作 commit、建立雙語 PR、驗證並合併，全程有結構。

---

## 前置需求 / Prerequisites

| 工具 | 安裝方式 | 確認指令 |
|------|---------|---------|
| **Claude Code** | [claude.ai/claude-code](https://claude.ai/claude-code) | `claude --version` |
| **Git** | [git-scm.com](https://git-scm.com) | `git --version` |
| **GitHub CLI** | 見下方 | `gh --version` |
| **OpenSpec CLI** | 依內部文件安裝 | `openspec --version` |

**安裝 GitHub CLI：**
```bash
# Windows
winget install GitHub.cli

# macOS
brew install gh

# 安裝後登入
gh auth login
```

**驗證所有工具就緒：**
```bash
git --version && gh auth status && openspec --version
```

---

## 安裝 / Installation

### 方式 A：Clone 到個人 skills 目錄（推薦）

```bash
# 1. Clone 這個 repo
git clone https://github.com/<your-username>/opsx-gh.git

# 2a. 複製 skills 到 Claude Code 個人目錄（全域可用）
cp -r opsx-gh/skills/openspec-gh-* ~/.claude/skills/

# 2b. 或複製 workflows 到專案 .agent/ 目錄（專案層級）
cp opsx-gh/workflows/opsx-gh-*.md your-project/.agent/workflows/
cp -r opsx-gh/skills/openspec-gh-* your-project/.agent/skills/
```

### 方式 B：直接複製需要的 Skill

```bash
# 只複製特定 skill（例如只要 opsx-gh-new）
mkdir -p ~/.claude/skills/openspec-gh-new
curl -o ~/.claude/skills/openspec-gh-new/SKILL.md \
  https://raw.githubusercontent.com/<your-username>/opsx-gh/main/skills/openspec-gh-new/SKILL.md
```

---

## 快速開始 / Quick Start

### 開一個新功能（3 步驟）

```
1. /opsx-gh-new add-user-auth
   → 選 feat/，確認 base: main
   → 顯示需要的 Artifacts 清單

2. /opsx-ff add-user-auth     （快速建立 artifacts）

3. /opsx-gh-apply              （實作 tasks，自動 commit）

4. /opsx-gh-pr                 （建立雙語 PR）

5. /opsx-gh-merge              （review 通過後合併）
```

### 緊急修正（Hotfix）

```
1. /opsx-gh-new critical-bug --hotfix
   → 需要 incident.md + rollback.md

2. /opsx-gh-apply
   → commit: fix(scope)!: patch critical bug

3. /opsx-gh-pr
   → PR 含 [HOTFIX] 標題 + Incident Summary

4. /opsx-gh-merge
   → merge commit + 詢問是否建立 git tag
```

---

## Skills 總覽

| Skill | 用途 | 版本 |
|-------|------|------|
| [`/opsx-gh-new`](skills/openspec-gh-new/SKILL.md) | 建立 OpenSpec change + Git 分支，含 Preflight 檢查 | v1.2 |
| [`/opsx-gh-apply`](skills/openspec-gh-apply/SKILL.md) | 實作 tasks，自動 commit，管理 explore/ 出口 | v1.2 |
| [`/opsx-gh-pr`](skills/openspec-gh-pr/SKILL.md) | 建立雙語 PR，含差異比較與 Draft PR 保護 | v1.2 |
| [`/opsx-gh-merge`](skills/openspec-gh-merge/SKILL.md) | 驗證並合併 PR，清理分支，阻止 explore/ 直接 merge | v1.1 |

---

## 分支類型速查

| 前綴 | 使用情境 | Merge 策略 | PR 類型 |
|------|---------|-----------|---------|
| `feat/` | 新功能 | Squash | Regular |
| `fix/` | 修 bug | Squash | Regular |
| `hotfix/` | 緊急修正 | Merge commit | Regular `[HOTFIX]` |
| `refactor/` | 重構 | Squash | Regular |
| `explore/` | 快速驗證 | Squash（需升格） | **Draft 只能** |

> ⚠️ `explore/` 分支**不得直接 merge**，需先走升格流程或轉為正式分支。

---

## 完整說明文件

- [操作指引（中文）](.agent/opsx-gh-guide.md) — 詳細說明、完整流程圖、常見問題
- [流程圖](.agent/opsx-gh-flowchart.md) — Mermaid 圖表說明全流程

---

## 常見錯誤排查

**`gh: command not found`**
→ GitHub CLI 未安裝，執行 `winget install GitHub.cli` 或 `brew install gh`

**`gh auth status` 顯示未登入**
→ 執行 `gh auth login`，選 GitHub.com，使用瀏覽器認證

**`openspec: command not found`**
→ OpenSpec CLI 未安裝，請參考內部安裝文件

**`git push` 失敗（remote 不存在）**
→ 執行 `git remote add origin https://github.com/<org>/<repo>.git`

**Skill 在 Claude Code 裡找不到（`/opsx-gh-new` 沒反應）**
→ 確認 SKILL.md 放在 `~/.claude/skills/openspec-gh-new/SKILL.md`
→ 或確認專案 `.agent/skills/openspec-gh-new/SKILL.md` 存在

---

## 已知限制 / Roadmap

- [ ] `/opsx-gh-status` — 查看當前 PR 狀態而不執行 merge
- [ ] `--reviewer` flag — 建 PR 時指定 reviewer
- [ ] Conflict 解決步驟指引
- [ ] GitLab（`glab`）版本支援

---

## License

MIT © 2026
