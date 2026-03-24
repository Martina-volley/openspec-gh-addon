# opsx-gh* 操作指引

> OpenSpec + GitHub / GitLab 整合工作流程說明文件
> 版本：v1.3 | 更新：2026-03-24

---

## 概覽

`opsx-gh*` 是一套結合 OpenSpec change management 與 GitHub / GitLab 的 AI 驅動工作流程。
透過 5 個主要 skill，從建立分支到合併 PR/MR，全程有結構地管理每個功能的開發週期。

```
/opsx-gh-new  →  建 artifacts  →  /opsx-gh-apply  →  /opsx-gh-verify  →  /opsx-gh-pr  →  /opsx-gh-merge  →  /opsx-archive
```

---

## 名詞說明（避免誤解）

| 名詞 | 實際意義 | 常見誤解 |
|------|---------|---------|
| `archive` | 把 OpenSpec 本地的 `changes/<name>/` 資料夾**移動**到 `changes/archive/`，純本地整理 | ❌ 不是 git 操作，不是 push，與分支無關 |
| `merge` | 將 GitHub PR / GitLab MR 合併到目標分支 | ❌ 不包含 archive，兩個是獨立步驟 |
| `verify` | 用 openspec 驗收實作內容是否符合 spec，開 PR 前的自我檢查 | ❌ 不是 code review，是 AI 自動比對 spec |
| `apply` | 逐一實作 tasks.md 裡的任務，自動 commit | ❌ 不會自動 push，完成後才 push |
| `explore/` 分支 | 本地快速驗證用，**不能直接 merge** | ❌ 不能跳過升格流程直接 merge |

---

## 前置需求

| 工具 | 說明 | 確認指令 |
|------|------|---------|
| `git` | 版本控制 | `git --version` |
| `gh` 或 `glab` | GitHub CLI / GitLab CLI | `gh --version` / `glab --version` |
| `openspec` | Change management CLI | `openspec --version` |
| GitHub / GitLab 認證 | 已登入對應 CLI | `gh auth status` |

---

## 分支類型速查表

| 分支前綴 | 使用情境 | TDD | PR/MR 類型 | Merge 策略 |
|---------|---------|-----|-----------|-----------|
| `feat/` | 新功能開發 | 建議 | Regular | Squash |
| `fix/` | 修復已知 bug | 建議 | Regular | Squash |
| `hotfix/` | Production 緊急修正 | 非必要 | Regular `[HOTFIX]` | Merge commit |
| `refactor/` | 重構，不加新功能 | 建議 | Regular | Squash |
| `explore/` | 快速驗證 / Spike | 不強制 | Draft / WIP（不得 merge） | Squash（需升格） |

---

## Skill 詳細說明

### `/opsx-gh-new` — 建立新的工作分支

**用途**：建立 OpenSpec change + Git 分支，初始化開發環境

**使用時機**：要開始一個新功能、修 bug、或快速驗證想法時

**執行流程**：
1. 說明要做什麼（change 名稱或描述）
2. Preflight 檢查（uncommitted changes、相似分支偵測）
3. 確認是全新功能還是既有功能延伸
4. 選擇分支類型（feat / fix / hotfix / refactor / explore）
5. **確認 base branch**（預設 main，可更改）
6. 建立 OpenSpec change 目錄
7. 建立 Git 分支並 push（explore 分支詢問是否 push）
8. 顯示此分支類型的 **最低 Artifact 要求**

**最低 Artifact 要求**：
| 分支類型 | 需要的 Artifacts |
|---------|----------------|
| `feat/` `refactor/` | `proposal.md` + `design.md` + `tasks.md` |
| `fix/` | `tasks.md` + `design.md`（設計筆記即可） |
| `explore/` | `note.md`（Hypothesis / Test Log / Observations / Conclusion） |
| `hotfix/` | `incident.md`（事件描述 + Root Cause）+ `rollback.md`（回滾步驟） |

**常用指令範例**：
```
/opsx-gh-new add-user-auth
/opsx-gh-new fix-login-timeout --fix
/opsx-gh-new payment-spike --explore
/opsx-gh-new db-index-tuning --hotfix
```

---

