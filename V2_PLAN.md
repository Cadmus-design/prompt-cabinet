# Prompt Cabinet v2 計畫

> 目前 v1 定位：個人工具範本，路徑寫死，只能本機用。
> v2 目標：可以直接分享、deploy、別人打開就能用。

---

## 問題清單（v1 已知限制）

1. **路徑寫死** — prompt 裡的 `~/Documents/dev/personal/prompt-cabinet/agents/xxx.md` 是本機路徑，別人用就錯
2. **agent 知識寫在 .md 檔** — deploy 後無法讀本地檔案，需要把內容搬進來
3. **dispatch 仍是靜態** — 關鍵字和規則寫死在 HTML，新增幕僚要改程式碼
4. **沒有設定介面** — 別人複製後不知道怎麼客製化自己的幕僚

---

## v2 功能規劃

### P0 — 讓它可以 deploy

**方案：把 agent 內容嵌進 HTML（standalone 模式）**
- `agents/*.md` 的內容在 build 時注入 HTML
- 或：提供一個 `config.js` 讓使用者填入自己的 agent 內容
- prompt 生成時不再引用本地路徑，直接把 agent 內容貼進 prompt

**方案：Base path 設定**
- 打開 HTML 時有一個 settings panel
- 使用者填入自己的 base path（如 `~/dev/my-agent`）
- 存在 localStorage，下次打開自動帶入

### P1 — 可設定的幕僚管理

- Settings panel：可以新增/編輯/刪除幕僚
- 每個幕僚可設定：名稱、顏色、關鍵字、agent 檔案路徑或內容
- 設定存在 localStorage

### P1 — 更好的 dispatch

- 讓使用者可以自訂關鍵字（不用改程式碼）
- 考慮用 Claude API 做真正的語意 dispatch（需要 API key）
- 或：用更精準的 NLP 規則取代現在的關鍵字計分

### P2 — 多人共用版

- 設定可以 export/import（JSON）
- 支援分享設定檔給團隊成員
- 可以有 team preset（預設幕僚陣容）

---

## 技術方向

| 功能 | 方案 | 複雜度 |
|------|------|--------|
| Standalone deploy | agent 內容嵌入 HTML 或 config.js | 低 |
| Base path 設定 | localStorage + settings UI | 低 |
| 幕僚管理 UI | localStorage CRUD | 中 |
| 語意 dispatch | Claude API | 高（需 API key） |
| 設定 export/import | JSON file | 低 |

---

## 優先順序建議

1. **先做 base path 設定**（最小改動，讓別人可以用）
2. **再做 agent 內容嵌入**（真正 standalone）
3. **然後做幕僚管理 UI**（讓非工程師也能客製）
4. **最後考慮 Claude API dispatch**（需要評估 token 成本）

---

_建立日期：2026-04-01_
