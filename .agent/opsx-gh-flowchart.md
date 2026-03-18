# opsx-gh* 完整流程圖

## 主流程

```mermaid
flowchart TD
    START([開始新工作]) --> NEW["/opsx-gh-new"]

    NEW --> BRANCH_TYPE{選擇分支類型}

    BRANCH_TYPE -->|新功能| FEAT["feat/name"]
    BRANCH_TYPE -->|快速驗證| EXPLORE["explore/name"]
    BRANCH_TYPE -->|修 bug| FIX["fix/name"]
    BRANCH_TYPE -->|緊急修正| HOTFIX["hotfix/name"]
    BRANCH_TYPE -->|重構| REFACTOR["refactor/name"]

    FEAT --> PUSH_FEAT["git push -u origin\n（強制）"]
    FIX --> PUSH_FIX["git push -u origin\n（強制）"]
    HOTFIX --> PUSH_HOT["git push -u origin\n（強制）"]
    REFACTOR --> PUSH_REF["git push -u origin\n（強制）"]
    EXPLORE --> ASK_PUSH{要 push 嗎？}
    ASK_PUSH -->|Yes| PUSH_EXP["git push -u origin"]
    ASK_PUSH -->|No| LOCAL["保留 local only"]

    PUSH_FEAT & PUSH_FIX & PUSH_HOT & PUSH_REF & PUSH_EXP & LOCAL --> ARTIFACTS["建立 OpenSpec Artifacts\n（/opsx-continue 或 /opsx-ff）"]

    ARTIFACTS --> APPLY["/opsx-gh-apply"]
```

---

## /opsx-gh-apply 實作流程

```mermaid
flowchart TD
    APPLY(["/opsx-gh-apply"]) --> DETECT["偵測當前分支"]

    DETECT --> MODE{分支類型}
    MODE -->|"feat/ fix/ refactor/"| FEATURE_MODE["Feature/Fix/Refactor Mode\nTDD 建議\ncommit: feat/fix/refactor(scope): desc"]
    MODE -->|"explore/"| EXPLORE_MODE["Exploratory Mode\n不強制 TDD\ncommit: wip(scope): desc"]
    MODE -->|"hotfix/"| HOTFIX_MODE["Hotfix Mode\n速度優先\ncommit: fix(scope)!: desc"]

    FEATURE_MODE & EXPLORE_MODE & HOTFIX_MODE --> LOOP["實作 Task Loop"]

    LOOP --> IMPL["實作一個 task"]
    IMPL --> COMMIT["git add -A\ngit commit -m type(scope): desc\n[Co-Authored-By: ...]"]
    COMMIT --> MARK["標記 task [x]"]
    MARK --> MORE{還有 tasks?}
    MORE -->|Yes| IMPL
    MORE -->|No| ALL_DONE["所有 tasks 完成"]
    MORE -->|Blocked| PAUSED["Agent 暫停"]

    ALL_DONE --> CHECK_BRANCH1{是 explore/ 嗎？}
    PAUSED --> CHECK_BRANCH2{是 explore/ 嗎？}

    CHECK_BRANCH1 -->|Yes| ASK_PUSH1{要 push 嗎？}
    CHECK_BRANCH1 -->|No| PUSH1["git push"]
    CHECK_BRANCH2 -->|Yes| ASK_PUSH2{要 push 嗎？}
    CHECK_BRANCH2 -->|No| PUSH2["git push"]

    ASK_PUSH1 -->|Yes| PUSH1
    ASK_PUSH1 -->|No| SKIP_PUSH1["skip push\n顯示手動指令"]
    ASK_PUSH2 -->|Yes| PUSH2
    ASK_PUSH2 -->|No| SKIP_PUSH2["skip push\n顯示手動指令"]

    PUSH1 --> SUGGEST_PR["建議 /opsx-gh-pr"]
    PUSH2 --> WAIT["等待使用者解決後繼續"]
```

---

## /opsx-gh-pr PR 建立流程