### 建立 Artifacts（`/opsx-continue` 或 `/opsx-ff`）

**這個步驟不在 opsx-gh* 範疇內，但必須完成後才能進行 apply。**

建議使用：
- `/opsx-continue` — 逐步引導建立 artifacts
- `/opsx-ff <change-name>` — 快速完成 artifacts（若已有明確需求）

---

### `/opsx-gh-apply` — 實作 Tasks + 自動 commit

**用途**：依 OpenSpec tasks 實作程式碼，每個 task 自動 commit

**使用時機**：Artifacts 準備好，可以開始寫 code 時

**執行流程**：
1. 偵測當前分支類型，決定 commit 格式
2. 取得 tasks 清單與 apply 指引
3. 逐一實作 → 標記 `[x]` → commit（**不會 mid-loop push**）
4. 全部完成或暫停時：push（非 explore 分支）
5. 推薦接著執行 `/opsx-gh-verify` 再開 PR

**Commit 格式**（依分支自動判斷）：
```
feat(scope): add login endpoint
fix(scope): resolve timeout on retry
fix(scope)!: patch critical null pointer in payment
refactor(scope): extract auth middleware
wip(scope): spike on redis session store
```

**可選 flags**：
```
/opsx-gh-apply --scope payments          # 覆蓋 commit scope
/opsx-gh-apply --co-author "Alice <alice@example.com>"  # 加 Co-Authored-By
```

**explore/ 分支完成後的出口選項**：

| 選項 | 說明 | 後續動作 |
|------|------|---------|
| **A. 關閉 / 歸檔** | 驗證結果不採用 | 刪除本地分支 + `/opsx-archive` |
| **B. 轉為正式分支** | 驗證通過，整理後走正式流程 | 建 `feat/<name>` 重新提交 |
| **C. 保留為 Draft/WIP PR** | 分享討論用，不 merge | push + `/opsx-gh-pr`（產生 Draft PR） |
| **D. 升格為正式功能** | 確認可交付，直接升格 | cherry-pick 到新正式分支 + `/opsx-gh-pr` |

> ⚠️ `explore/` 分支**不得直接 merge 到 main**，必須先走上述出口之一。

---

### `/opsx-gh-verify` — 驗收實作（開 PR 前）

**用途**：確認實作內容是否符合 OpenSpec artifacts 的規格，**開 PR 之前的自我驗收**

**使用時機**：`/opsx-gh-apply` 完成後，push 之後，建 PR 之前

**重要**：verify 的目的是確保開出的 PR body 內容（artifacts）與實際程式碼一致，避免 PR review 時才發現落差。

---

### `/opsx-gh-pr` — 建立 GitHub Pull Request

**用途**：從 OpenSpec artifacts 自動組合雙語 PR body，建立 PR

**使用時機**：`/opsx-gh-verify` 驗收通過後

**前置條件**：
- `gh` CLI 已安裝並登入（`gh auth login`）
- 建議先執行 `/opsx-gh-verify` 確認實作無誤

**執行流程**：
1. 確認 PR 目標分支（顯示 `<branch> → <base>`，需使用者確認）
2. **差異比較**：無差異則跳過 push；PR body 無變動則跳過更新
3. 讀取 OpenSpec artifacts，組合雙語 PR body
4. 建立或更新 PR

**PR body 結構**：
```markdown
## 概述        ← 來自 proposal.md
## 規格        ← 來自 specs/
## 設計決策    ← 來自 design.md
## 任務進度    ← 來自 tasks.md（中文區段限定）

---
*Generated from OpenSpec change: `<name>`*

---

## English Summary
### Overview        ← AI 翻譯摘要
### Key Specifications
### Design Notes
```

**PR 類型**：
- `explore/` → Draft PR（`--draft`，標題加 `[Draft]`）
- `hotfix/` → Regular PR（標題加 `[HOTFIX]`）
- 其他 → Regular PR

---

### `/opsx-gh-merge` — 合併 PR + 清理分支

**用途**：驗證 PR 條件並合併，清理本地分支，可選目標分支與版號 tag

**使用時機**：PR 已獲得 review approval 後

