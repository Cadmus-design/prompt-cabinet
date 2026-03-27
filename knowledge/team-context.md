# Team Context — Data & Innovation Team

## 團隊使命
把資料集中、定義清楚、知識結構化給 AI 用。
解決三個根本問題：資料散落、知識流失、重複勞動。

## 四層架構
| 層 | AI 獲得的能力 | 一句話 |
|----|-------------|--------|
| L1 Trusted Data | 看得到 | 資料存在、乾淨、可信 |
| L2 Defined Data | 會用 | 知道指標怎麼算、什麼意思 |
| L3 Business Context | 會想 | 懂 business logic、能推理 |
| L4 AI Agents | 會做 | 包成 agent + interface，讓人直接用 |

## 團隊成員與分工
| 人 | 負責 | Q2 重點 |
|----|------|--------|
| **Bill** | L1+L2 主力 | Bulldozer data quality、L2 description review |
| **Tim** | L3+L4 設計 + 方向判斷 | L3/L4 framework 設計、AIGC quick win 文件 |
| **An** | L3+L4 執行 | AIGC quick win (Genie)、Finance discovery |
| **Cadmus** | L1+L2 支援 | Finance 發票系統、SEO/AEO pipeline、L2 description 初版 |

## Q2 目標產品
1. **AI Agent**: 自動化已知流程（Peter月報 2hr→2min）
2. **AI Interface**: 讓人自助探索資料（GTM 自己問 AI）

## Q2 各人任務

### Cadmus Q2 分配
1. **Finance 自動發票系統接手**（優先）
   - 穩定接手，不急著加功能
2. **SEO/AEO 數據準備**（階梯式）
   - Step 1: 確認客戶 GA/GTM proper setup
   - Step 2: Google 後台接 GSC 到 GA
   - Step 3: 資料流進 Databricks
   - Step 4: metric view 初版
3. **L2 description 維護**
   - AIGC metric description 初版（Bill review）
   - Bulldozer metric description 初版

## 公司三條產品線
| 產品線 | 說明 |
|--------|------|
| AIGC | AI 內容生成，GTM 團隊主推 |
| Multi CDN (Bulldozer) | CDN 流量管理 |
| C2G | Cloud to Ground 服務 |

## 監控工具
| 工具 | 用途 |
|------|------|
| Grafana (prod.grafana.mlyxs.pro) | 服務監控 |
| Databricks | 資料平台 |
| Jira (DATAI project) | 工單追蹤 |