```mermaid
flowchart TD
    PR(["/opsx-gh-pr"]) --> CHECK_PUSH["確認已 push\ngit push"]

    CHECK_PUSH --> READ_ART["讀取 OpenSpec Artifacts\nproposal.md / specs/ / design.md / tasks.md"]

    READ_ART --> BUILD_BODY["組合雙語 PR body"]

    BUILD_BODY --> CHINESE["🇹🇼 中文完整版\n## 概述\n## 規格\n## 設計決策\n## 任務進度"]
    BUILD_BODY --> ENGLISH["🇺🇸 English Summary\n### Overview\n### Key Specifications\n### Design Notes\n（AI 翻譯摘要）"]

    CHINESE & ENGLISH --> PR_TYPE{分支類型}

    PR_TYPE -->|"explore/"| DRAFT["Draft PR\n--draft\ntitle: [Draft] xxx"]
    PR_TYPE -->|"feat/ fix/ refactor/"| REGULAR["Regular PR\ntitle: xxx"]
    PR_TYPE -->|"hotfix/"| URGENT["Regular PR\ntitle: [HOTFIX] xxx"]

    DRAFT & REGULAR & URGENT --> CREATE["gh pr create\n--title --body --base main"]

    CREATE --> SHOW_URL["顯示 PR URL\n建議 /opsx-gh-merge"]
```

---

## /opsx-gh-merge 合併流程

```mermaid
flowchart TD
    MERGE(["/opsx-gh-merge"]) --> CHECK_PR["gh pr view\n確認 PR 狀態"]

    CHECK_PR --> CONFLICT{有衝突？}
    CONFLICT -->|Yes| STOP["❌ 停止\n顯示衝突檔案"]
    CONFLICT -->|No| VALIDATE{驗證條件}

    VALIDATE -->|"feat/fix/refactor"| NEED_APPROVAL["需要 ≥ 1 approved review"]
    VALIDATE -->|"hotfix/"| HOTFIX_CHECK["CI pass 或 1 approval\n其中一個滿足即可"]
    VALIDATE -->|"explore/"| NO_REQUIRE["無需 approval"]

    NEED_APPROVAL --> APPROVED{已 approved?}
    APPROVED -->|No| BLOCK["❌ 阻止合併\n顯示缺少條件"]
    APPROVED -->|Yes| STRATEGY

    HOTFIX_CHECK --> HOT_OK{條件滿足?}
    HOT_OK -->|No| HOT_BYPASS["⚠️ 警告\n詢問是否強制合併"]
    HOT_OK -->|Yes| STRATEGY
    HOT_BYPASS -->|確認| STRATEGY

    NO_REQUIRE --> STRATEGY

    STRATEGY{決定 merge 策略}
    STRATEGY -->|"feat/fix/refactor/explore"| SQUASH["gh pr merge --squash --delete-branch"]
    STRATEGY -->|"hotfix/"| MERGE_COMMIT["gh pr merge --merge --delete-branch"]

    SQUASH & MERGE_COMMIT --> CLEANUP["本地清理\ngit checkout main\ngit pull\ngit branch -d name"]

    CLEANUP --> IS_HOTFIX{是 hotfix?}
    IS_HOTFIX -->|Yes| ASK_TAG{建立 tag?}
    IS_HOTFIX -->|No| DONE

    ASK_TAG -->|Yes| TAG["git tag vX.Y.Z\ngit push origin vX.Y.Z"]
    ASK_TAG -->|No| DONE

    TAG --> DONE["✅ 完成\n建議 /opsx-archive"]
```

---

## 完整端到端流程（概覽）

```mermaid
flowchart LR
    A(["/opsx-gh-new\n建立分支"]) --> B["/opsx-continue 或 /opsx-ff\n建立 Artifacts"]
    B --> C(["/opsx-gh-apply\n實作 + commit"])
    C --> D(["/opsx-gh-pr\n建立 PR\n雙語 body"])
    D --> E["Code Review\n（team member）"]
    E --> F(["/opsx-gh-merge\n合併 + 清理"])
    F --> G(["/opsx-archive\n歸檔 change"])

    style A fill:#4A90D9,color:#fff
    style C fill:#4A90D9,color:#fff
    style D fill:#4A90D9,color:#fff
    style F fill:#4A90D9,color:#fff
    style G fill:#7B68EE,color:#fff
    style E fill:#F5A623,color:#fff
```

---

## 分支類型對照表

| 分支前綴 | 用途 | TDD | Push on new | Commit type | PR type | Merge strategy |
|---------|------|-----|------------|-------------|---------|----------------|
| `feat/` | 新功能 | 建議 | 強制 | `feat(scope):` | Regular | Squash |
| `explore/` | 快速驗證 | 不強制 | 詢問 | `wip(scope):` | Draft | Squash |
| `fix/` | 修 bug | 建議 | 強制 | `fix(scope):` | Regular | Squash |
| `hotfix/` | 緊急修正 | 非必要 | 強制 | `fix(scope)!:` | Regular `[HOTFIX]` | Merge commit |
| `refactor/` | 重構 | 建議 | 強制 | `refactor(scope):` | Regular | Squash |