**執行流程**：
1. 取得 PR 狀態（approval、CI、衝突）
2. **explore/ 分支直接阻擋**（除非明確輸入 `--force-explore-merge` + `YES`）
3. **選擇目標分支**（三層選單）：
   - 主線：`main` / `master` / `develop`
   - Release：`release/x.x.x`（列出現有的）
   - 自訂：手動輸入任意分支名稱
4. 驗證合併條件（見下表）
5. 顯示 merge 策略後執行
6. 清理本地分支（`git checkout <target> && git pull && git branch -d`）
7. **詢問是否建立 Git tag**（所有分支類型）：輸入版號（如 v1.2.0）或留空跳過

**合併條件**：
| 分支類型 | 需求 |
|---------|------|
| `feat/` `fix/` `refactor/` | ≥ 1 approved review（必要） |
| `hotfix/` | CI pass **或** ≥ 1 approval（其中一個即可） |

**衝突**：若 PR 有衝突 → 立即停止，顯示衝突檔案清單

**Merge 策略**：
- `feat/` `fix/` `refactor/` `explore/`（例外）→ Squash merge
- `hotfix/` → Merge commit（保留完整緊急修正 context）

**Tag 說明**：
- Tag 在 `git pull` 同步目標分支後建立，確保打在 merge 後的正確 commit 上
- `git tag <name>` 只在本機建立，需 `git push origin <tag>` 才會上傳到 GitHub/GitLab

---

### `/opsx-archive` — 歸檔 OpenSpec change

**用途**：將本地 OpenSpec change 資料夾移動到 `changes/archive/`，整理工作目錄

**使用時機**：PR merge 完成後的最後一步

**重要**：這個步驟**與 git 完全無關**，只是把 `openspec/changes/<name>/` 搬到 `openspec/changes/archive/YYYY-MM-DD-<name>/`。分支已在 merge 時刪除，archive 是 OpenSpec 本地的資料清理。

---

## GitLab 版本

所有 GitHub skills 均有對應的 GitLab 版本，差異如下：

| GitHub | GitLab | 差異 |
|--------|--------|------|
| `gh` CLI | `glab` CLI | 工具不同 |
| Pull Request (PR) | Merge Request (MR) | 名稱不同 |
| Draft PR (`--draft`) | WIP MR (`--wip`) | 旗標不同 |
| `/opsx-gh-pr` | `/opsx-gl-mr` | skill 名稱不同 |
| `/opsx-gh-merge` | `/opsx-gl-merge` | skill 名稱不同 |

核心流程、驗收邏輯、branch 規則完全相同。

---

## 常見使用情境

### 情境 1：開發新功能

```
1. /opsx-gh-new add-oauth-login
   → 選 feat/
   → 確認 base: main
   → 顯示需要 proposal.md + design.md + tasks.md

2. /opsx-ff add-oauth-login          （快速建立 artifacts）

3. /opsx-gh-apply
   → 自動 commit 每個 task
   → 完成後 git push

4. /opsx-gh-verify                   （驗收實作符合 spec）

5. /opsx-gh-pr
   → 確認 PR base: main
   → 建立雙語 PR

6. （team member review & approve）

7. /opsx-gh-merge
   → 選目標分支（main）
   → squash merge
   → 詢問 tag（如 v1.3.0）
   → 清理分支

8. /opsx-archive add-oauth-login
   → 本地 OpenSpec 資料夾移到 archive/
```

---

### 情境 2：緊急 Hotfix

```
1. /opsx-gh-new null-ptr-payment --hotfix
   → 需要 incident.md + rollback.md

2. 快速建立 incident.md 和 rollback.md

3. /opsx-gh-apply
   → commit: fix(payment)!: patch null pointer on refund

4. /opsx-gh-verify

5. /opsx-gh-pr
   → PR 標題: [HOTFIX] patch null pointer on refund
   → PR 含 Incident Summary + Rollback Steps

6. /opsx-gh-merge
   → 選目標分支（main）
   → merge commit（保留 context）
   → 詢問 tag（如 v1.2.1）

7. /opsx-archive null-ptr-payment
```

---

### 情境 3：快速驗證 Spike（explore）

