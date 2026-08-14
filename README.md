# AI 開發工作流架構設計 — Josh Lo

## English Executive Summary

This portfolio documents anonymized architecture patterns from an AI-native engineering workflow that I helped build and operate for a B2C backend team at a travel e-commerce platform.

As of July 2026, the internal toolkit contained **16+ Claude Code skills and approximately 5,500 lines of workflow definitions and scripts**. I designed and implemented **12 development workflow skills** spanning Git and pull-request operations, release management, testing and verification, document parsing, multi-agent design review, and guarded SIT deployment. The wider toolkit integrated 10 categories of engineering systems, including four MCP servers, with explicit fallback paths.

My work focused on making agentic workflows reliable enough for team use:

- separating LLM reasoning from deterministic authentication, retry, rate-limit, and validation code;
- building evidence-based multi-agent reviews with dependency gates and file-and-line anchors;
- reducing repeated context consumption through document fingerprints and diff-only re-reading;
- enforcing deny-by-default deployment and credential guardrails outside the model;
- governing layered user instructions, project memory, and repository-backed skills across sessions;
- turning failures into versioned rules, onboarding material, and repeatable team workflows.

During a 30-day measurement window in June-July 2026, the team's automated code-review loop recorded **41 executions**. This is evidence of workflow adoption, not a claim that 41 defects were found or that quality improved by a specific percentage. I operated and governed that shared workflow; its core implementation was designed by a teammate, as stated in the corresponding case study.

The production tools and company-specific code are not public. Every document and diagram in this repository was independently rewritten and anonymized to present the design decisions, trade-offs, controls, and operating lessons without exposing internal code, domains, system identifiers, or business data.

## 中文說明

這個 repo 收錄我在任職公司（旅遊電商平台 B2C 後端團隊）建置與營運 **AI 開發工作流**的架構設計文件。

截至 2026 年 7 月，工具本體包含 **16+ 個 Claude Code skills、約 5,500 行 workflow 定義與腳本**，屬公司內部 repo，不對外公開。
本 repo 的所有文件與圖均為**重新撰寫的去識別化版本**：聚焦設計決策與取捨，不含任何內部程式碼、
網域、系統識別資訊或商業內容。

## 背景

團隊以 Claude Code 為核心，把開發流程中重複性高的環節做成 skills（宣告式的工作流定義）：
PR review、系統設計文件審查、部署、線上 log／指標查詢、需求文件解析等，
整合了 ELK、Prometheus/Grafana、CI/CD、GitHub、Jira/Confluence、Slack、Google Drive 等
10 類系統（含 4 個 MCP server，皆設計 fallback／降級路徑）。

我在其中的角色：

- 主導 **12 個開發流程 skills 的設計與實作**，涵蓋 Git／PR、release、測試驗證、文件解析、multi-agent review 與安全部署
- 自動化 code review 迴圈（團隊共同工具）的**日常營運與治理**：2026 年 6–7 月的一個 30 日統計區間內實際執行 41 次
- 建立 user instructions、project memory 與 repository-backed skills 的分層治理與升級規則
- 將個人工作流制度化為**團隊 onboarding 文件**，推廣至團隊採用

## 文件目錄

| 文件 | 主題 | 我的角色 |
|---|---|---|
| [01 — Multi-agent 文件審查框架](docs/01-multi-agent-review.md) | N+2 agent 依賴圖派工、file:line 證據鏈 | 設計與實作 |
| [02 — Token 經濟的文件解析管線](docs/02-doc-parsing-pipeline.md) | docx 解析、內容指紋、diff-only re-read | 設計與實作 |
| [03 — AI 自動化安全護欄](docs/03-ai-guardrails.md) | 環境白名單、憑證治理、production 查詢安全 | 設計與實作（查詢治理部分為團隊工具，註明於文內） |
| [04 — Slack 驅動的 Code Review 迴圈](docs/04-code-review-loop.md) | 六階段流程、增量 re-review、雙層驗證 | 營運與治理（工具本體為團隊同事設計，註明於文內） |
| [05 — AI Engineering Memory & Skill Governance](docs/05-memory-skill-governance.md) | 分層記憶、failure-to-rule、skill 版控與 stale-memory 治理 | 工作脈絡與治理設計（memory engine 為 Claude Code 平台能力） |

## 跨所有工具的設計原則

1. **LLM 與 deterministic code 的職責分界**——LLM 負責語意轉換（自然語言 → 查詢、diff → 評論），
   認證、session、限流、重試一律交給腳本。模型不該碰它不擅長的事。
2. **驗證優於宣稱**——自動化流程不以「指令送出成功」收尾：部署後撈線上版本比對、
   review 後逐條核銷、每個宣稱都要有可檢查的錨點。
3. **Token 經濟**——上下文是稀缺資源：progressive disclosure（主文件精簡、細節外置）、
   diff-only re-read、metadata 一次抓齊複用。
4. **安全護欄預設拒絕**——權限邊界寫死在工具層（regex 白名單、唯讀沙箱），不依賴模型自律，
   也不可被對話說服繞過。
5. **失敗模式固化**——每個 skill 維護 Common Mistakes 清單，把 LLM 的實際失誤
   （語意誤判、格式鏡像模仿）沉澱成防呆規則；錯誤 → 記憶 → 規則回寫，形成學習迴圈。
6. **記憶有生命週期**——短期 context、project memory、versioned skill 與 deterministic gate 分層管理；
   過期或衝突的記憶需要淘汰，不能因為被保存就視為真相。

---

*文件皆為個人重新撰寫，僅描述通用架構模式；如對實作細節有興趣，歡迎面談時交流。*
