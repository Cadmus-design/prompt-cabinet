# di-agent — D&I Agent Dispatch System

Cadmus 的個人 AI 幕僚系統，針對 Mlytics Data & Innovation Team 工作流程設計。

## 架構

```
di-agent/
├── agents/              # 每個 agent 的 system prompt
│   ├── orchestrator.md  # 幕僚長：任務分析 + dispatch 規則
│   ├── data.md          # 資料幕僚：Databricks、AIGC、SEO metrics
│   ├── l2-description.md# L2描述幕僚：metric description 撰寫
│   ├── finance.md       # Finance幕僚：發票系統
│   └── jira.md          # Jira幕僚：DATAI 工單追蹤
├── knowledge/
│   └── team-context.md  # 團隊架構、成員分工、Q2 目標
└── monitor/
    └── index.html       # HTML Dispatch Center（用瀏覽器開）
```

## 使用方式

### 方式 A：HTML Monitor（視覺化）
```
open monitor/index.html
```
輸入問題 → 自動 dispatch → 複製 Prompt → 貼到 Claude Code

### 方式 B：直接在 Claude Code 使用
告訴 Claude Code 載入對應的 agent prompt：

```
請讀取 agents/data.md，以資料幕僚角色回答：AIGC 上個月 CTR 多少？
請讀取 agents/jira.md，以 Jira 幕僚角色回答：DATAI-32 現在狀態？
請讀取 agents/orchestrator.md，幫我分析這個問題應該找哪個幕僚：...
```

## Agents

| Agent | 負責 | 觸發關鍵字 |
|-------|------|----------|
| 幕僚長 | 任務路由 | 所有問題先過這裡 |
| 資料幕僚 | Databricks SQL、AIGC/SEO metrics | 數據、查詢、SQL、pipeline |
| L2描述幕僚 | Metric description 撰寫 | description、定義、指標、caveat |
| Finance幕僚 | 發票系統、Finance 資料 | finance、發票、invoice |
| Jira幕僚 | DATAI 工單、進度追蹤 | jira、DATAI、工單、PR |