```
1. /opsx-gh-new redis-session-spike --explore
   → 詢問是否 push（可選 No，保留 local）
   → 需要 note.md

2. 建立 note.md（Hypothesis / Test Log）

3. /opsx-gh-apply
   → commit: wip(scope): ...
   → 完成後顯示出口選項

4. 選擇出口：
   A. 驗證失敗 → 歸檔，刪除分支
   B. 驗證通過，整理後正式化 → 建 feat/ 分支重新開始
   C. 分享討論 → push + Draft PR（不能 merge）
   D. 直接升格 → cherry-pick 到 feat/ 分支，走完整 PR 流程
```

---

### 情境 4：多人協作

```
1. /opsx-gh-apply --co-author "Bob <bob@example.com>"
   → 每個 commit 自動加上 Co-Authored-By: Bob <bob@example.com>

2. /opsx-gh-pr
   → PR body 包含完整 tasks 進度供 reviewer 對照
```

---

### 情境 5：merge 到非 main 分支（Release 流程）

```
1. 正常走 new → ff → apply → verify → pr

2. /opsx-gh-merge
   → 選目標分支清單中選 release/1.3.0
   → squash merge 到 release 分支
   → 詢問 tag（如 v1.3.0-rc1）

3. 等 release 分支通過測試後，再另開一個 PR 從 release/1.3.0 → main
```

---

## 缺少功能 / 潛在改善點

### 高優先（功能缺口）

| # | 項目 | 說明 |
|---|------|------|
| 1 | **`/opsx-gh-status`** | 顯示當前 PR 狀態（CI 結果、review 人數、mergeable）而不執行 merge |
| 2 | **Reviewer 指定** | `/opsx-gh-pr` 時，可透過 `--reviewer @alice` 指定 reviewer |
| 3 | **Conflict 解決指引** | `/opsx-gh-merge` 偵測到衝突時，提供解決步驟指引 |
| 4 | **explore/ 選項 B 的 cherry-pick 細節** | 目前缺乏具體的 cherry-pick 或 rebase 指引 |

### 中優先（體驗改善）

| # | 項目 | 說明 |
|---|------|------|
| 5 | **`/opsx-gh-overview`** | 列出所有 active change 及對應的 PR 狀態 |
| 6 | **PR label / milestone** | 建 PR 時自動加上 label 或 milestone |
| 7 | **`/opsx-gh-revert`** | 已 merge 的 PR 需要 revert 時，自動產生 revert commit 並建新 PR |

---

## 常見問題

**Q: verify 是必要步驟嗎？**
A: 不強制，但強烈建議。verify 讓你在 PR 開出去之前先確認實作符合 spec，避免 reviewer 發現落差需要重新修改。

**Q: PR 建好了，但 artifacts 更新了，body 需要重新產生嗎？**
A: 直接再執行 `/opsx-gh-pr`，skill 會比較差異，若 body 有變動才更新，不會重複觸發 CI。

**Q: merge 一定要 merge 到 main 嗎？**
A: 不是，`/opsx-gh-merge` 提供三層目標分支選單（主線 / release/* / 自訂），可以 merge 到任意分支。

**Q: tag 什麼時候建立？**
A: merge 完成後，`git pull` 同步目標分支之後才建立 tag。這樣 tag 才是打在包含你的 merge 的正確 commit 上。

**Q: `git branch -d` 刪不掉本地分支怎麼辦？**
A: `/opsx-gh-merge` 偵測到這種情況會詢問是否強制刪除（`-D`），不會自動強制執行。

**Q: explore/ 分支可以強制 merge 嗎？**
A: 可以，但需要輸入 `--force-explore-merge`，然後再次輸入 `YES` 確認。建議走升格流程（選項 D）。

**Q: Co-author 設定只對這次 session 有效嗎？**
A: 是的，`--co-author` 設定是 session 層級，下次執行需要重新指定。

**Q: GitHub 版和 GitLab 版可以混用嗎？**
A: 不行，`/opsx-gh-*` 使用 `gh` CLI，`/opsx-gl-*` 使用 `glab` CLI，需要分開使用。

---

*文件由 AI 生成，基於 opsx-gh* skill suite v1.3*
