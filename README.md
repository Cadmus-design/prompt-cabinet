# Prompt Cabinet

個人 AI 幕僚系統 — 把工作拆成專才，按需載入，不浪費 context。

針對特定團隊的工作流程設計，每個幕僚只負責一個領域，問題進來自動路由給對的人。

**線上版：** https://cadmus-design.github.io/prompt-cabinet/monitor/

---

## 概念

不要讓 AI 當通才，要讓 AI 扮演專才。

把你的背景知識寫進 `agents/*.md`，需要哪個幕僚就載入哪個 prompt。Claude Code 讀檔即是載入 system prompt；瀏覽器版可接 Gemini / Claude / OpenAI API key 做語意路由。

```
你的問題
   ↓
幕僚長（intent 分析 + entity 抽取）
   ├── 資料問題  → 資料幕僚（SQL / Databricks）
   ├── 工單進度  → Jira 幕僚
   ├── 指標定義  → L2 描述幕僚
   ├── PR / 程式碼 → GitHub 幕僚
   ├── 頻道討論  → Slack 幕僚
   ├── GA4 / GSC  → Google 幕僚
   └── 跨域問題  → 序列派發，Step 間交接上下文
```

---

## 幕僚陣容

| 角色 | 負責 | MCP 工具 |
|------|------|---------|
| 大谷（幕僚長） | 任務分析、路由派發 | — |
| 安娜（資料幕僚） | Databricks SQL、AIGC/SEO metrics | `databricks-*` |
| 安海（L2描述） | Metric description 撰寫 | — |
| 達利恩（Finance） | 發票系統、Finance 資料 | `postgres-billing` |
| JJ（Jira） | DATAI 工單追蹤 | `claude_ai_Atlassian` |
| 窩賈（GitHub） | PR 狀態、程式碼搜尋 | `github` |
| 劉嘉玲（Slack） | 頻道訊息、決策脈絡搜尋 | `claude_ai_Slack` |
| Yumi（Google） | GA4 報表、Search Console | `google-analytics`, `gsc` |

---

## 使用方式

### 方式 A：Dispatch Center（線上 / 本地）

線上直接開：https://cadmus-design.github.io/prompt-cabinet/monitor/

本地開：
```bash
open monitor/index.html
```

**第一次使用：** 頁面頂部選擇 LLM provider（Gemini / Claude / OpenAI）→ 貼入 API key → 儲存。Key 存在瀏覽器 `localStorage`，不上傳任何伺服器。無 key 時自動 fallback 至 rule-based 路由。

輸入問題 → 幕僚長自動分析 → 顯示思考流 → 生成任務簡報 + Prompt → 複製貼到 Claude Code

Dispatch 引擎決策順序：
1. **LLM 語意路由**（有 API key）— Gemini / Claude / GPT-4o mini 理解問題意圖，決定派發對象與任務指示
2. **Rule-based fallback**（無 key）— Named Entity 抽取 → Intent 偵測 → 加權計分（entity ×2、intent ×1）

生成的 Prompt 包含幕僚長任務簡報：

```
請讀取 ~/di-agent/agents/jira.md

┌─ 幕僚長任務簡報
│  原始問題：DATAI-32 改完後要怎麼驗證？
│  意圖：驗證確認
│  實體：DATAI-32
│  協作順序：Step 1 / 2
│  建議工具：mcp__claude_ai_Atlassian__getJiraIssue
└─────────────────────────────────

以「Jira幕僚（JJ）」角色執行：
查詢 DATAI-32 目前狀態與待辦驗證步驟，整理後供 Step 2 繼續使用。
```

### 方式 B：直接在 Claude Code

```
請讀取 agents/jira.md，DATAI-32 目前進度？
請讀取 agents/data.md，以資料幕僚角色回答：AIGC 上個月 CTR 多少？
請讀取 agents/orchestrator.md，幫我分析這個問題應該找哪個幕僚：...
```

---

## 檔案結構

```
prompt-cabinet/
├── agents/
│   ├── orchestrator.md     # 幕僚長
│   ├── data.md             # 資料幕僚
│   ├── l2-description.md   # L2 描述幕僚
│   ├── finance.md          # Finance 幕僚
│   ├── jira.md             # Jira 幕僚
│   ├── github.md           # GitHub 幕僚
│   ├── slack.md            # Slack 幕僚
│   └── google.md           # Google 幕僚
├── knowledge/
│   └── team-context.md     # 跨幕僚共用背景知識
└── monitor/
    └── index.html          # HTML Dispatch Center
```

---

## 複製到你的團隊

1. Fork 這個 repo
2. 把 `agents/*.md` 裡的專業知識換成你的團隊背景
3. 把 `knowledge/team-context.md` 換成你的團隊架構
4. 在 `monitor/index.html` 的 `AGENTS` 物件更新關鍵字與路徑
5. 設定對應的 MCP 工具（Jira / Slack / DB…）

詳細步驟見 [GUIDE.md](./GUIDE.md)。

---

## 核心價值

這個系統最大的價值不是技術複雜度，而是**強迫你把隱性知識外化成文字**。
`agents/*.md` 不只是工具，是你把「每次都要重新解釋的背景」變成可版本控制的外部記憶體的過程。
