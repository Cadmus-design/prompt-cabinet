# di-agent 完整設計指南

> 如何打造一套個人 AI 幕僚系統 — 原理、架構、串接細節

---

## 目錄

1. [這是什麼？](#1-這是什麼)
2. [核心原理](#2-核心原理)
3. [檔案架構詳解](#3-檔案架構詳解)
4. [Agent Prompt 設計](#4-agent-prompt-設計)
5. [Orchestrator 模式](#5-orchestrator-模式)
6. [知識庫（Knowledge Base）](#6-知識庫knowledge-base)
7. [HTML Dispatch Center 運作原理](#7-html-dispatch-center-運作原理)
8. [MCP 工具串接](#8-mcp-工具串接)
9. [如何複製給你的團隊](#9-如何複製給你的團隊)
10. [進階玩法](#10-進階玩法)

---

## 1. 這是什麼？

di-agent 是一個**個人化 AI 幕僚系統**，針對特定團隊的工作流程設計。

核心想法：**不要讓 AI 當通才，要讓 AI 扮演專才。**

一個資深工程師每天會遇到很多種不同類型的問題：
- 「這個 SQL 跑出來的數字對不對？」（資料問題）
- 「JIRA 上那個工單現在進度怎樣？」（工單問題）
- 「這個指標的定義要怎麼寫？」（文件問題）

如果每次都從零開始告訴 AI 背景，很累。這個系統的解法是：**預先把背景知識寫進 agent prompt，需要時直接載入對應的角色。**

### 不需要的東西

- ❌ API key（不呼叫 Anthropic API，直接用 Claude Code）
- ❌ 後端 server
- ❌ 資料庫
- ❌ 任何 framework

### 需要的東西

- ✅ Claude Code（CLI 或 IDE extension）
- ✅ 一些 `.md` 檔案
- ✅ 一個 `index.html`（選用，視覺化用）
- ✅ MCP 工具（視需求，如 Jira / Slack / Databricks）

---

## 2. 核心原理

### 2.1 Claude Code 讀檔等於載入 System Prompt

Claude Code 有一個關鍵能力：你可以叫它讀取檔案，然後「以那個角色」回答。

```
請讀取 agents/data.md，以資料幕僚角色回答：AIGC 上個月 CTR 多少？
```

這一句話做了三件事：
1. 讀取 `agents/data.md`（角色卡）
2. 把裡面的內容當作 context（等同於 system prompt）
3. 以那個角色的知識和工具回答問題

### 2.2 知識存在磁碟，不存在記憶體

傳統做法是把所有背景知識塞進對話開頭，每次都重複。

這個系統的做法：

```
di-agent/
├── agents/      ← 每個角色的專業知識
└── knowledge/   ← 跨角色共用的團隊背景
```

需要哪個幕僚，就載入哪個檔案。不需要時不佔 context window。

### 2.3 Dispatch 模式

問題進來 → 幕僚長判斷類型 → 派給對應的幕僚。

這是標準的 **Orchestrator-Worker** 模式：

```
用戶問題
    ↓
幕僚長（Orchestrator）
    ├── 資料問題 → 資料幕僚
    ├── 工單問題 → Jira幕僚
    ├── 指標定義 → L2描述幕僚
    ├── 跨域問題 → 多個幕僚同時
    └── ...
```

---

## 3. 檔案架構詳解

```
di-agent/
├── agents/
│   ├── orchestrator.md     # 幕僚長：問題分析 + dispatch 規則
│   ├── data.md             # 資料幕僚：DB/SQL/metrics 相關
│   ├── l2-description.md   # L2描述幕僚：指標定義撰寫
│   ├── finance.md          # Finance幕僚：發票系統相關
│   ├── jira.md             # Jira幕僚：工單追蹤
│   ├── github.md           # GitHub幕僚：PR/程式碼搜尋
│   ├── slack.md            # Slack幕僚：頻道訊息搜尋
│   └── google.md           # Google幕僚：GA4/GSC 報表
├── knowledge/
│   └── team-context.md     # 團隊架構、Q2目標、成員分工
├── monitor/
│   └── index.html          # 視覺化 Dispatch Center
└── README.md
```

### 為什麼這樣切割？

| 類型 | 放什麼 | 為什麼 |
|------|--------|--------|
| `agents/*.md` | 角色定義 + 工具清單 + 特定領域知識 | 每個幕僚只載入自己需要的 |
| `knowledge/team-context.md` | 跨幕僚共用的背景（團隊、產品線、目標） | 需要時可被多個幕僚引用 |

---

## 4. Agent Prompt 設計

每個 agent prompt（`agents/*.md`）包含三個核心區塊：

### 4.1 角色定義（Role Definition）

```markdown
## 角色定義
你是 [姓名] 的 [角色名稱]。你熟悉 [專業領域]，能 [具體能力]。
```

重點：**具體、有限制範圍**。不要寫「你是萬能的 AI 助手」，要寫「你是資料幕僚，你熟悉 Databricks SQL 和 AIGC metrics」。

### 4.2 背景知識（Domain Knowledge）

這是讓 AI 真正有用的關鍵。把你自己需要反覆說明的背景知識寫在這裡。

以資料幕僚為例：

```markdown
## 四層架構
- L1 Trusted Data: raw data 清洗後的 Silver layer
- L2 Defined Data: 定義清楚的 Gold table + semantic description
...

## 重要 Tables
- `aigc_daily_billing` — Gold table，每日 06:00 Asia/Taipei 更新
  - `data_count`: seo_urls_silver JOIN view_logs_silver
...
```

這樣問「aigc_daily_billing 什麼時候更新？」，幕僚直接知道答案，不用你解釋。

### 4.3 工具清單（Tools）

明確列出這個幕僚可以用哪些 MCP 工具：

```markdown
## 工具
- `mcp__databricks-prod__execute_sql_read_only` — 查 prod 資料
- `mcp__databricks-uat__execute_sql_read_only` — 查 uat 資料
- 預設用 read-only，除非明確需要寫入
```

### 4.4 Dispatch 建議（跨幕僚協作）

```markdown
## Dispatch 建議
- 如果問題涉及 Jira 工單 → 建議諮詢 Jira幕僚
- 如果問題涉及指標定義 → 建議諮詢 L2描述幕僚
```

這讓幕僚知道自己的邊界，遇到跨域問題時會主動建議。

---

## 5. Orchestrator 模式

### Orchestrator 做什麼？

```markdown
# orchestrator.md 的核心邏輯

## Dispatch 規則
| 問題類型 | 派給 |
|---------|------|
| Databricks SQL、metrics 查詢 | 資料幕僚 |
| 寫 metric description | L2描述幕僚 |
| Finance 發票、Finance 資料 | Finance幕僚 |
| Jira 工單、任務進度 | Jira幕僚 |
| 跨領域問題 | 幕僚長自行整合多個觀點 |

## 回覆格式
[幕僚長] 判斷：這個問題屬於「XX類型」，派給 → XX幕僚
```

### 兩種使用方式

**方式 A：先問幕僚長再切換**
```
請讀取 agents/orchestrator.md，幫我分析：DATAI-32 改完後要怎麼驗證？
→ 幕僚長回答：「這個問題跨 Jira + 資料幕僚，建議先看工單狀態再查數字」
→ 你再個別載入對應幕僚
```

**方式 B：知道類型直接載入**
```
請讀取 agents/jira.md，DATAI-32 目前進度？
```

---

## 6. 知識庫（Knowledge Base）

`knowledge/team-context.md` 放**跨幕僚共用**的背景知識：

- 團隊架構（誰負責什麼）
- Q2 目標（為什麼在做這些事）
- 產品線說明（AIGC / Multi CDN / C2G）
- 監控工具（Grafana / Databricks / Jira）

### 使用方式

需要全局背景時，可以同時載入：

```
請讀取 knowledge/team-context.md 和 agents/data.md，幫我規劃 SEO pipeline 優先順序
```

### 知識庫更新原則

- **時效性高的**（工單狀態、PR 進度）→ 放進個別 agent prompt，方便針對性更新
- **相對穩定的**（團隊架構、產品線定義）→ 放在 team-context.md
- **動態資料**（當下的數字）→ 直接用 MCP 工具查，不寫死在檔案裡

---

## 7. HTML Dispatch Center 運作原理

`monitor/index.html` 是一個純前端、零依賴的視覺化介面。

### 架構

```
用戶輸入問題
     ↓
detectAgents() — 關鍵字比對，決定派給哪個幕僚
     ↓
Canvas 動畫 — 顯示幕僚工作狀態（思考/執行/完成）
     ↓
生成 Prompt — 自動組出「請讀取 agents/xxx.md，以角色回答：...」
     ↓
複製按鈕 — 貼到 Claude Code 執行
```

### Dispatch 邏輯

```javascript
// 每個 agent 有關鍵字列表
const AGENTS = {
  data: {
    keywords: ['sql','databricks','數據','aigc','ctr','pipeline',...],
    buildPrompt: q => '請讀取 ~/di-agent/agents/data.md\n以「資料幕僚」角色回答：' + q
  },
  jira: {
    keywords: ['jira','datai','工單','ticket','進度','deploy',...],
    buildPrompt: q => '請讀取 ~/di-agent/agents/jira.md\n以「Jira幕僚」角色回答：' + q
  },
  ...
}

// 計分：看問題裡有幾個關鍵字命中
function detectAgents(q) {
  const scored = Object.entries(AGENTS)
    .map(([k,a]) => ({ key:k, score: a.keywords.filter(w=>q.includes(w)).length }))
    .filter(x => x.score > 0)
    .sort((a,b) => b.score - a.score);

  // 如果第二名也有命中，同時派兩個幕僚（跨域問題）
  if (scored.length >= 2 && scored[1].score >= 1)
    return [scored[0].key, scored[1].key];
  return [scored[0].key];
}
```

### Canvas 動畫

8個角色在 Canvas 上各自有桌子和像素人形，狀態分四種：

| 狀態 | 動畫 | 顏色 |
|------|------|------|
| idle（待命） | 緩慢上下飄動 | 暗色 |
| thinking（分析中） | 思考泡泡 + 黃色閃爍 | 黃色 |
| working（執行中） | 打字動作 | 角色主色 |
| done（完成） | 跳躍 + 金色閃光 | 綠色 |

像素人形圖片從開源 [pixel-agents](https://github.com/pablodelucca/pixel-agents) 載入，失敗時用純矩形作 fallback。

### 重點設計：Prompt 生成

HTML 不「執行」任何 AI 功能。它只做一件事：**把問題包裝成格式化的 Prompt，讓你複製貼到 Claude Code**。

```javascript
buildPrompt: q => '請讀取 ~/di-agent/agents/data.md\n以「資料幕僚」角色回答：' + q
```

這個設計非常輕量，不需要 API key，不需要後端。

---

## 8. MCP 工具串接

這是讓幕僚真正有「行動能力」的關鍵。

### MCP 是什麼？

MCP（Model Context Protocol）讓 Claude 能呼叫外部 API。設定好後，Claude Code 就能直接查 Jira、讀 Slack、執行 Databricks SQL。

### 本系統用到的 MCP 工具

| 幕僚 | MCP Server | 主要工具 |
|------|-----------|---------|
| 資料幕僚 | databricks-prod/uat/sit | `execute_sql_read_only` |
| Jira幕僚 | claude_ai_Atlassian | `searchJiraIssuesUsingJql`, `getJiraIssue` |
| Slack幕僚 | claude_ai_Slack | `slack_search_public_and_private`, `slack_read_thread` |
| GitHub幕僚 | github | `github_list_pull_requests`, `github_search_code` |
| Google幕僚 | google-analytics, gsc | `run_report`, `search_analytics` |
| Finance幕僚 | postgres-billing | `query` |

### MCP 設定方式

在 `~/.claude/.mcp.json` 或專案層級 `.mcp.json` 設定：

```json
{
  "mcpServers": {
    "databricks-prod": {
      "command": "/path/to/node",
      "args": ["/path/to/databricks-mcp/index.js"],
      "env": {
        "DATABRICKS_HOST": "https://your-workspace.azuredatabricks.net",
        "DATABRICKS_TOKEN": "dapi..."
      }
    },
    "jira": {
      "command": "/path/to/node",
      "args": ["/path/to/jira-mcp/index.js"],
      "env": {
        "JIRA_URL": "https://your-org.atlassian.net",
        "JIRA_TOKEN": "..."
      }
    }
  }
}
```

### Agent Prompt 裡的工具聲明

在每個 agent prompt 裡明確寫出工具名稱，有兩個效果：
1. Claude 知道可以用哪些工具（不會亂用）
2. 有記錄，方便維護

```markdown
## 工具
- `mcp__databricks-prod__execute_sql_read_only` — 正式環境，謹慎查詢
- `mcp__databricks-uat__execute_sql_read_only` — 測試環境，正常使用
```

---

## 9. 如何複製給你的團隊

### Step 1：定義你的幕僚陣容

問自己：**我每天處理的問題有哪幾種類型？**

常見類型：

| 類型 | 幕僚角色 |
|------|---------|
| 資料查詢 / SQL | 資料幕僚 |
| 工單 / 任務追蹤 | Jira幕僚 |
| 文件 / 規格撰寫 | 文件幕僚 |
| 程式碼搜尋 / PR review | GitHub幕僚 |
| 客戶溝通 / Slack | Slack幕僚 |
| 產品指標 / GA | 分析幕僚 |

### Step 2：建立資料夾

```bash
mkdir -p my-agent/{agents,knowledge,monitor}
```

### Step 3：寫 team-context.md

```markdown
# Team Context

## 我的角色
[你在團隊裡負責什麼]

## 團隊成員
[誰負責什麼，你常需要配合誰]

## 主要工作系統
[Jira project name、Databricks workspace、GitHub org 等]

## 產品線 / 業務背景
[你的產品是什麼，核心指標是什麼]
```

### Step 4：寫第一個 Agent Prompt

以「資料幕僚」為範本：

```markdown
# [角色名稱] — [Agent 名稱]

## 角色定義
你是 [你的名字] 的 [角色]。你熟悉 [系統/工具/領域]，能 [具體幫忙做什麼]。

## 背景知識

### [知識分類 1，如：資料架構]
[具體描述，越詳細越好]

### [知識分類 2，如：重要 Tables]
[具體描述]

## 工具
- `mcp__xxx__tool_name` — [用途說明]

## Dispatch 建議
- 如果問題涉及 [X] → 建議諮詢 [Y幕僚]
```

**寫作訣竅**：
- 把你自己平時需要「解釋背景」的話，全部寫進去
- 越具體越好：表名、欄位名、時間頻率、業務含義
- Caveat（限制、注意事項）要特別寫清楚

### Step 5：寫 Orchestrator

```markdown
# 幕僚長

## 我的工作背景
[直接複製 team-context.md 的重點]

## Dispatch 規則
| 問題類型 | 派給 |
|---------|------|
| [你的問題類型] | [對應幕僚] |

## 回覆格式
[幕僚長] 判斷：這個問題屬於「XX」，派給 → XX幕僚
```

### Step 6：更新 HTML（選用）

打開 `monitor/index.html`，修改：

1. **CREW 陣列**（第 155 行附近）— 改成你的幕僚名字和顏色
2. **AGENTS 物件**（第 297 行附近）— 改成你的幕僚和關鍵字
3. **chips**（HTML 裡的 `<span class="chip">`）— 改成你常用的 quick questions

### Step 7：測試

```bash
# 方式 A：直接在 Claude Code 輸入
請讀取 agents/data.md，以資料幕僚角色回答：[你的問題]

# 方式 B：打開 HTML
open monitor/index.html
```

---

## 10. 進階玩法

### 10.1 幕僚長自動化（Claude Code slash command）

建立 `~/.claude/commands/ask.md`：

```markdown
---
description: 問幕僚長
argument-hint: 你的問題
---

請讀取 ~/my-agent/agents/orchestrator.md
問題：$ARGUMENTS
```

之後只要：
```
/ask DATAI-32 現在進度？
```

### 10.2 多幕僚串聯

跨域問題可以明確串聯：

```
請依序：
1. 讀取 agents/jira.md，查 DATAI-32 的待辦步驟
2. 讀取 agents/data.md，驗證 subpage_count 計算是否正確
3. 整合兩個幕僚的回答，給我行動建議
```

### 10.3 知識庫定期更新

設計一個簡單的更新 SOP：

- 每次工單完成 → 更新 `agents/jira.md` 裡的工單狀態
- 每次有新的 table 或 schema 變更 → 更新 `agents/data.md`
- 每季 OKR 更新 → 更新 `knowledge/team-context.md`

### 10.4 拆分知識深度

當一個 agent prompt 太長時，可以進一步拆分：

```
agents/
├── data.md              # 資料幕僚（主要角色卡）
├── data-aigc.md         # AIGC 子專題（更深的知識）
└── data-bulldozer.md    # Bulldozer 子專題
```

使用時根據問題選擇：

```
請讀取 agents/data.md 和 agents/data-aigc.md，回答：...
```

---

## 設計哲學總結

| 原則 | 做法 |
|------|------|
| 知識外化 | 把你腦中的背景知識寫成 .md 檔 |
| 按需載入 | 不同問題載入不同幕僚，不浪費 context |
| 工具聲明 | 在 prompt 裡明確列出 MCP 工具 |
| 邊界清晰 | 每個幕僚只做一件事，知道何時轉手 |
| 持續更新 | 工作進度放在 agent prompt 裡，不是腦子裡 |

這個系統最大的價值不是技術複雜度（它真的不複雜），而是**強迫你把隱性知識（tacit knowledge）外化成文字**。這個過程本身就讓你更清楚自己在做什麼。
