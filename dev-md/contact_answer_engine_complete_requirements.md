# Contact Answer & Contact Engine — 完整產品與工程需求規格

> **文件類型：** Product Requirements + System Architecture + Engineering Specification  
> **產品負責方：** Starman Inc.  
> **前台產品：** Contact Answer — `answer.contact`  
> **智能基礎設施／開發者 API：** Contact Engine — `engine.contact`  
> **版本：** 1.0（整合版，2026-10-20）  
> **文件狀態：** 建議作為 MVP 工程實作的單一規格來源；價格、贈點與成本門檻由服務端設定，不得散落硬編碼於前端。  
> **讀者：** Product、Frontend、Backend、AI/ML、Data、Security、QA、Finance/Operations。

---

## 文件使用方式與規格優先級

本文件把現有 `answer_md_old.html`、`answer_md(1).txt`、`documentation_api.html`、`engine.html` 及 Answer Room 介面原型整合成一套可實作規格。現有原型用於理解產品 UX 與既有路徑，不代表示範資料、假資料或當前前端邏輯可以直接作為後端真實狀態。

**規格衝突採以下優先級：**

1. 本文件「產品決策與舊規格調和」章節明確列出的最終決策。
2. 最新補充 `answer_md(1).txt` 的 Consumer / Engine API 價格、Free 使用額度、Room 背景 AI 政策與 Drive 參考資料政策。
3. 最近確認的 Answer Settings：Follow-up Suggestions、Response Style、Response Format，以及 Answer 專屬 General / Billing / Account 設定。
4. `answer_md_old.html` 中仍與以上內容一致的架構、Credit Ledger、Reservation、資料模型與安全要求。
5. `documentation_api.html` 和 `engine.html` 所示現有 API 路徑、品牌與開發者文件結構。
6. 前端原型僅作為目前互動和 UI 結構參考；不得把前端 local state 當作帳務或權限依據。

文內的 **MUST / 必須** 是驗收要求；**SHOULD / 建議** 是一般預設；**MAY / 可選** 是可依產品測試與運行數據採用的能力。所有價格、模型 ID、成本比率、路由權重、額度、逾時與限流值都必須可從伺服器端設定中心調整。

## 目錄

1. [Executive Summary](#1-executive-summary)
2. [產品定位與成功原則](#2-產品定位與成功原則)
3. [產品範圍與非目標](#3-產品範圍與非目標)
4. [產品與服務邊界](#4-產品與服務邊界)
5. [參考系統架構](#5-參考系統架構)
6. [核心使用者流程](#6-核心使用者流程)
7. [Answer 深度與 Credit 預估 UX](#7-answer-深度與-credit-預估-ux)
8. [Follow-up Suggestions：點選式互動](#8-follow-up-suggestions點選式互動)
9. [Answer Room 規格](#9-answer-room-規格)
10. [Contact Drive 共同參考資料](#10-contact-drive-共同參考資料)
11. [Contact Engine 核心模組](#11-contact-engine-核心模組)
12. [智慧路由與成本最佳化演算法](#12-智慧路由與成本最佳化演算法)
13. [多模型驗證與反駁工作流](#13-多模型驗證與反駁工作流)
14. [Research、證據與引用](#14-research證據與引用)
15. [最終答案、透明度與回饋](#15-最終答案透明度與回饋)
16. [API 介面與請求契約](#16-api-介面與請求契約)
17. [狀態機、資料模型與事件](#17-狀態機資料模型與事件)
18. [Credit 計費總體設計](#18-credit-計費總體設計)
19. [Contact Answer Consumer 定價](#19-contact-answer-consumer-定價)
20. [Contact Engine API 定價](#20-contact-engine-api-定價)
21. [免費額度與 Contact 訂閱權益](#21-免費額度與-contact-訂閱權益)
22. [Reservation、Settlement 與退款](#22-reservationsettlement-與退款)
23. [Auto-recharge 與支付整合](#23-auto-recharge-與支付整合)
24. [Answer 專屬 Settings](#24-answer-專屬-settings)
25. [資料庫設計建議](#25-資料庫設計建議)
26. [安全、隱私與多租戶](#26-安全隱私與多租戶)
27. [可靠性、延遲與限流](#27-可靠性延遲與限流)
28. [Observability 與商業監測](#28-observability-與商業監測)
29. [Feedback、Verified Corpus 與長期護城河](#29-feedbackverified-corpus-與長期護城河)
30. [管理設定與功能開關](#30-管理設定與功能開關)
31. [測試與驗收標準](#31-測試與驗收標準)
32. [分階段交付計畫](#32-分階段交付計畫)
33. [產品決策與舊規格調和](#33-產品決策與舊規格調和)
34. [名詞表](#34-名詞表)

---

## 1. Executive Summary

### 1.1 Contact Answer 是什麼

Contact Answer 是面向用戶的智能答案產品，不應只是把同一個問題依次送給數個模型再把答案拼在一起。它的核心價值是：根據問題的性質與風險，選擇合適的模型和證據工具，讓模型獨立分析、互相 challenge、辨識矛盾、按需查核來源，最後合成一個直接、可理解、保留必要不確定性的答案。

產品定位可概括為：**The best AI isn’t always the best answer. Contact Answer finds the most reliable one.**

### 1.2 Contact Engine 是什麼

Contact Engine 是支撐 Answer 與未來 Contact AI 生態產品的模型中立智能基礎設施，不是第四個 AI 模型。它負責：

- 問題正規化、任務分類、複雜度與風險評估。
- 以成本、能力、品質、延遲、可用性與信心為條件，選取模型及工具。
- 動態分配 token、模型呼叫、搜尋、證據查核及 challenge 的預算。
- 編排 OpenAI、Anthropic Claude、Google Gemini 等 provider 的獨立分析。
- 比較候選答案、找出實質矛盾、查核重要主張並整合證據。
- 回傳一份最終答案、來源與有限而誠實的驗證摘要。
- 為每個付費工作建立可審計的 quote、reservation、usage、settlement 與 cost record。
- 透過真實成本、品質、錯誤與用戶回饋持續改善路由。

### 1.3 產品核心流程

```text
User / Room Discussion
        ↓
Low-cost Preflight (classify, assess complexity/risk)
        ↓
Recommend Verified / Deep / Research
        ↓
Estimate Credits + show quote
        ↓ explicit user confirmation / authorized API budget
Reserve Credits
        ↓
Independent Model Analysis
        ↓
Cross-model Challenge + Claim/Contradiction Analysis
        ↓
Evidence Retrieval / Verification when justified
        ↓
Adaptive additional computation within authorized budget
        ↓
Final Synthesis + citations + uncertainty
        ↓
Credit Settlement + cost/quality telemetry
        ↓
Final Answer + relevant click-to-continue suggestions
```

### 1.4 不可妥協的工程原則

1. **不因模型數量多就假設答案正確。** 多模型一致也可能一起錯；答案必須呈現實質不確定性。
2. **付費工作先報價、再確認、再執行。** 不可在用戶確認前啟動完整付費推理，也不可默默超收 Credits。
3. **一般 Room 討論免費。** 背景 AI 介入判斷不從用戶 Credits 扣款，但必須受內部預算、批次、限流與反濫用機制約束。
4. **用戶只需選深度，不需懂模型或 token。** Engine 自動決定實際模型、呼叫與可接受成本。
5. **Follow-up 優先點選而非打字。** 使用者仍可隨時自由輸入，不可把自由輸入鎖死。
6. **Consumer Credits 與 Engine API Credits 分帳。** 不得交換、混算或套用同一套兌換率。
7. **Credit Ledger 是帳務真相。** 只改餘額欄位、依賴前端計數或直接信任支付 webhook payload 均不合格。
8. **成本、品質、延遲要一起優化。** Engine 不以單純減少模型呼叫作為成功，也不以多呼叫來假裝更可靠。
9. **模型中立。** 增加或替換模型 provider 不應要求重寫 Answer UI、計費規則或公開 API。
10. **從第一天收集可用但最少化的觀測數據。** 不得把每次未經驗證的答案自動當成真實訓練資料。

---

## 2. 產品定位與成功原則

### 2.1 Contact Answer 的差異化

一般聊天產品著重於回答問題；Contact Answer 要把「取得較可靠答案」做成一個流程：

- **Independent analysis：** 初始分析盡可能獨立，避免一個模型的觀點過早污染其他模型。
- **Challenge：** 每個候選答案都需接受對立檢查，而不是只做文風合併。
- **Contradiction detection：** 區分語句差異與真正的事實／推論衝突。
- **Evidence checks：** 對需要時效性、可追溯來源或高正確性的重要主張查證。
- **Synthesis：** 對證據充分的結論做整合；證據不足時保留差異，不強行取得共識。
- **Adaptive compute：** 額外算力只投放在能實質提升品質的地方。
- **Low-friction interaction：** Follow-up 選項盡量預測用戶下一步意圖，讓用戶點一下就能繼續。
- **Room collaboration：** 人與人先交流，Contact AI 在有價值時提供協助，正式答案再依深度明確確認與扣點。

### 2.2 商業模型

- Contact Answer 是消費者使用的付費答案產品，以預購 Consumer Credits 為主，而不是每生成一個答案就建立一筆小額支付交易。
- Contact Engine API 是面向開發者的 Intelligence Infrastructure，以獨立 API Credits、API keys、rate limits、usage records 和 API 定價方案運作。
- Engine 的利潤來源應是高效率路由、可靠驗證、Research、Synthesis 與可控的單位經濟，而不是無差別地呼叫最貴模型。
- Contact 訂閱方案可以提供 Answer Credits 作為 AI Draft／生態產品權益，但不得再額外創建一個與現有 Contact 訂閱重疊的 Answer seat plan。
- 組織可使用共享 Credit balance；MVP 不要求以人頭／seat 收費。

### 2.3 成功指標

產品上線後至少追蹤：

- 首次提問至取得答案的完成率、延遲、放棄率。
- Quote 展示後的確認率、深度選擇分布、Credit 首購轉換率。
- Follow-up suggestion 顯示率、點選率、後續答案完成率、每次點選節省的手動輸入比例。
- Verified / Deep / Research 使用比例、重試比例、每個答案的平均 Credits 和平均 provider COGS。
- 每個付費答案的 contribution margin、不同 Credit pack 的有效收入／Credit。
- Provider 成功率、fallback 比例、模型間分歧率、查核來源比例、可追溯主張比例。
- 用戶有用性評分、錯誤回報、修正率、引用來源的有效性。
- 免費用戶月回訪、免費額度消耗、免費轉付費率、每位免費用戶的實際 API 成本。

單一成功指標不可以只看「多模型使用率」或「回答速度」；需要同時看品質、成本與用戶價值。

---

## 3. 產品範圍與非目標

### 3.1 MVP 必須包含

- Answer standalone 對話介面和 Answer history。
- Verified、Deep、Research 三種用戶可理解的深度控制。
- 低成本問題預估、深度建議、Credit quote、執行前確認。
- OpenAI、Claude、Gemini 三個 provider 的統一抽象與可用路由。
- 獨立分析、cross-model challenge、矛盾辨識、按需 evidence checking、final synthesis。
- Credit ledger、reservation / settlement / release、過期、FEFO、付款、Credit pack、Auto-recharge。
- Follow-up Suggestions、Response Style、Response Format Preferences。
- Answer Room 的人類訊息、AI 背景介入判斷、正式 Get an Answer 付款流程。
- Room payer 選擇（Room owner / requester）及相應記帳。
- Room Drive 參考文件授權、來源引用與撤回權限。
- Settings：General、Preferences、Billing、Account。
- Engine API 認證、API operation、用量記錄、限流、錯誤碼與開發者文件。
- 基本 observability、路由成本分析、provider fallback、安全與反濫用控制。

### 3.2 MVP 不應建置

- 不應對用戶暴露原始模型選擇、token 數、內部 routing score 或每一步呼叫細節作為主要操作。
- 不應為每個答案固定跑同一套昂貴模型／固定輪數，無視問題難度。
- 不應預設向用戶展示 1、2、3、5、8、10、20、34、50 等一系列原始算力級別供自行研究。
- 不應在每次普通 Room 訊息後生成完整付費答案。
- 不應依賴模型「同意票數」直接判定真實性。
- 不應自動產生未經審核的公開「Verified Answer 百分比正確率」。
- 不應用一個不可審計的 mutable balance 代替 Credit ledger。
- 不應將整個 Drive 分享給 Room；只允許具體文件及授權範圍。
- 不應建立獨立 Answer seat-based team plan。
- 不應在前端持有 AI provider secrets、Payment secret、Engine service secret。
- 不應把用戶的私有 Answer、Room 或 Drive 文檔默認用於訓練自有模型。

---

## 4. 產品與服務邊界

| 層 | 名稱／入口 | 職責 | 不負責的事 |
|---|---|---|---|
| 用戶產品 | `answer.contact` | Standalone Answer UX、History、Room、Settings、Quote 顯示、確認互動 | 不直接呼叫 OpenAI／Claude／Gemini，不決定最終扣點 |
| Answer application backend | Contact Answer backend | 身份、使用者／Room 權限、對話資料、產品設定、計費互動、前端 API | 不將業務路由邏輯寫死在前端 |
| 智能基礎設施 | `engine.contact` / Contact Engine | Classification、routing、budgeting、provider orchestration、verification、research、synthesis、usage measurement | 不是第四個模型，不擁有消費者付款 UI |
| Provider adapters | OpenAI / Claude / Gemini adapters | 統一封裝 provider API、模型能力、usage、timeout、retry、error mapping | 不自行執行產品計費或修改用戶餘額 |
| Billing & Credit service | Consumer Credit Ledger / Engine API Ledger | quote、reservation、settlement、pack、subscription grant、expiry、refund、auto-recharge | 不允許 AI workflow 直接修改帳務表 |
| Evidence / tools layer | Search、source retriever、Drive connector、document parsing | 提供可追溯的來源與文件片段 | 不把工具輸出當成無條件可信指令 |
| Learning & quality layer | Eval、Feedback、Verified Corpus、Routing Analytics | 由運行數據改善路由與品質 | 不默認拿私有內容訓練模型 |

**架構建議：** 第一版使用「模組化單體 + 背景工作佇列」即可，先以明確模組界線分開 Engine、billing、provider adapters、Room、document retrieval。當流量、隔離需求或部署週期證明必要時，再把高負載模組拆為獨立服務。不要在尚無營運數據時先引入過度複雜的微服務網絡。

---

## 5. 參考系統架構

```text
┌───────────────────────────────────────────────────────────────┐
│ answer.contact — Web UI                                      │
│ Answer composer / History / Answer result / Follow-up choices │
│ Room chat / Drive reference panel / Settings                 │
└──────────────────────────────┬────────────────────────────────┘
                               │ Session-authenticated App API
┌──────────────────────────────▼────────────────────────────────┐
│ Contact Answer Application Backend                             │
│ Identity + RBAC + Conversations + Room + User Preferences      │
│ Quote UI contract + Billing orchestration + Attachments        │
└──────────────┬──────────────────────┬─────────────────────────┘
               │                      │
               │                      └── Consumer Credit Service
               │                          Ledger / Reservation / Settlement
               ▼
┌───────────────────────────────────────────────────────────────┐
│ Contact Engine Core — engine.contact                           │
│ API Gateway/Auth → Normalize → Classify → Complexity/Risk      │
│ → Cost/Quality Router → Budget Planner → Workflow Orchestrator │
│ → Independent Analysis → Challenge → Claim/Evidence Analysis   │
│ → Adaptive Stop/Continue → Synthesis → Usage Report            │
└─────┬────────────────┬────────────────┬────────────────────────┘
      │                │                │
      ▼                ▼                ▼
 OpenAI Adapter     Claude Adapter    Gemini Adapter
      │                │                │
      └────────────────┴────────────────┘
                               │
          ┌────────────────────┴─────────────────────┐
          ▼                                          ▼
 Evidence Tools                              Usage & Telemetry
 Search / Browser / Drive RAG                Provider cost / Quality evals
          │                                          │
          ▼                                          ▼
 Permission-aware document index              Pricing/Routing configuration
```

### 5.1 強制架構原則

- 前端不得知道 provider API keys 或直接對 provider 發請求。
- 請求流程、狀態轉移、Credit 計算和扣款都由伺服器端驗證。
- Engine workflow 每次執行必須帶上 `answer_id`、`execution_id`、`tenant/account_id`、`depth`、`quote_id` 和 `budget_policy_version`。
- Provider adapters 回報原始 provider usage 與成本估算；統一的 cost service 將其轉換成內部標準記錄。
- 所有模型、工具、價格、路由權重及限制以版本化配置存放，不散落硬編碼在各個模組。
- AI workflow 使用背景 job／queue，並支援重試、取消、狀態查詢和 process crash recovery。
- Consumer App 和 Engine API 可以共用 Engine Core，但需分開認證、付款人、配額、Credit ledger 與產品價格。

### 5.2 建議內部模組

1. `request_normalizer`
2. `task_classifier`
3. `complexity_risk_estimator`
4. `model_registry`
5. `routing_policy`
6. `cost_estimator`
7. `credit_quote_service`
8. `workflow_orchestrator`
9. `provider_adapters`
10. `claim_extractor_and_clusterer`
11. `contradiction_detector`
12. `evidence_retriever_and_verifier`
13. `synthesis_service`
14. `followup_suggestion_service`
15. `credit_ledger_service`
16. `payment_and_webhook_service`
17. `room_participation_monitor`
18. `drive_permission_and_rag_service`
19. `feedback_and_eval_service`
20. `audit_and_observability_service`

---

## 6. 核心使用者流程

### 6.1 Standalone Answer

1. 使用者在 `answer.contact` 輸入問題，或從 History 開啟一個既有 Answer。
2. 前端可執行基本字數、空白、附件格式與大小檢查；不可自行判定預計成本。
3. 按送出後，backend 建立 `AnswerRequest`，執行**低成本 preflight**，不生成完整答案。
4. Engine 分析任務類型、複雜度、時效性、來源需求、預期成本與信心，回傳建議深度和可執行的 Credit quote。
5. UI 預選 Engine 建議的深度（Verified / Deep / Research），顯示預估 Credits；可允許用戶選擇其他深度或降低／提高深度。
6. 用戶點擊明確確認按鈕後，backend 再檢查 quote 未過期、餘額足夠、帳戶有效與權限，並建立 reservation。
7. Engine 在已授權預算內執行工作流，持續保存狀態、每個 stage 的 usage、延遲和錯誤。
8. 完成 synthesis 後計算實際 Credits、settle 並釋放未使用 reservation。
9. UI 展示最終答案、適用時的來源／驗證摘要、實際扣點，並提供可選的 Follow-up Suggestions。
10. 使用者可以複製、評分、分享或 Try Again。Try Again 是新的付費工作，必須重新 quote 和確認，不可視為同一答案的免費重跑。

### 6.2 付費開始前禁止事項

- 不可在未確認時執行完整的多模型分析、付費搜尋或完整合成。
- 允許的 quote 前工作僅限必要的低成本分類、估算、校驗與快取查找；此類成本由平台營運預算吸收，並設上限。
- 不可用「先偷偷跑完，再要求用戶付費」來製造既成事實或誘導購買。
- 如果 quote 已過期、上下文或附件變更、深度更改或路由成本配置發生重大變更，必須重新估價並重新確認。

### 6.3 用戶無足夠 Credits

顯示預估 Credits、目前可用餘額、缺少 Credits 數量與直接購買路徑。不可在餘額不足時先部分執行、結束後才告知用戶需要更多點數。使用者完成充值後需回到同一個待確認操作；不可自動執行一個用戶尚未確認或 quote 已過期的工作。

### 6.4 History 與 Answer 版本

- 每個正式執行的 Answer 有持續穩定的 `answer_id`，必要時保存不同 `answer_version`。
- 新問題、Follow-up、Try Again 和 Room Get an Answer 都要保留其 parent／source 連結，便於追蹤上下文與計費。
- 選擇 Follow-up suggestion 後，將對應的語意意圖作為新的 user turn，不能讓 UI 在沒有送出對話訊息的情況下偷偷執行付費工作。
- Favorite、history labels、分享連結或刪除行為必須由產品權限管理；刪除前端記錄不得自動刪除 audit / billing record。

---

## 7. Answer 深度與 Credit 預估 UX

### 7.1 三個主要深度

| 深度 | 用戶看到的名稱 | 初始 Credit 區間 | 定位 |
|---|---|---:|---|
| `verified` | Verified Answer | 1–3 Credits | 一般問題；多模型獨立分析、基本 cross-model challenge 和必要的基本查核 |
| `deep` | Deep Answer | 5–10 Credits | 複雜推理；更深入的挑戰、額外查核與針對性分析 |
| `research` | Research Answer | 20–50 Credits | 多步研究、來源搜尋、來源交叉驗證、多輪 challenge 與綜合分析 |

這些是產品目前的初始價格政策，不是固定模型呼叫數。具體額度必須依實際 COGS、品質與轉換率校準。

### 7.2 使用者操作規則

- 預設選擇由 Engine 的 preflight 給出，UI 以簡潔標示如 `Recommended` 表達。
- 使用者只需決定想要的答案深度，不需決定要使用哪個模型、執行幾輪或使用多少 token。
- 不將 1 / 2 / 3 Credits 做成與 Verified / Deep / Research 並列的永久主層級。
- 可在已選深度的範圍內顯示實際預估，例如 `Estimated usage: 2 Credits`；使用者確認的金額是該工作不可超過的收費上限。
- 最終 quote 必須由 server 返回。前端展示的 `1–3 Credits` 是大致區間，不代表這次必定花費區間上限或特定值。
- 若 Engine 無法在所選深度的最大 Credit cap 下完成最小品質門檻，需在執行前建議更高深度或調整路由；不得以靜默降低品質或超收來維持完成狀態。
- Deep / Research 不代表一定使用更貴模型；可以用更適合任務的模型組合、更多目標性驗證和證據工具來提升品質。

### 7.3 Quote UI 建議

```text
Recommended: Verified Answer
Estimated usage: 2 Credits
Scope: Independent model analysis + cross-checking

[Get Verified Answer · 2 Credits]
Change depth: Verified | Deep | Research
```

按鈕文字依深度與實際 quote 動態生成。使用者在確認前可更換深度；一旦確認，quote 和 Credit 上限固定。此 UI 是規格示意，非固定設計稿。

### 7.4 Credit quote 規則

Quote 至少包含：

- `quote_id`、`answer_id`、`account_id`、`depth`。
- `estimated_credits`、`max_authorized_credits`（MVP 可相同）。
- `estimated_cost_usd_internal`（僅內部可見；不得洩漏 provider 成本細節給 consumer）。
- `routing_plan_version`、`pricing_version`、`expires_at`。
- 簡短、用戶理解的工作範圍描述。

**不得把完整內部 COGS、模型選擇權重、供應商合約價格或安全政策回傳到消費者端。**

---

## 8. Follow-up Suggestions：點選式互動

### 8.1 產品目標

Answer 的一項核心交互革新，是在用戶與 AI 的 Follow-up 過程中，盡量預測使用者最可能想做的下一步，將其轉換成可點選的方向，使使用者不必每一次都手動敲字。這應是標準產品體驗，而不是付費解鎖的附加功能。

### 8.2 使用方式

每次最終答案完成後，若 `Follow-up Suggestions` 開啟且相關性足夠，可展示約 2–4 個高價值選項。例如：

- Compare alternatives
- Explain the main risks
- Give me a step-by-step plan
- Check the strongest counterargument
- Show examples
- Make the answer more concise

實際選項必須依上下文生成，不可在所有答案上永遠重複同一組固定選項。簡單問題可以只顯示 1–2 個選項或不顯示；不能為了符合數量而加入低相關選項。

### 8.3 生成策略與成本控制

優先順序：

1. **同一輪 synthesis 一併輸出結構化 suggestions。** 若 synthesis 已理解問題，不應再為相同上下文額外啟動一個昂貴、多模型工作流。
2. **規則／意圖模板補充。** 常見意圖（比較、摘要、風險、步驟、例子、反方觀點）可由低成本邏輯提供候選，再用上下文排序。
3. **輕量模型生成。** 只有答案上下文複雜且同輪無法可靠產生選項時才考慮；需明確設置成本上限。
4. **快取。** 同一答案的 suggestions 以 answer version 為 key 儲存，避免每次重繪都重算。
5. **不顯示就不生成。** 使用者在 Preferences 關閉 Follow-up Suggestions 時，不應仍執行專門用於 suggestions 的額外模型呼叫。

Suggestions 的成本應納入產品預算／對話工作流，而不是另行向用戶扣一筆未告知的 Credits。若生成 suggestions 需要超出此次授權預算的昂貴呼叫，則略過 suggestions，不能默默擴大工作量。

### 8.4 選項資料格式

每個建議不是單純的顯示文字，至少要包含：

```json
{
  "id": "compare_alternatives",
  "label": "Compare alternatives",
  "intent": "compare_options",
  "prompt": "Compare the leading alternatives discussed above, including trade-offs.",
  "requires_new_answer": true,
  "suggestion_version": "v1"
}
```

- `label` 是 UI 顯示文字，受用戶語言設定影響。
- `intent` 是穩定枚舉，用於分析點選表現與防止任意動作。
- `prompt` 是後端可審計的真正 follow-up 指令；不可從未驗證的任意前端文字直接拼進系統 prompt。
- 點擊後先建立正常 user turn，並依需求重新 quote；如果這是獨立的付費 Answer，必須依同一套確認與扣點規則執行。

### 8.5 UX 及安全要求

- 始終保留自由輸入框；Suggestions 是捷徑，不是唯一互動方式。
- 顯示的選項不能偽裝成用戶已經作出的決定；只有點擊後才執行。
- 建議應是可理解的下一步任務，不應顯示「Buy now」類操縱消費的選項。
- 不為了促成深度升級而刻意提供不完整答案。
- 對醫療、法律、金融及其他高風險情境，選項可以引導用戶查證限制或尋求專業人士，但不代替必要安全處理。
- 記錄曝光、點擊、未採用、後續完成與額外成本；以實際節省輸入與提高完成率評估價值，而不是只看點擊率。

---

## 9. Answer Room 規格

### 9.1 產品目的

Answer Room 是人類成員與 AI 共同討論同一話題的協作空間。成員先自由交流，Contact AI 在有潛在價值時提供中性提示；只有使用者明確要求正式 AI 答案時，才進入 Verified / Deep / Research、quote、confirm 和 Credit 扣點流程。

### 9.2 Room 主要能力

- 建立、命名、邀請成員、加入與離開 Room。
- 成員訊息、時間戳、回覆／引用、反應、copy/share 等基本對話能力。
- Room header 顯示名稱、成員資訊及 Room options。
- `AI participation` 開關控制 Engine 是否對該 Room 執行低成本價值判斷。
- `Hide members’ Contact Cards` 控制 Room 是否允許成員互相開啟公開 Contact Card 預覽；這是展示權限，不等同於關閉成員資料的後端授權。
- Room owner 可設定正式 Answer 的付款方式和允許的 Answer depth；所有設定由伺服器端保存並授權驗證。
- Room 可以附加由任何成員授權的 Contact Drive 參考文件。
- 正式答案必須保留 Room 的討論摘要、相關訊息引用、附加文件和執行深度等上下文來源。

### 9.3 Room 的三類 AI 工作

**A. 背景分析與介入判斷（不扣用戶 Credits）**

- 判斷最新的討論是否有明顯遺漏、衝突、值得補充的觀點，或是成員在等待一個答案。
- 使用低成本模型、規則和批次分析；一般情況只輸入最近變更的訊息和更新後的摘要，不重送完整 Room 歷史。
- 採 debounce／batching：短時間連發多則訊息時合併檢查，不為每一則訊息單獨觸發 AI 呼叫。
- 設置 Room 級、帳戶級、租戶級的內部預算、頻率上限、最大上下文與防濫用 circuit breaker。此限制不作為可見的「每日免費次數」向用戶售賣或展示。
- 不允許 background monitor 在用戶未請求正式答案時先生成整篇完整答案。

**B. 中性提示（不扣用戶 Credits）**

例如 `Contact AI may have a useful perspective.` 或 `Contact AI may help resolve a disagreement.` 提示必須是可選、節制、不反覆推銷。它只代表值得進一步分析，不代表完整答案已經預先產生，也不可以以「已經有 X 個 AI 觀點」營造付費既成事實。

**C. 使用者主動要求正式答案（使用 Credits）**

- 用戶點擊 **Get a Verified Answer for This Discussion**（或符合 UI 尺寸的短版 `Get an Answer`，但完整入口需明確連結到當前討論）。
- 預設推薦深度，顯示 Verified / Deep / Research 以及預估 Credits。
- 用戶明確確認後才建立 reservation 並開始正式工作流。
- 正式工作可使用 Room 訊息、Room 摘要及仍有效授權的 Drive 文件；只取與問題相關的內容。

### 9.4 Room Credit payer

Room 支援兩種設定：

- `payer = owner`：Room owner 支付所有正式 Get an Answer 請求。用戶端在該答案上顯示 **Paid by Room owner**。
- `payer = requester`：提出正式請求的成員支付自己的 Credits。用戶端顯示 **Paid by you**（對請求者而言）或對其他成員顯示適合的中性付款資訊。

規則：

- payer 是 Room 設定的伺服器端值，不能接受任意前端覆蓋。
- `owner` 模式需確認 Room owner 有足夠 Credits 並有權限；請求者不得繞過餘額不足或 Room 的深度限制。
- `requester` 模式需確認請求者有足夠個人／組織可用餘額。
- 若 Room 設定只允許某些 depth，UI 不得讓使用者執行未獲允許的 depth；後端仍需再次驗證。
- 每次正式 Answer 的付款人、Credit ledger entries、Room ID、requester ID、depth、quote ID、answer ID 均要可審計。
- Owner 付款與 requester 付款都使用相同的 quote/reservation/settlement；只有付款來源不同。

### 9.5 Room AI 參與與通知

- AI 參與設定必須可以關閉；關閉時不再啟動 background monitor，但不影響成員手動請求正式答案（除非產品另有設定）。
- 預設提示頻率需要節制，例如每個 Room 經過新訊息批次、且達到明確介入價值門檻才提示；不可在每一輪對話都出現。
- 同一討論的重複提示需冷卻時間；成員已忽略或關閉提示後，應降低相同情境重複出現。
- 監測只做「是否值得介入」判斷，不可以向 UI 回傳尚未完成的虛假結論。
- Room 普通對話不扣 Answer Credits，但產品內部仍需追蹤每個 Room 的背景 COGS，讓營運端防止成本失控。

---

## 10. Contact Drive 共同參考資料

### 10.1 權限模型

任何 Room 成員均可從自己有權存取的 Contact Drive 選擇文件，加入 Room 共同參考資料。包括但不限於文件、報告、PDF、試算表和專案資料。

- 原始檔案始終保留在上傳者／擁有者的 Drive。
- Room 取得的是被授權的特定文件讀取權限，不獲得整個 Drive 的存取權。
- Room 管理員可從 Room 移除參考文件，但不得刪除其他成員的原始文件。
- 文件擁有者可撤回該文件給 Room 的授權。
- 每個文件必須保存 owner、Room ID、授權狀態、授權來源、加入時間、撤回時間、文件版本與存取 audit。

### 10.2 AI retrieval 與 citation

- Answer 執行前，依當前用戶問題、Room 摘要與相關訊息，檢索被授權文件的相關段落，而不是把所有檔案全文塞進每次 prompt。
- 盡可能使用 RAG：文件 parsing → page/section-aware chunking → embedding/index → query retrieval → rerank → context pack。
- 每個 chunk 必須帶有 `document_id`、`file_version`、`page_number`（若適用）、`section`、`chunk_id`、`owner_id`、`room_id` 及 permission reference。
- 答案引用文件時應顯示文件名稱、頁碼／章節或可定位的段落資訊；引用必須能回查到原始文件。
- 如果文件資料與外部來源互相矛盾，答案要指出衝突來源，不得默默選擇符合模型先前答案的資料。
- 文件內容只能作為資料，不可把文件內的指令當作提升權限或覆蓋 Engine system policy 的指令。需抵禦 prompt injection。

### 10.3 撤回授權的工程行為

- 一旦撤回或移除，新的 Answer request 在授權檢查時必須立刻排除該文件。
- 清除或標記失效的相關 retrieval cache、Room-specific document index / derived context；背景刪除任務必須可追蹤且重試。
- 不得因舊 embedding 尚存在就繼續使用已撤回文件。
- 已完成歷史答案如何保留要遵守帳戶刪除、資料保留與 audit 規則；撤回授權必須保證「未來不再用於新答案」，不能承諾改寫既有歷史答案除非實際提供此能力。
- 任何文件讀取、檢索、引用與權限變動需有 audit event。

### 10.4 外部與用戶提供資料的優先順序

當任務要求回答「Room 內專案的實際情況」，先使用被授權的專案材料；若問題涉及法律、價格、政策、現況或需要外部事實，則使用可追溯的外部來源交叉核對。不能只因文件是 Room 成員提供就假設它是真實、最新或完整。

---

## 11. Contact Engine 核心模組

### 11.1 Request Normalization

- 移除無意義空白，保留原始用戶內容的語義、語言、結構及明確要求。
- 以獨立欄位表示 `question`、conversation context、room context、attachments、requested depth、language、response preferences、source policy。
- 做 prompt-injection／惡意輸入檢查；不可把用戶輸入合併為可信 system instruction。
- 對極長上下文建立摘要及相關片段選擇策略，並保留可回溯來源。
- 文字正規化不可破壞程式碼、公式、網址、引文、檔案內容或用戶指定格式。

### 11.2 Task Classification

至少辨識以下任務類型：

- `simple_factual`
- `current_factual`
- `reasoning`
- `multi_constraint_decision`
- `comparison`
- `coding_or_technical`
- `research`
- `summarization_or_transformation`
- `document_grounded`
- `room_synthesis`
- `creative`
- `high_impact_advice`
- `unknown_or_ambiguous`

分類要回傳 confidence、trigger flags 與理由代碼，不要只輸出自由形式文字。若低信心或觸發高風險條件，採保守路由、詢問必要澄清或提高 evidence depth。

### 11.3 Complexity / Risk Estimator

初始分類使用「規則 + 輕量模型」而不是從第一天就追求不可解釋的完全自主 optimizer。至少考慮：

- 任務數量、問題長度及輸入／附件 token 預估。
- 推理深度、約束數、比較項數、依賴前提數。
- 對時效性、外部資料、引用與文件 groundedness 的需求。
- 事實錯誤的影響程度和不確定性。
- 是否存在互相衝突的候選主張。
- 需要的輸出長度與結構。
- 可用模型的預測品質、歷史失敗率、延遲與成本。
- 是否有授權 Drive 文檔或可重用的可靠快取。

分類器必須輸出 structured result，例如 `task_type`、`complexity_score`、`risk_flags`、`freshness_required`、`evidence_required`、`recommended_depth`、`confidence`。

### 11.4 Model Registry

每個可用模型至少記錄：

- Provider、model ID、display/internal name、版本／快照。
- 支援的任務能力、最大 input/output、structured output、tool use、JSON mode 等。
- input / output / cached token 價格與有效期間。
- 可用區域、資料處理政策、延遲分布、成功率、rate limits。
- 被允許的使用場景（classifier、independent reasoning、challenge、synthesis、judge 等）。
- 品質評估資料與版本。
- fallback alternatives、timeout、retry policy。

**模型名稱與價格必須集中配置。** 不得把特定模型 ID 固定寫死在 Answer 邏輯或公開 API schema。

### 11.5 Workflow Orchestrator

- 依 request 建立持久化工作流，stage 間的輸入／輸出需結構化。
- 支援並行 provider execution、超時、取消、重試、fallback 和部分結果。
- 每個 stage 有獨立 job ID、開始／結束時間、usage、狀態、錯誤碼與 cost estimate。
- 每次重試要有 idempotency scope，避免重複付款與重複插入結果。
- 當進程中斷時可從最近安全狀態恢復，而不是將整個答案重跑。
- 所有迭代都受 user-confirmed Credits budget 和系統級 COGS 上限共同控制。

---

## 12. 智慧路由與成本最佳化演算法

### 12.1 設計目標

Engine 的目標不是無條件使用便宜模型，也不是每題都使用最昂貴模型。正確目標是：在使用者選擇的深度、授權 Credit 上限、安全要求與品質門檻內，選出**預期品質／可靠性足夠且成本最低的模型與工具組合**。

### 12.2 Preflight 原則

1. 先使用本地規則、字數、問題類型、context 長度和固定風險特徵做初步判斷。
2. 只有規則不能可靠分類時才呼叫低成本 classifier 模型；classifier 不應預先完成用戶真正付費的答案。
3. 用輕量結果產生深度建議和成本預估，回傳 quote。
4. 高成本推理、長篇 research、外部搜尋及 synthesis 都要等用戶確認後才執行（Room background monitor 的明確產品預算政策除外）。
5. 對可安全重用的分類結果做短期快取；涉及私有資料時用 account/tenant scope 做隔離，避免跨帳戶快取洩漏。

### 12.3 初始複雜度評分

建議先以 0–100 的可配置 score 驅動 routing，權重需透過離線 eval 與真實工作負載調整。例如：

| Feature | 初始權重示例 |
|---|---:|
| 多任務／多問題數 | 15% |
| 推理深度與約束數 | 20% |
| 時效性與外部查證需求 | 15% |
| 證據要求／引用要求 | 15% |
| 事實錯誤風險 | 15% |
| 候選答案矛盾風險 | 10% |
| 上下文／附件複雜度 | 10% |

以上權重只是初始可調配置，不應視為科學常數。也不應只因文字很長就自動視為複雜；短問題可能需要研究，長文可能只需摘要。

**初始深度推薦規則（建議配置）：**

- `Verified`：一般問題、任務清楚、低至中等複雜度，沒有強烈的多步 Research 要求。
- `Deep`：多約束推理、比較方案、複雜技術／商業判斷或需要幾輪有針對性的 challenge。
- `Research`：用戶明確要求研究／引用／最新資料，或需要多來源、多步調查與交叉驗證。
- 高風險或含最新事實的問題可能不一定需要整個 Research 深度，但應強制開啟 evidence policy，並在 UI 說明其限制。
- 分類器低信心時，推薦保守深度或顯示清楚的工作範圍，不要以低估價吸引用戶再於執行中升級。

### 12.4 路由評分

每個候選模型／工具方案可依以下維度評分：

- 任務能力與歷史品質適配度。
- 估計可靠性及該任務上的評估集表現。
- 需不需要外部工具、JSON schema 或長上下文。
- 延遲、可用性、近期錯誤率與 rate limit 狀態。
- Input/output token 預估成本、tool cost、重試成本。
- 與其他候選模型的獨立性／觀點多樣性。
- 隱私區域、資料處理與政策約束。

路由需先套用硬性約束（可用能力、隱私／安全、最大成本、已選深度），再在可行方案間以加權效用選取。不要只以模型名稱或單次 benchmark 排名作路由。

```text
candidate utility =
    expected_task_quality
  + expected_reliability
  + provider_diversity_value
  + availability_value
  - normalized_cost_penalty
  - latency_penalty
  - policy_risk_penalty
```

權重必須版本化且可以用離線測試集比較。模型品質分不能被解讀為該答案的「正確機率」。

### 12.5 成本預估公式

Cost estimator 需要估算整個預計 workflow，而不是只估算第一次模型呼叫：

```text
estimated_provider_cost =
    Σ(model_input_tokens × configured_input_rate)
  + Σ(model_output_tokens × configured_output_rate)
  + Σ(cached_token_costs)
  + search/browser/tool_costs
  + embedding/rerank/document_processing_costs
  + expected_retry_and_fallback_costs
```

接著加上估算誤差 buffer：

```text
risk_adjusted_cost = estimated_provider_cost × (1 + uncertainty_buffer)
```

`uncertainty_buffer` 依任務類型、上下文長度、模型輸出方差、Research 步數和歷史估算誤差設定，且可隨遙測修正。

**Credit 映射不得假設 1 Credit 永遠等於 1 次模型呼叫，也不得固定等於某個 token 數。** 應由產品定價表、最優惠 Credit pack 的有效收入、支付費、稅／退款、目標 COGS 比率與工作成本共同推導。

```text
net_revenue_per_credit =
    expected_net_collected_revenue_for_plan / credits_issued_for_plan

target_cogs_per_credit =
    net_revenue_per_credit × configured_max_provider_cogs_ratio

estimated_credits =
    ceil(risk_adjusted_cost / target_cogs_per_credit)
```

以上公式屬於起始經濟模型，實作時要有最低值、深度範圍、最優惠 pack、優惠期間、幣別換算、支付手續費及不同市場稅務等配置。Engine API 與 Consumer Answer 必須各自計算，不能互用轉換率。

### 12.6 深度範圍與成本保護

- 所有 depth 的 Credit 下限／上限及模型預算由後端設定。
- 先嘗試使用符合最低品質門檻的較低成本模型版本，再依必要性增加更強模型、更多輪 challenge 或 evidence tools。
- 若經可行路由優化後仍無法在已選 depth 的最大 Credits 內完成最低工作範圍，必須在執行前要求更高深度或明確縮小範圍。
- 不可把預估值簡單 clamp 到區間上限後仍執行昂貴 workflow；否則估價會失真並傷害毛利。
- quote 顯示後，該次工作最高消耗不得高於使用者確認的數字。若執行時發現需要額外工作，應在預算停止點合成目前可支持的答案、標記限制，或中止並請使用者重新確認更高預算；不得默默加收。
- 若 provider 的意外輸出量超預算，先停止後續工作並依 fallback／截斷政策處理；不得以信用卡或餘額自動追加未授權消費。

### 12.7 單位經濟的上線門檻

建議初期設定 provider inference COGS 目標佔淨收入比例（例如 30%）作為可調的啟動保護線，超過警戒線自動告警，超過硬性預算則停止擴展工作流或選擇成本更低的可行路由。**30% 是建議的初始營運設定，不是由目前文件證明已達成的事實。** 上線前必須用實際 token usage、Provider 價格與各 Credit pack 的收入測試。尤其對 $500 Consumer pack（有效收入約 $0.167/Credit），1 Credit 工作是否能維持三個 provider 的多模型獨立分析，必須以真實 COGS 實測決定；不能以營銷口號代替經濟驗證。

### 12.8 成本／品質自我改進

- 先將新的路由器以 shadow mode 對真實匿名化／獲授權工作負載做預測，不立即控制真實付款。
- 比較「預測成本」與「實際成本」、「建議深度」與人工／離線 eval 深度、失敗率及用戶滿意度。
- 每次策略調整保存 `routing_policy_version`，保留 rollback 路徑。
- 在線學習只能在經驗證的評估框架下推出；不得讓單純的點擊率或模型自評直接改變品質門檻。

---

## 13. 多模型驗證與反駁工作流

### 13.1 工作流階段

1. **Independent first pass**：初始模型盡可能各自只收到原始問題、相關必要上下文與相同任務約束；不要先把某個模型答案灌入其他模型後仍標示為獨立分析。
2. **Structured extraction**：將候選答案正規化為結論、主張、假設、推理依賴、證據需求及不確定性。
3. **Claim alignment**：將不同模型表達的等價主張聚合，區分語義衝突與文風差異。
4. **Cross-model challenge**：針對重要分歧和脆弱主張，要求一個或多個模型提出反例、指出缺失假設、找出邏輯斷點或替代解釋。
5. **Evidence checks**：對當前／可查證／重要主張查找適當來源。若無可靠工具或證據則明確保留未知。
6. **Targeted re-evaluation**：只針對實質分歧或證據不足的部分追加 computation，不重跑整個流程。
7. **Stop decision**：當證據與分歧達到足夠門檻，或剩餘預算不足／追加分析的預期價值太低時，停止。
8. **Final synthesis**：回覆核心問題，呈現有證據支持的結論、限制與需要時的 citations。
9. **Usage settlement**：彙總所有 provider/tool usage，結算 Credits 並釋放餘額。

### 13.2 Verified 的最低品質語義

Verified 必須實際完成多模型交叉分析，而不是把單模型答案套上一個 Verified 標籤。建議路由：

- 預設使用 OpenAI、Claude、Gemini 的適當低成本／中階模型版本執行獨立第一輪，模型強度由任務能力與預算決定，不強制每次使用三個最昂貴 frontier models。
- 產品核心路徑應優先保留三個 provider family 的比較能力；但若單位經濟或 provider 可用性不允許，需在 launch policy 明確選擇最低 provider 數。建議最低為兩個不同 provider family 加一個 challenge／judge 路徑，且必須在內部與外部透明標示實際完成程度。
- 不得宣稱「Verified across 3 AI systems」除非本次執行確實完成三個不同 AI 系統的分析，且有執行紀錄佐證。
- 若只有兩個 provider 可用，允許 graceful degradation，但狀態需顯示 `Partial verification` 或相當的清楚文案，不可假稱三方完成。
- 上線前須通過 unit economics gate：在各最優惠 Consumer pack 的有效收入情境下，最低 Verified 工作流要符合配置的 COGS 上限。如果不能，必須調整最低 Credit 價、深度區間或最小工作範圍，不能靠暗中降低驗證品質補差額。

### 13.3 Challenge 要檢查什麼

- 未明說但成立所需的前提。
- 關鍵結論是否超出證據可支持範圍。
- 反例、替代解釋與可能的邊界條件。
- 算術、單位、日期、名稱、引用是否一致。
- 模型是否把猜測寫成事實。
- 來源是否可信、更新時間是否適當、是否真正支持主張。
- 是否忽略用戶提供的文件或上下文。
- 模型之間同意是否只是源自共同訓練／相似錯誤模式；同意不能當作最終真理證明。

### 13.4 Claim / Contradiction 狀態

重要主張在內部可使用以下狀態：

- `supported`
- `mostly_supported`
- `conflicting_evidence`
- `insufficient_evidence`
- `unresolved`
- `not_checked`（因為不是關鍵主張或預算／工具限制）

每個 claim 應記錄支援模型／來源、相反主張、重要性、 evidence references、最終狀態與採用理由。

### 13.5 Adaptive stop

追加工作只在下列情況之一成立時執行：

- 有高重要性主張發生實質衝突。
- 關鍵來源缺失或可疑。
- 輸出包含高風險現況／規則／政策而尚未證實。
- 模型初始 confidence 不一致且可能改變核心建議。
- 工具回傳新證據需要針對性解釋。

停止條件：

- 關鍵主張已被足夠證據支持且重大矛盾已解決；或
- 剩餘最大預算／Credits 到達上限；或
- 追加計算的邊際品質收益預期低於其成本；或
- 合法／政策／工具限制使更多查證不可行。

預算耗盡時仍需產出合理、誠實的結果或清楚說明執行未完成原因；不可捏造一個「已充分驗證」的標籤。

---

## 14. Research、證據與引用

### 14.1 需要 Evidence 的情況

- 當前事件、當前公司／產品／人物資訊。
- 價格、產品規格、版本、時間表、API 行為。
- 法律、法規、政策、合規及政府規定。
- 學術／技術研究與重要統計數據。
- 對用戶決策有實質影響且可外部驗證的主張。
- 用戶明確要求 source、citation 或 research。
- Room 內文件提供的事實需要與外部現況互相核對。

Evidence checking 是自適應流程，不要求每個低風險問答都做相同深度搜尋；但遇到高時效性或高影響主張時，不能只憑多模型共識。

### 14.2 Evidence schema

每筆證據至少保存：

- `evidence_id`、`answer_id`、`claim_id`。
- `source_type`（web、Drive file、user attachment、official docs、database 等）。
- `source_url` 或 `document_id`、標題、domain／owner、來源版本。
- `retrieved_at`、來源發布／更新時間（若可取）、page／section／chunk。
- relevant excerpt／snippet 或可回查引用位置。
- `support_status`、source reliability signals、與主張的關係。
- 原始資料 hash 或版本 ID，供稽核與變更偵測。

### 14.3 Source quality policy

- 優先第一手來源（官方文件、原始研究、政府／監管網站、產品供應商文檔）來驗證相應主張。
- 第二手來源可用於補充背景或交叉檢查；單篇高流量文章不能自動視為權威證據。
- 搜尋引擎 snippet 不得被當作已完整核驗的原文；可行時讀取來源本身並記錄時間。
- 區分來源「存在」與來源「支持這個主張」；citation 不能只是與問題相似的網址。
- 若來源內容不可訪問、過期或相互矛盾，要保留限制。
- 不得捏造標題、網址、作者、頁碼或引用文字。

### 14.4 Research 工作流

- 根據任務建立 research plan／subquestions（限量、可檢查）。
- 進行多來源檢索並追蹤已查主張；避免重複搜尋相同查詢而浪費成本。
- 從來源抽取相關 claims，建立 source-to-claim map。
- 讓模型針對證據建立支持／反駁／不確定狀態。
- 來源數量不是品質代理指標；更重要的是來源獨立性、適用性與直接支持程度。
- 最終答案應讓用戶可以打開來源檢查必要的關鍵主張。

---

## 15. 最終答案、透明度與回饋

### 15.1 Synthesis 要求

最終答案應：

1. 先回答用戶實際提出的問題。
2. 將主要結論與解釋組織成清晰結構，符合 `Response Style` / `Response Format` 偏好。
3. 讓證據最充分的結論優先，而不是採納最有自信語氣的模型。
4. 清楚指出重要前提、風險與未知之處。
5. 當矛盾無法解決時保留差異，必要時說明哪部分仍未決。
6. 有 evidence 的可查證主張提供 citations；引用要對應真正支持它的來源。
7. 不暴露無必要的原始模型長篇答案、內部 chain-of-thought 或 provider 私有細節。
8. 不使用沒有統計校準的 `94% true` 或任意正確率百分比。

### 15.2 驗證狀態 UI

可向用戶展示簡短摘要，例如：

- `Cross-model analysis completed`
- `Evidence checks completed`
- `Partially verified — one provider unavailable`
- `Some disagreement remains`

可以顯示實際模型系統數、challenge rounds、evidence checks 等有執行紀錄支撐的數字，但必須由 execution log 計算，不可以寫死範例數字。數字說明執行過程，不代表正確率保證。

### 15.3 回饋

- 支援 `Rate Answer` / helpfulness 回饋與錯誤回報。
- 可收集：答案是否有用、事實錯誤、引用失效、未遵守任務、與其他候選答案差異。
- 回饋要連結 `answer_id` 和 `answer_version`，但不代表用戶的「有用」評分等同於答案正確。
- 對模型品質評估使用抽樣人工標註、可驗證 outcome 或來源 check；不可完全依賴模型自我評分。

---

## 16. API 介面與請求契約

### 16.1 API 設計原則

- Consumer App 使用 session／帳戶授權的 application API；不暴露 Engine API key。
- 開發者以 Contact Engine API key 呼叫 `engine.contact`。API key 只能存 server-side，支援撤銷、rotate、scope、額度及使用報告。
- 對外 API 使用穩定 schema、machine-readable error code、明確 rate limits 和 request ID。MVP 正式 canonical paths 沿用目前開發者文件的 `/api/answer`、`/api/verify`、`/api/research`、`/api/synthesis`；不要在沒有必要時同時實作兩套不同的 `/api` 與 `/v1` 業務路徑。未來需要破壞相容性時，再以版本化路徑推出新 major version。
- 所有有帳務影響的請求均接受 `Idempotency-Key` 或等效機制。
- 目前文件展示的四個 `/api/...` 路徑即為 MVP canonical API；不要同時維護兩套不同的業務端點。若未來需要 breaking change，再增加新的 major version 路徑並制定遷移期。

### 16.2 Consumer App 內部端點建議

| Method | Endpoint | 功能 |
|---|---|---|
| `POST` | `/app/answers/estimate` | 建立 Answer 草稿／執行低成本 preflight，回傳推薦 depth 與 quote，不執行完整付費答案 |
| `POST` | `/app/answers/{answer_id}/confirm` | 驗證 quote、權限與餘額，reservation 後啟動正式工作流 |
| `GET` | `/app/answers/{answer_id}` | 查詢答案內容、狀態、可見的 verification summary 和消耗量 |
| `POST` | `/app/answers/{answer_id}/cancel` | 在允許階段取消尚未完成的工作流並依規則結算 |
| `POST` | `/app/answers/{answer_id}/feedback` | 提交評分／錯誤回報 |
| `GET` | `/app/credits/balance` | 取得當前消費者可用餘額及按 batch 的必要資訊摘要 |
| `GET` | `/app/credits/ledger` | 取得使用者有權查看的 Credit transactions |
| `GET/PUT` | `/app/settings/preferences` | 讀取／更新 Answer Preferences |
| `GET/PUT` | `/app/settings/general` | 讀取／更新 Answer General 設定／inheritance overrides |
| `GET` | `/app/billing/history` | 取得支付交易與發票歷史 |
| `POST` | `/app/rooms/{room_id}/answers/estimate` | 對當前 Room 內容預估正式答案 |
| `POST` | `/app/rooms/{room_id}/answers/{answer_id}/confirm` | 確認 Room Answer，依 payer 規則 reservation |

實際前綴可以沿用既有 backend 標準，但端點職責與兩階段確認原則不可改變。

### 16.3 Contact Engine 公開 API

現有開發者文件列出的核心操作：

- `POST /api/answer`：一般 Answer orchestration。
- `POST /api/verify`：對既有答案／主張執行驗證。
- `POST /api/research`：多步 Research 與來源查核。
- `POST /api/synthesis`：合成多個模型輸出或來源資料。

其他必須提供或規劃的端點：

- `GET /api/usage` 或等效 usage endpoint。
- `GET /api/credits/balance`、`GET /api/credits/ledger`（對該 API customer 可見）。
- `GET /api/models` 或能力 registry endpoint（只回傳公開能力資訊，不洩漏內部 routing policy）。
- `GET /api/health`（不回傳 secrets 或內部敏感環境資訊）。

Engine API 可為程式化客戶配置 auto-execute budget；與 Consumer UI 不同，開發者請求可以在帳戶預先授權的 budget／quota 內直接執行，不必每一個 HTTP request 都進行人工按鈕確認。但必須有明確的 `max_credits` / billing policy、餘額檢查及使用者在帳戶設定的支出上限。為防止意外消費，`execute` 預設為 `false`；只有 request 明確提供 `execute: true`，或 API account 已在 server-side 設定可驗證的 auto-execute policy 且請求符合該 policy，才可進入付費執行。現有 developer quickstart 中只傳 `question` 和 `depth` 的範例不足以表達生產環境的預算授權，必須更新文件，加入 `execute`、`max_credits` 與 `Idempotency-Key` 示例。

### 16.4 `POST /api/answer` 請求示例

```json
{
  "question": "Compare the trade-offs of PostgreSQL and DynamoDB for a multi-tenant SaaS.",
  "depth": "verified",
  "response_style": "balanced",
  "response_format": "automatic",
  "language": "en",
  "max_credits": 3,
  "execute": true,
  "metadata": {
    "project": "architecture-review"
  }
}
```

參數：

- `question`（必填）：當前問題。
- `depth`：`auto | verified | deep | research`；`auto` 由 Engine 推薦。
- `response_style`：`balanced | professional | friendly | direct | academic`。
- `response_format`：`automatic | structured | plain_prose`。
- `language`：有效語言標記。
- `max_credits`：Caller 授權的最大 API Credits；不可在未提供上限且 account 未設定 auto-budget 時默認無限消費。
- `execute`：是否可以執行。若 false，或帳戶 policy 要求 quote-first，僅返回估價。
- `metadata`：可選業務標籤；需限制大小並禁止在此存放 secrets／敏感個資。

### 16.5 回應示例

```json
{
  "answer_id": "ans_01J...",
  "status": "completed",
  "depth_requested": "verified",
  "depth_used": "verified",
  "credits": {
    "estimated": 2,
    "reserved": 2,
    "consumed": 2,
    "released": 0,
    "currency": "engine_api_credits"
  },
  "answer": "...",
  "verification": {
    "status": "completed",
    "provider_families_used": 3,
    "challenge_rounds": 1,
    "evidence_checks": 0,
    "unresolved_issues": []
  },
  "sources": [],
  "follow_up_suggestions": [],
  "usage": {
    "input_tokens": 0,
    "output_tokens": 0,
    "tool_calls": 0
  },
  "request_id": "req_01J..."
}
```

數字需由本次 execution 記錄產生，上例的 `provider_families_used: 3`、Credits 與 tokens 只是 schema 示意。若 `verification.status` 是 `partial`，不能同時回報為完整完成。

### 16.6 Error response

```json
{
  "error": {
    "code": "INSUFFICIENT_CREDITS",
    "message": "The request exceeds the available credit balance.",
    "retryable": false,
    "details": {
      "required_credits": 8,
      "available_credits": 5
    }
  },
  "request_id": "req_01J..."
}
```

穩定錯誤碼至少包括：`INVALID_REQUEST`、`UNAUTHORIZED`、`FORBIDDEN`、`RATE_LIMITED`、`INSUFFICIENT_CREDITS`、`QUOTE_EXPIRED`、`BUDGET_EXCEEDED`、`PROVIDER_UNAVAILABLE`、`PROVIDER_TIMEOUT`、`VERIFICATION_PARTIAL`、`ATTACHMENT_NOT_ALLOWED`、`DOCUMENT_ACCESS_REVOKED`、`CANCELLED`、`INTERNAL_ERROR`。

---

## 17. 狀態機、資料模型與事件

### 17.1 Answer state machine

```text
DRAFT
  ↓
ESTIMATING
  ↓
AWAITING_CONFIRMATION
  ↓
RESERVED
  ↓
QUEUED / RUNNING
  ↓
VERIFYING
  ↓
SYNTHESIZING
  ↓
SETTLING
  ↓
COMPLETED
```

失敗或中止狀態：

- `ESTIMATION_FAILED`
- `QUOTE_EXPIRED`
- `INSUFFICIENT_CREDITS`
- `RESERVATION_FAILED`
- `PROVIDER_FAILED`
- `EXECUTION_FAILED`
- `BUDGET_EXCEEDED`
- `CANCELLED`
- `PARTIAL_COMPLETED`
- `SETTLEMENT_PENDING`

狀態轉移由 server-side state machine 控制；前端不能把 loading 完成或局部結果直接視為最終答案。

### 17.2 核心資料實體

#### `AnswerRequest`

- `id`
- `account_id`
- `organization_id`（nullable）
- `room_id`（nullable）
- `requester_id`
- `payer_type` / `payer_account_id`（Room 請求）
- `parent_answer_id`（Follow-up / Try Again）
- `question` 或指向加密內容存儲的 reference
- `requested_depth`、`recommended_depth`、`depth_used`
- `task_type`、`complexity_score`、`risk_flags`
- `quote_id`、`estimated_credits`、`max_authorized_credits`
- `status`、`idempotency_key`
- `preference_snapshot`、`routing_policy_version`、`pricing_version`
- `created_at`、`confirmed_at`、`started_at`、`completed_at`

#### `AnswerExecution`

- `id`、`answer_id`、`stage`、`iteration`
- `provider`、`model_id`、`purpose`（analysis/challenge/judge/synthesis/classifier）
- `input_metadata`、`output_metadata`（token count、content hash、schema validity）
- `latency_ms`、`status`、`error_code`、`retry_of_execution_id`
- `estimated_cost_usd`、`actual_cost_usd`（internal only）
- `created_at`、`started_at`、`completed_at`

#### `AnswerUsage`

- `answer_id`
- `credit_type`（consumer / engine_api）
- `quote_id`
- `reserved_credits`、`actual_credits`、`released_credits`
- `provider_cost_usd`、`tool_cost_usd`、`total_internal_cost_usd`
- `billing_status`、`settlement_id`、`created_at`、`settled_at`

#### `AnswerClaim`

- `id`、`answer_id`、`claim_text`、`importance`
- `support_status`、`contradiction_status`
- `supporting_execution_ids`、`contradicting_execution_ids`
- `evidence_ids`、`resolution_summary`

#### `AnswerEvidence`

- `id`、`answer_id`、`claim_id`
- `source_type`、`source_url` / `document_id`
- `title`、`domain`、`retrieved_at`、`published_at`（nullable）
- `page_number`、`section`、`chunk_id`、`source_version`
- `relevance`、`support_status`、`excerpt_hash`

#### `AnswerResult`

- `answer_id`、`answer_version`
- `final_answer`／encrypted body reference
- `verification_status`、`verification_summary`
- `uncertainty_summary`、`sources`、`follow_up_suggestions`
- `created_at`

### 17.3 Room / Drive 核心資料

`Room` 至少需有：`id`、`owner_id`、`title`、`settings`、`ai_participation_enabled`、`hide_member_cards`、`payer`、`allowed_depths`、`created_at`、`updated_at`。

`RoomMember` 至少有：`room_id`、`user_id`、`role`、`joined_at`、`left_at`、`status`。

`RoomMessage` 至少有：`id`、`room_id`、`author_id`、`content_ref`、`reply_to_id`、`created_at`、`edited_at`、`deleted_at`（如支援編輯／刪除）。

`RoomDocumentGrant` 至少有：`id`、`room_id`、`document_id`、`owner_id`、`granting_user_id`、`permission_scope`、`status`、`granted_at`、`revoked_at`、`document_version`。

`RoomMonitorRun` 至少有：`room_id`、`trigger_batch_id`、`summary_version`、`model_id`、`estimated_cost`、`status`、`hint_emitted`、`created_at`。

### 17.4 Credit ledger 必須是 immutable ledger

`CreditLedgerEntry` 至少記錄：

- `id`
- `account_id`
- `organization_id`（nullable）
- `credit_type`（consumer / engine_api）
- `transaction_type`
- `credits`（signed integer 或以 direction 分離，規則全系統一致）
- `source`、`source_id`、`purchase_id`、`answer_id`、`reservation_id`
- `batch_id`、`expires_at`
- `idempotency_key`
- `created_at`、`created_by`（system/user/admin）

Transaction types 至少包括：`PURCHASE`、`BONUS`、`FREE_CREDIT`、`SUBSCRIPTION_GRANT`、`RESERVE`、`RELEASE`、`CONSUME`、`EXPIRE`、`REFUND`、`ADJUSTMENT`、`CHARGEBACK_REVERSAL`。

可維護 cache balance 以加快讀取，但 ledger 才是審計真相；週期性 reconciliation 必須核對 cache balance 與 ledger 聚合結果。

### 17.5 Event / audit

至少發出／保存：

- `answer.estimated`、`answer.confirmed`、`answer.started`、`answer.stage_completed`、`answer.completed`、`answer.failed`、`answer.cancelled`。
- `credit.reserved`、`credit.released`、`credit.consumed`、`credit.expired`、`credit.granted`。
- `payment.succeeded`、`payment.failed`、`payment.refunded`、`payment.disputed`。
- `room.monitor_started`、`room.hint_emitted`、`room.answer_requested`、`room.document_granted`、`room.document_revoked`。
- `provider.failed`、`provider.fallback`、`routing_policy.changed`、`pricing_config.changed`。

所有涉及金額、權限、文件 grant 或 Credit 的事件都需可去重、可追蹤與可稽核。

---

## 18. Credit 計費總體設計

### 18.1 Credit 定義

- Credit 是「智慧工作用量」的抽象單位，不是「一題問題」、不是「一次 provider call」，也不固定代表某個 token 數。
- 一個 answer 可使用多個 provider、工具、查核與合成步驟；不同任務的內部成本差距很大。
- 一次工作真正可收費多少，取決於 consumer/API product、定價版本、深度、quote、確認上限及實際可結算工作量。
- Consumer Credits 與 Engine API Credits 必須分成不同 ledger、balance、pack catalog、usage display 和 pricing conversion。除非未來有明確定義的轉換政策，否則不可轉帳或相互兌換。

### 18.2 Reservation → Execute → Settlement

```text
Preflight quote
    ↓
User confirmation / API budget authorization
    ↓
Atomic reserve (ledger + available balance)
    ↓
Execute within reserved hard cap
    ↓
Measure actual provider/tool workload
    ↓
Calculate authorized charge
    ↓
CONSUME actual amount + RELEASE unused amount
    ↓
Store reconciliation and final usage receipt
```

### 18.3 不可妥協規則

- 未成功 reservation 不得開始付費工作流。
- 同一個 `answer_id` 的 confirm 重試不可以重複 reserve 或重複執行工作。
- 不可以超過用戶確認的最大 Credits；如果需要額外預算，必須重新 quote 和明確確認。
- `reserved`、`consumed`、`released` 分開記錄，不能只保存最後餘額。
- provider retry/fallback 不可建立重複收費；實際成本可增加，但 consumer 收費仍受確認上限約束，超出的內部成本需由平台預算保護／吸收或在下一步前停止。
- 所有支付 webhook、Credit issuance、subscription grants 和退款流程皆需 idempotent。
- 使用 FEFO（First Expire, First Out）消耗批次；不同種類 Credit 須有清楚優先順序。
- 取消、失敗、部分完成和退款要有明確狀態；不得把付款成功等同於答案執行成功。

### 18.4 FEFO 與消耗順序

建議在同一 credit type 內，依 `expires_at ASC` 消耗最早到期批次；同一到期日則依發行時間。Consumer Credits 與 Engine API Credits 不混在一個消耗序列。若有贈點與購買點不同的消費者權益，請用 `batch_type`／`expires_at` 表達，而不是由前端指定要扣哪一種。

### 18.5 退款與已完成答案

- 支付退款先處理付款／Credit ledger，而不是直接覆寫 balance。
- 若該批 Credits 已使用，政策需定義是否允許退款、部分退款、負餘額或 future balance adjustment；不可由工程師自行選擇。
- Provider failure 若未產出可用答案，應依失敗類型釋放未使用 Credits，或退回已收費部分；必須保持同一種政策並在 UI 說明。
- 使用者對答案不滿意不自動等同於付款失敗；可提供 Try Again，或由明確退款政策處理，不應讓評分按鈕直接修改帳務。

---

## 19. Contact Answer Consumer 定價

### 19.1 Credit packs（採用最新補充版本）

| 付款金額 | 發放 Credits | 基礎 Credits | Bonus | 每 Credit 有效價格 | 標籤 |
|---:|---:|---:|---:|---:|---|
| $5 | 20 | 20 | — | $0.250 | — |
| $10 | 40 | 40 | — | $0.250 | — |
| $20 | 88 | 80 | +10% | $0.227 | Value |
| $50 | 240 | 200 | +20% | $0.208 | — |
| $100 | 520 | 400 | +30% | $0.192 | Popular |
| $200 | 1,120 | 800 | +40% | $0.179 | — |
| $500 | 3,000 | 2,000 | +50% | $0.167 | Max |

此表取代舊 HTML 中 `$5 = 5 Credits`、`$10 = 10 Credits` 等舊方案。Bonus 百分比相對基礎 Credits 計算；有效價格是付款金額除以總發放 Credits，兩者不是同一個百分比。

### 19.2 定價配置

- 以 `consumer_credit_packs` 中央配置保存 amount、currency、base credits、bonus credits、label、Stripe Price/Product ID、available countries、active dates、version。
- 前端只讀 server-side public pricing config，不在頁面手寫多份不同價格。
- 訂單建立時保存該筆交易使用的 pack 和 pricing version，確保未來價格修改不改寫歷史交易。
- `$5` 為目前最低公開購買檔；不在 MVP 另外新增 `$1` 或 `$1,000` pack，除非商業測試明確決定。
- Auto-recharge 只能購買明確允許的 pack／自動充值商品，不可無限制地使用客戶的 payment method。

### 19.3 深度與 Consumer Credits

目前 Answer 初始深度區間：Verified 1–3、Deep 5–10、Research 20–50。實際消耗量由 quote 預估，確認後不超出該額度。新價格有效收入比舊版本低，因此正式上線前必須以最低有效收入的 pack（$500 pack 的約 $0.167/Credit）壓測每個層級的最小可接受品質及 COGS。若無法達成，不得暗地讓 Verified 變成單模型回答；應調整價表、區間、最小服務範圍或 Bonus。

---

## 20. Contact Engine API 定價

### 20.1 API Credit packs（獨立於 Consumer Credits）

| 付款金額 | 發放 API Credits | 基礎 API Credits | Bonus | 每 API Credit 有效價格 | 標籤 |
|---:|---:|---:|---:|---:|---|
| $25 | 500 | 500 | — | $0.050 | — |
| $100 | 2,000 | 2,000 | — | $0.050 | — |
| $500 | 11,000 | 10,000 | +10% | $0.045 | Value |
| $2,000 | 48,000 | 40,000 | +20% | $0.042 | — |
| $5,000 | 130,000 | 100,000 | +30% | $0.038 | Popular |
| $10,000 | 280,000 | 200,000 | +40% | $0.036 | — |
| $25,000 | 750,000 | 500,000 | +50% | $0.033 | Max |
| Enterprise | Custom | Custom | Custom | Contract | Contact Sales |

### 20.2 API 與 Consumer Credit 分開處理

- Consumer pack 是 Contact Answer 使用者購買；Engine API pack 是開發者為伺服器端 API usage 購買。
- 不共用 balance，不共用 pack ID，不共用 expiration rule，不共用 conversion formula，不互相轉換。
- 所有 API usage 回報 `credit_type = engine_api`，避免被誤寫進消費者 Answer balance。
- API 客戶支出上限、rate limit、API key scope、帳單和 credits ledger 均需在 Engine developer account 管理。
- Public API 允許每個 operation 有不同 API Credit 配置，但操作成本必須以中央 policy 管理；不能把 Consumer UI 上的 1–3 / 5–10 / 20–50 原封不動等價套用到 API Credits。

### 20.3 API 費用可計算性

每次 API response／usage record 至少要能回答：用了多少 API Credits、操作為何、request ID、execution status、時間、是否 partial、該 API account 的剩餘餘額。不得將 provider 的真實供應商費率暴露為必要的 public field；可以提供標準化 token／tool usage（依服務契約）。

---

## 21. 免費額度與 Contact 訂閱權益

### 21.1 Free 用戶月額度（最新補充）

目前補充文件提出的暫行試驗政策：

- Free 用戶每月最多獲得 **6 Consumer Credits**，用作免費試用 Answer 的額度上限。
- 初始策略為每 5 天恢復／發放 1 Credit；同一月份內未使用的額度可以累積。
- 月末將該月 Free allowance 未使用餘額清零，不累積到下個月。
- 這是一項需要以回訪率、Verified Answer 使用率、首購轉換率和免費用戶實際 API 成本驗證的成長實驗；應以中央配置控制，不能寫死在業務程式。

為避免時區與月底產生不同結果，建議使用伺服器端定義的 `free_allowance_period`（預設 UTC calendar month）與可審計的 schedule：每個五日週期發放 1 Credit，上限 6／月；月底只清除本月尚未使用的月度 allowance batch，不能刪除購買 Credits 或仍有效的其他批次。具體起始日、月底規則和第一週的 partial window 必須在 implementation config 中固定並寫測試。

**最新月額度政策取代舊 HTML 的 Free one-time 1 Credit 與 Free plan 2 Credits 規則。不得將三種免費發放疊加。**

### 21.2 Contact 訂閱附帶 Answer Credits

舊規格還定義了 Contact 訂閱作為 AI Draft benefit 的附帶 Answer Credits：

| Contact plan | 建議附帶 Consumer Credits |
|---|---:|
| Free | 由最新免費月額度政策決定：6／月；不再另外發舊版 2 Credits |
| Pro | 20 Credits／subscription grant period（先沿用既有方案值） |
| Premium | 50 Credits／subscription grant period（先沿用既有方案值） |

Pro / Premium 的發放週期要跟 Contact 訂閱的實際 entitlement／billing cycle 同步，配置化處理，不可每次登入發一次或每次 webhook 重送就重複發點。若既有 Contact 商業方案的實際週期不是每月，應由 subscription entitlements config 決定；Engineering 不可自行將它猜為每日／每週。

### 21.3 購買 Credits 與贈點到期規則

- Purchased Consumer Credit batch：沿用一般政策，發放後 12 個月到期，FEFO 消耗。
- Subscription-granted Credits：每次發放一個獨立 batch，初始沿用 12 個月有效期／FEFO；如果 Contact 訂閱政策要求 billing-cycle end 過期，需在 entitlement config 明確變更。
- Free monthly allowance batch：月底過期、不跨月累積，優先於較晚到期的點數被消耗。
- Engine API Credits：屬獨立商品；建議採用 12 個月批次到期與 FEFO 作為初始預設，若企業合同規定不同則以合同 policy 覆蓋並保留 audit。
- 歷史 issuance 記錄不得因新政策重寫；只對新發放的 batch 使用新 policy。

### 21.4 反濫用

- 需要經過基本 email/account verification 才能取得可轉換為商業成本的促銷額度。
- Rate limit、異常請求模式、重複帳號／promotion abuse 風險評估。
- 裝置/IP 指標應以比例原則使用，不因共享公司網絡或公共 Wi-Fi 便直接封鎖正常用戶。
- 不使用前端 local storage 決定使用者是否可領取免費 Credits。
- anti-abuse decision 要可被回查，避免用戶因錯誤風控被永久失去應得餘額。

---

## 22. Reservation、Settlement 與退款

### 22.1 保留額度和結算

1. Quote service 先計算使用者可理解的 estimated credits 和該次工作 authorized cap。
2. 用戶確認／API account authorization 後，以資料庫 transaction 或等效的原子操作建立 reservation。
3. 同時驗證 available balance、有效 batch、租戶付款權限與 idempotency key。
4. Workflow 開始後，執行器需要在每個 stage 記錄 usage 與預算剩餘量。
5. 工作完成／部分完成／失敗時，billing service 依已授權用量結算；未使用部分 RELEASE。
6. Reservation 設定 timeout／lease；若 worker crash，reconciler 必須查明 job 狀態再決定繼續、釋放或結算，不能直接全扣或一直凍結。

### 22.2 原子性與冪等性

- Credit reserve／consume／release 要使用 transaction、row-level lock、compare-and-swap 或其他可驗證的原子機制。
- 同一 idempotency key 的重送應回傳相同交易結果。
- webhook 至少使用 provider event ID 去重；不能靠延遲假設事件只到一次。
- 同時多個 Answer request 不得花掉相同的一份 available balance。
- `available_balance = valid_issued_credits - settled_consumption - active_reservations - expired_or_revoked_credits` 的計算規則須統一，實際 SQL 需確保 reservation 與 FEFO 正確。

### 22.3 追加預算

- 不可以在用戶已確認 2 Credits 後，因模型多講了 token 或需要追加搜尋而靜默扣 3 Credits。
- 若下一輪 computation 需要超出目前預算，停止新增昂貴工作；可以以當前可用內容完成一份較有限答案並說明限制，或回傳 `BUDGET_EXCEEDED` 讓 UI 再次估價確認。
- 用戶在第二次確認前不能被額外扣點。

### 22.4 失敗和部分完成處理

- Provider timeout：對 transient errors 進行有限 retry；使用 fallback 時仍在同一個 authorized budget 內。
- 部分 provider 成功且可產出具價值結果時，可 `PARTIAL_COMPLETED`；其實際扣點需依明確的 settlement policy，並在 UI 告知部分驗證。
- 若完全無可用結果且答案不可交付，建議釋放全部未使用 Credits；實際已消耗部分是否扣點需按已公布的 failure policy 判斷，不可以由 provider 例外處理程式臨時決定。
- 結算服務故障時將答案標為 `SETTLEMENT_PENDING`，重試 settlement 而不是再次執行 provider calls。

---

## 23. Auto-recharge 與支付整合

### 23.1 UI 與規則

Answer Billing settings 顯示 `Auto-recharge`：

> Automatically add $10 when your balance falls below 2 credits.

- Auto-recharge 為 opt-in，不可默認替新用戶打開。
- 使用者需有有效授權的付款方式。
- 需在帳戶提供啟用／停用、當前狀態、最近充值結果與支出安全限制。
- 當 Consumer Credit 可用餘額低於 2 時，若功能已啟用，系統可觸發一次 $10 的 Consumer pack recharge（依現行 pack 配置發點）。
- Auto-recharge 需具備單次觸發鎖、最低間隔、每月／每週支出 cap、付款失敗告警及重試規則。

### 23.2 支付服務要求

- 使用 Stripe 或選定 PSP 的 server-side Checkout / PaymentIntent / SetupIntent 流程；前端只處理 provider 提供的安全 token / element。
- 不儲存或記錄完整卡號與 CVC；**CVC 不可被系統儲存或顯示，即使以 `***` 表示也不要建立自己的 CVC 欄位。** UI 僅可顯示付款品牌、last4、到期月份／年份與 PSP 託管的更新流程。
- Credits 只能在已驗證的付款成功 webhook 後發放。
- webhook signature 必須驗證；所有 event 需去重並存原始 event ID、processing status 與處理結果。
- checkout 成功跳轉不能獨自作為發放 Credits 的依據；需有 server-confirmed payment state。
- 退款、爭議、chargeback 和 payment method update 需影響帳務 ledger 並產生 audit event。
- 自動充值不可因多個並發 Answer request 同時偵測餘額不足而重複扣款；需以 account-level lock 或原子 job claim 避免 duplicate recharge。

### 23.3 Billing History

每一筆記錄包括 Date、Service／Product、Amount、Currency、Status、Receipt／Invoice download link、order ID。Invoice／receipt 連結須驗證用戶與付款帳戶權限，不得使用可被猜測的公開文件 URL。

---

## 24. Answer 專屬 Settings

> **範圍提醒：本章 Settings 是 `answer.contact` 產品專屬設定，不是只針對 Contact Engine 的開發者控制台，也不是整個 Contact Dash 的全站設定。** Answer 可預設繼承 Dash 通用設定，但允許在 Answer 內獨立覆寫。

### 24.1 Settings 導覽

```text
answer.contact/settings

General   Preferences   Billing   Account
```

### 24.2 General — `/settings/general`

以文字／表單結構實作以下設定：

```text
# General

- Appearance -
✓ System
  Dark
  Light

- Theme -
✓ Blue
  Teal
  Green
  Yellow
  Orange
  Red
  Pink
  Purple
  Brown
  Slate

- Language -
✓ English
  Español
  ...

- Currency -
USD — US Dollar
EUR — Euro
GBP — British Pound
CAD — Canadian Dollar
AUD — Australian Dollar
CHF — Swiss Franc
JPY — Japanese Yen
SGD — Singapore Dollar
KRW — South Korean Won
TWD — New Taiwan Dollar
HKD — Hong Kong Dollar
NZD — New Zealand Dollar
```

要求：

- Appearance、Theme、Language、Currency 預設繼承 Dash 的設定。
- 每項需支持 `inherit` 與 Answer 本地 override。使用者改動 Dash 設定時，只更新仍處於 inherit 狀態的項目；使用者已在 Answer 覆寫的項目保持不變。
- Currency 主要控制價格顯示格式／顯示幣別。若實際扣款仍以 USD 結算，需清楚表示匯率換算為顯示估算，並標明實際支付幣別；不可讓用戶誤以為改顯示幣別就改變 Stripe 訂單幣別。
- Theme 的視覺 token 集中管理，不能為每個頁面自行定義不同藍色或暗色背景。
- Language 套用 UI、系統文案與生成答案的預設語言；若使用者在單次 prompt 明確指定其他語言，該次指令優先於預設。
- 可用語言／貨幣清單從中央配置載入；`...` 表示未來可擴展，並非可在目前直接存取不存在的 locale。

General 說明文字：

> If Appearance, Theme, Language, or Currency has been configured in Dash, the settings will be inherited by default. However, these settings can be configured independently in each interface.

### 24.3 Preferences — `/settings/preferences`

這是 Answer 的核心互動偏好，不應只有常見的文字風格設定；它也包含 Answer 的低摩擦 Follow-up 交互。

```text
# Preferences
answer.contact/settings/preferences
────────────────────────────────

- Interaction -
✓ Follow-up Suggestions

- Response Style -
✓ Balanced
  Professional
  Friendly
  Direct
  Academic

- Response Format -
✓ Automatic
  Structured (Headings & Lists)
  Plain Prose

────────────────────────────────
* Preferences apply by default to new answers. Users can override them in individual discussions.
```

實作要求：

- `Follow-up Suggestions` 預設開啟，可切換。開啟代表在有價值時生成並顯示回答後的點選建議；關閉代表不生成專門 suggestions 的額外工作，不只是在 UI 隱藏。
- `Response Style`：`balanced`、`professional`、`friendly`、`direct`、`academic`。
- `Response Format`：`automatic`、`structured`（Headings & Lists）、`plain_prose`。
- 套用於新 Answer，不能自動改寫已完成的歷史答案。
- 每個 discussion / Room 可有局部 override；局部設定與使用者全域設定分開儲存。建立 Answer 時保存 preference snapshot，讓歷史答案能解釋當時使用了什麼設定。
- 沒有設定時預設 `balanced` + `automatic` + Follow-up Suggestions enabled。
- Preferences 不得成為對 Answer 的可靠性或安全行為的控制開關；Response Style 不可以移除必要限制與 uncertainty statement。
- 不在 MVP 暴露更多內部選項，例如 suggestion 數量、路由權重、分析模型或 token budget；由 Engine 智慧處理。

### 24.4 Billing — `/settings/billing`

主要資訊結構：

```text
# Billing

- Current Credits -
3,000
You will be charged automatically.
[Recharge]
View pricing

Auto-recharge
Automatically add $10 when your balance falls below 2 credits.
[Enable / Disable]

- Payment method -
Visa **** 4242
Expires 12/27
[Update] [Remove]

- Billing History -
Date | Service | Amount | Status | Invoice (Download)
```

工程要求：

- Balance 顯示來自 server-side consumer ledger，不可保留硬編碼 `3,000` 作正式值。
- `You will be charged automatically` 只在 auto-recharge 真正啟用且有有效付款方式時顯示；否則顯示符合實際狀態的訊息。
- `[Recharge]` 開啟 Consumer Credit packs；`View pricing` 導向同一套中央定價表。
- 支付卡資訊由 PSP tokenized payment method 提供；不顯示／儲存 CVC。
- `Update` 與 `Remove` 必須透過 PSP 安全流程；移除唯一付款方式前需處理 Auto-recharge 開啟狀態並清楚告知使用者。
- Payment history 從 payment/order ledger 讀取；發票下載需做 authorization。
- 不可將 Engine API Credits 和 Consumer Credits 合併顯示為同一餘額。

### 24.5 Account — `/settings/account`

```text
# Account

Username     jack123
Email        jack@gmail.com

Manage Account >
(Account link, redirect to Contact’s Account Management page.)
```

- username／email 由 Contact identity/account service 提供，不在 Answer 自行維護另一份帳戶真相。
- `Manage Account` 連接 Contact 的 Account Management 頁面。
- 需要登入、權限和適當的 return URL，避免開放 redirect 漏洞。

### 24.6 Preferences 儲存格式示例

```json
{
  "appearance": { "mode": "inherit" },
  "theme": { "mode": "inherit" },
  "language": { "mode": "inherit" },
  "currency": { "mode": "inherit" },
  "interaction": {
    "follow_up_suggestions": true
  },
  "response_style": "balanced",
  "response_format": "automatic",
  "updated_at": "2026-10-20T00:00:00Z"
}
```

實際 schema 可依既有 Contact identity/settings service 改造，但 inheritance 與 Answer override 語義必須保持。

---

## 25. 資料庫設計建議

### 25.1 基本實體

建議最初至少有下列 tables／collections（實際命名由工程師依現有 database convention 決定）：

- `accounts` / `organizations` / `organization_members`
- `answer_requests` / `answer_results` / `answer_versions`
- `answer_executions` / `answer_claims` / `answer_evidence`
- `conversations` / `conversation_messages` / `favorites` / `history_items`
- `rooms` / `room_members` / `room_messages` / `room_monitor_runs`
- `room_document_grants` / `document_metadata` / `document_chunks` / `retrieval_audit`
- `user_preferences` / `discussion_preferences`
- `consumer_credit_accounts` / `engine_api_credit_accounts`
- `credit_batches` / `credit_ledger_entries` / `credit_reservations` / `credit_settlements`
- `credit_pack_catalog` / `orders` / `payment_methods_reference` / `payment_events` / `invoices`
- `api_keys` / `api_usage_records` / `api_customers` / `rate_limit_buckets`
- `provider_registry` / `model_pricing_versions` / `routing_policy_versions`
- `feedback` / `evaluation_runs` / `verified_corpus_records` / `audit_events`

### 25.2 Database principles

- 對會變動／需去重的商務物件提供 UUID/ULID 等唯一 ID 與 created/updated timestamps。
- 金額使用 decimal / integer minor units，不使用 binary floating point 做結算。
- Credits 使用可精確計算的整數單位；如果未來需要 fractional unit 必須作為明確版本化政策，不可在不同 service 裡自行四捨五入。
- 需對 `idempotency_key`、provider event ID、reservation state 建立唯一約束。
- 對大部分 user content 分離 metadata 與加密內容本體，讓資料保留與刪除較容易落實。
- Credit ledger 採 append-only；修正以 reversal／adjustment 記錄，而不是修改既有歷史交易。
- 租戶／Room／文件查詢都要把 permission scoping 放進 server-side query，而不是在 UI 拿到資料後才過濾。

---

## 26. 安全、隱私與多租戶

### 26.1 認證與權限

- Answer web 使用 Contact account/session 認證；Server 端驗證每個 answer、conversation、room、file 和 billing resource 的 owner／member 權限。
- Engine public API 使用獨立 API key、account scopes、可撤銷 key、可設定配額；key 只可存於 server-side secure storage。
- Organization 的 Credit 使用必須檢查 member role、organization active status、shared balance permissions 和可用支出限制。
- Room requester 不能透過修改 JSON 將 payer 改為 owner 或提高 allowed depth。
- 管理員更改價格、模型 registry、routing policy 和 Credit balance 必須有 RBAC 與 audit trail。

### 26.2 Secrets 與付款安全

- Provider API keys、Stripe secrets、DB secrets 存 secret manager/environment secret，不可放入 repo、client bundle、HTML、log 或例外訊息。
- 定期輪換 key；撤銷失效 API key 後必須立即拒絕請求。
- 驗證支付 webhook signature、timestamp／重放防護及 event ID。
- 不儲存 CVC；信用卡資料由 PSP tokenization 管理。

### 26.3 敏感資料與 prompt injection

- Room 訊息、上傳文件、外部 web page 和模型輸出都是不可信資料，不得覆蓋 Engine system policy。
- 對文件和 web 內容做 prompt injection 抵禦：分隔引用內容、來源標示、工具權限最小化、禁止外部內容直接觸發付款／外部動作。
- 避免送出不必要的 PII 到 provider；若 provider 支援區域、zero retention 或不訓練條件，按產品 privacy policy 配置。
- 使用者在 Room 分享文件時，需清楚告知它可能被用作當前 Room 的 AI 參考上下文。
- 不預設拿 Private Answer、Room 訊息、Drive 文檔或附件訓練 Contact 自有模型。未來訓練／Corpus 需用明確授權、最少化、刪除／撤回政策與資料來源 provenance。

### 26.4 Logging 與資料保留

- 一般日誌以 ID、stage、token counts、status、error code、cost 與 latency 為主，不寫入完整私有 prompt／文件內容。
- Debug sample 如需保留輸入／輸出內容，需有嚴格取樣、權限、加密、保留期限與敏感資訊遮蔽。
- 用戶要求刪除資料時，分開處理產品內容、可刪除的 provider prompt data、向量索引、cache、備份保留和法律要求保留的 billing/audit records；不可不加區分地承諾即時物理刪除全部備份。
- 對文件的每次 retrieve／citation 都需要帶 permission check；不能依靠「它曾經加入 Room」作永久存取權。

---

## 27. 可靠性、延遲與限流

### 27.1 Provider reliability

- 所有 provider call 設 timeout，並區分 transient／permanent／policy／quota／invalid request errors。
- 對 429、temporary server errors 和網絡失敗採 bounded exponential backoff + jitter；不能無限重試。
- Retry 需計入預算，並且不重複建立 Credit settlement。
- Provider 有持續錯誤時啟動 circuit breaker，暫時從候選池移除，定期 half-open probe。
- Fallback 需維持同一工作目標與符合用戶授權上限；fallback 不可隱藏品質降級。
- 若同時失去多個 provider，回傳 partial／failed 狀態，依 settlement policy 結算，不假稱 Verified 完成。

### 27.2 Latency

- Independent first pass 在可行時並行執行。
- Research 多步工作使用 job queue／streaming progress，而不是令 web request 無限阻塞。
- 前端提供狀態事件：estimating、awaiting confirmation、running analysis、checking evidence、synthesizing、settling、completed／partial。
- 對重複文件檢索、答案上下文摘要和 suggestions 做合理快取；快取需有 permission-aware key、TTL 和 invalidation。
- 需要即時回應的簡單問題優先低延遲路徑，但不能以 latency 為由略掉當前任務要求的關鍵驗證。

### 27.3 Rate limits 與防濫用

- API key、account、organization、IP／風險指標、Room 和 endpoint 可以有不同限額。
- Consumer quote preflight、完整 Answer、Research、Room background monitor 要使用不同成本池與限流策略。
- Room monitor 必須有批次、cooldown、單 Room 及帳戶每日／週內部預算；此為系統成本安全，不展示成可消費的「免費 AI 次數」。
- 對突發高負荷設 queue depth 上限、backpressure 和可恢復的錯誤回應。
- 限流回應提供 `Retry-After`（適用時）與穩定機器碼。

---

## 28. Observability 與商業監測

### 28.1 每次 Answer 必須可觀測

最少記錄：

- `answer_id`、`request_id`、account / organization / room ID（以必要最小化原則）。
- task type、depth requested／recommended／used、complexity score、routing policy version。
- estimate credits、reserved、consumed、released、provider COGS、tools COGS。
- provider/model、purpose、input/output tokens、latency、retry count、status、error code。
- provider families count、challenge rounds、claims checked、evidence checks、unresolved contradictions。
- final status、partial verification reason、feedback result（如有）。

### 28.2 Dashboard

最少可按日期、depth、provider、model、account plan、地域／幣別（依政策）、Room／standalone 分析：

- Mean / P50 / P95 latency。
- Provider error rate、timeout、fallback rate、retry cost。
- Average COGS / Answer、average credits / Answer、gross margin proxy。
- Quote estimate vs actual cost error。
- 各深度的選擇／確認／完成／放棄率。
- Suggestions generation cost、exposure、CTR、follow-through rate。
- Room background monitor cost／提示採用率，但不對一般使用者顯示背景扣點。
- Source click-through／citation failure／evidence support status。
- Free credits usage、回訪、首購轉換、平均每位免費使用者成本。

### 28.3 Alerts

- 預測／實際 COGS 超設定目標。
- 高錯誤率／provider outage／fallback spike。
- Quote cost calibration error 超閾值。
- Credit ledger reconciliation mismatch。
- Payment webhook backlog 或簽章錯誤。
- Auto-recharge 重複觸發或異常充值。
- Room background monitor 超過設定的內部預算。
- 未授權 Drive 文檔出現在 retrieval results。

---

## 29. Feedback、Verified Corpus 與長期護城河

### 29.1 演進方向

Contact Engine 的長期優勢不應只依賴租用 OpenAI／Claude／Gemini。應逐步形成自己的高品質答題與路由資料資產，但要以可信 provenance、使用權、正確性檢查與隱私為前提。

### 29.2 演進階段

**Stage 1 — Rule-based + model-assisted routing**

- 用既定規則、低成本 classifier、模型 registry、明確 cost cap 執行。
- 建立 cost／quality eval dataset 和 provider usage telemetry。

**Stage 2 — Learned routing**

- 用已經評估的任務類型、實際成本、provider 延遲、失敗率、獨立評測和用戶回饋訓練路由預測器。
- 在 shadow mode 與 A/B 測試先驗證，保持人工設定的 budget／quality guardrails。

**Stage 3 — Verified Answer Corpus / safe reuse**

- 儲存通過來源查核、人工抽樣或可驗證 outcome 的主張／答案，帶 provenance、語境、時間、適用條件、信心等 metadata。
- 對完全重複且不需要時效更新的問題，可重用已驗證材料來減少模型成本；涉及當前事實、法律、價格或資料已過期時必須重新查核。
- 不可只靠模型彼此同意就讓答案進入 trusted corpus。
- 對每個可重用紀錄設 TTL、品質等級與 invalidation。

**Stage 4 — Contact-owned intelligence components**

- 由高品質評估資料逐步建立自有 classification、claim checking、routing、retrieval reranking、follow-up intent 和 synthesis helper models。
- 新模型先在離線 eval、red-team、cost comparison 和 shadow traffic 取得結果後才逐步上線。
- 不把個人私有內容默認用於訓練；對用戶或組織提供適當的 data control。

### 29.3 Answer feedback 不等於 truth label

`Helpful`、長期採用或點擊某一方案是有價值的訊號，但不等同於事實正確。Verified Corpus 需結合權威來源、可驗證結果、人工抽查、來源新鮮度與明確的資料授權。任何模型自動標註都要設品質 gate。

---

## 30. 管理設定與功能開關

所有以下配置應放在有版本、權限與 audit 的後端設定服務／資料表中；前端只讀取公開欄位，Engine 讀取完整內部配置。

### 30.1 Answer depth config

- 各 depth 的可見名稱、Credit min/max、最大允許 provider calls／tool calls／token budget。
- 最低 provider families、是否要求 evidence、最大 challenge rounds。
- Quote TTL、預算停止方式、partial completion policy。

### 30.2 Model / provider config

- Provider enable/disable、model IDs、capabilities、token price、timeout、retry、fallback order。
- Provider family 與 model tier（economy / balanced / frontier / specialist）。
- 每個任務可用的模型候選集。
- 地域／資料政策限制。

### 30.3 Cost / credit config

- Consumer pack catalog 與 Engine API pack catalog。
- `max_provider_cogs_ratio`、cost buffer、各 depth budget cap、minimum quote。
- Credit batch expiration／FEFO rules、free allowance cadence、subscription grant mapping。
- Auto-recharge amount／threshold／cooldown／monthly cap。
- Pricing version、FX display config、tax / PSP fee assumptions。

### 30.4 Feature flags

- `follow_up_suggestions_enabled`
- `room_ai_participation_enabled`
- `room_drive_references_enabled`
- `evidence_search_enabled`
- `provider_{name}_enabled`
- `learned_router_shadow_mode`
- `verified_corpus_reuse_enabled`
- `auto_recharge_enabled`

Feature flag 需可按環境／帳戶／比例逐步 rollout，支援快速 kill switch 與 audit。

---

## 31. 測試與驗收標準

### 31.1 Routing / cost tests

- 同一個任務、同一配置可重現相同 routing policy decisions（除明確的動態 provider availability 外）。
- 簡單、複雜、Research、時效性與高風險樣本能合理分流。
- Classifier 無信心時不會一律選最低成本路由。
- Cost estimator 對 token、tool、retry 與 cache usage 的估計具有可量測誤差指標。
- 修改模型價格／Credit pack／cost ratio 不需改動前端或重新部署應用程式核心邏輯。
- 預算到達上限時不會追加未授權計算或收費。

### 31.2 Verification tests

- 初始模型分析盡可能獨立，不先共享彼此結論。
- 測試集中的實質矛盾會被偵測；單純措辭差異不被錯誤標記為矛盾。
- 缺乏證據時系統能標示 insufficient / unresolved，而不是捏造 citation。
- Provider 失敗時呈現 partial status；不宣稱實際未完成的 provider counts 或 evidence checks。
- 重要來源 URL、頁碼、文件版本可以回查。
- 最終 synthesis 能保留 unresolved disagreements，而不是強制所有模型一致。

### 31.3 Credit / payment tests

- Quote 未確認時不執行完整付費 workflow。
- 餘額不足不會開始付費執行。
- 同時提交多個請求不會造成負餘額或重複 reserve。
- 同一 confirm request／webhook event 重試不會重複扣點／發點。
- 成功、失敗、部分完成、取消、worker crash 與 settlement retry 都能正確處理 reserve／consume／release。
- FEFO 正確消耗最先過期 batch。
- Free allowance 每五日累積、最多六枚、月末到期的邊界時間有明確測試；跨月、UTC 時區和重複 scheduler event 不會多發點。
- Consumer 與 Engine API credits 無法被錯誤互用。
- Auto-recharge 並發時最多建立一次有效充值；付款失敗不發 Credits。
- CVC 不儲存、不顯示、不落入 logs。

### 31.4 Answer Preferences / UI tests

- Preferences 預設 Follow-up Suggestions enabled、Balanced、Automatic。
- 關閉 Follow-up Suggestions 時不會呼叫 suggestions 專用模型路徑。
- 建議選項與當前答案相關，點擊後建立可審計的 follow-up user turn；不點擊不會執行。
- 自由輸入始終可用。
- Response Style／Format 對新 Answer 生效，discussion override 有優先權，歷史答案保留當時的 snapshot。
- General 設定仍在 inherit 時跟隨 Dash；本地 override 不因 Dash 更改被覆蓋。
- Billing 顯示實際 balance、auto-recharge 狀態、支付資料、發票權限；不出現硬編碼 3,000 或 CVC。
- Account 管理頁連結使用安全 return URL。

### 31.5 Room / Drive tests

- 普通 Room 成員聊天不扣 Answer Credits。
- Background monitor 不產生完整答案、不按每則訊息重做昂貴 workflow，並有內部 cost cap。
- 背景提示中性、可選且有冷卻；AI participation 關閉後不再執行 monitor。
- `payer=owner` 只從 owner balance reserve；`payer=requester` 只從 requester 的正確 balance reserve。
- Room 只檢索仍有權限的文件；撤回／移除後新 Answer 不能使用該文件。
- 任何 Room 成員可以授權自己的文件，但無法藉此公開整個 Drive 或刪除原始文件。
- Citation 可指到檔案／頁碼／章節；prompt injection 內容不會改變 system policy。

### 31.6 上線驗收（Definition of Done）

- 使用者可以安全購買 Consumer Credits、看到正確 balance 並查看交易記錄。
- 使用者提交問題時 Engine 推薦深度並顯示明確預估，必須確認後才啟動付費 Answer。
- Engine 可以調用 OpenAI、Claude、Gemini 的統一 provider interface 執行獨立分析、challenge、矛盾檢查、必要的 evidence verification 和 synthesis。
- Credits 依 confirmed cap 原子 reserve、settle、release，所有 transactions 可稽核。
- Suggestions／Preferences／Room payer／Drive permissions 符合本文件。
- 主要 failure modes 有測試，無重複收費、無未授權超收、無跨租戶資料洩漏。
- Dashboard 可計算每個深度的估算成本、實際成本、延遲、品質與毛利指標。
- Consumer pricing、API pricing、模型 ID、Credit grant policy 與路由配置可無需修改程式碼核心而調整。

---

## 32. 分階段交付計畫

### Phase 0 — 配置、成本與帳務基礎（P0）

- 建立獨立 Consumer / Engine API Credit ledgers、batch、reservation、settlement 與 idempotency。
- 定義中央 model registry、token pricing registry、Credit pack config、depth config。
- 建立 Answer request state machine、quote/confirm API、成本觀測結構。
- Provider key 管理與 payment webhook security。

### Phase 1 — Engine MVP（P0）

- OpenAI、Claude、Gemini provider adapters。
- Rule + lightweight classifier，複雜度建議與 cost estimator。
- Verified / Deep / Research workflows。
- Independent first pass、基本 challenge、矛盾辨識、final synthesis。
- Adaptive budget stop、provider fallback、usage telemetry。

### Phase 2 — Answer product (P0)

- Standalone composer、depth choice、quote confirmation、answer result、History、Try Again。
- Follow-up Suggestions 與 Preferences。
- Billing pack purchase、balance、history、Auto-recharge。
- Answer 專屬 General / Preferences / Billing / Account settings。

### Phase 3 — Room (P1)

- Room roles／members／messages、AI participation background monitor、payer 模式、allowed depth。
- Background batching／cooldown／internal budget cap。
- Get an Answer for This Discussion 共享上下文、quote、confirm、settlement。

### Phase 4 — Drive RAG 與 Research 增強（P1）

- 文件 grant／revocation、permission-aware index、citations。
- Search/source retrieval、claim-to-source map、Research workflow。
- Evidence quality metrics、prompt injection defenses。

### Phase 5 — Learning & external API growth（P2）

- Engine API customer/keys/rate limits/usage pages、API Credit packs、public documentation。
- Routing shadow evaluation、A/B tests、cost quality scorecards。
- Verified Corpus、cache reuse 與有授權的自有 intelligence components。

---

## 33. 產品決策與舊規格調和

為了讓工程師不必自行猜測，本節固定目前規格中的衝突解法。

| 衝突／模糊處 | 最終實作建議 | 理由 |
|---|---|---|
| 舊 HTML Consumer pack `$5 = 5 Credits` 與新補充 `$5 = 20 Credits` | **採新補充表**：$5/20、$10/40、$20/88、$50/240、$100/520、$200/1,120、$500/3,000 | 新補充是最新且明確的 Answer Consumer pricing |
| 舊 Free one-time 1 Credit、舊 Free plan 2 Credits 與新 Free 月 6 Credits | **採新月額度政策**；不再另外疊加舊一次性 1 或舊 Free 2 | 避免同一 Free 用戶重複得到多套舊贈點 |
| 以 1/2/3 Credits 分成三種 Verified micro-options vs 只讓用戶選 Verified/Deep/Research | **只把 Verified/Deep/Research 當主要深度控制**；Engine 在層級內算 quote，顯示這次的確切 estimated Credits | 維持極簡、用戶不必理解算力數值，同時保留成本控制 |
| Verified 是否永遠用三個最昂貴 frontier models | **不強制最貴模型**；以三個 provider families 的適當低成本／中階模型作優先路徑，額外 frontier 呼叫按需要升級；在最低預算／品質門檻無法兼容時須重新校準價格，不得暗中降級 | 保留產品多模型交叉分析差異，又不把昂貴模型固定用在簡單任務 |
| Verified 是否必須三家 provider 都成功才可出結果 | **預設目標三家；最低 degraded path 至少兩個不同 provider families + 明確 challenge，並誠實標示 partial**。宣稱三方驗證只限實際完成三方執行 | 避免供應商 outage 令產品完全不可用，同時不誤導用戶 |
| Consumer Credits 與 Engine API Credits | **分開餘額、ledger、pack catalog、pricing conversion、expiration policy** | 兩者的有效單價明顯不同，不能以相同單位計費 |
| Room background AI 是否扣點 | **不扣用戶 Credits**；由 Engine 的內部 background budget、batching、摘要與 rate limits 控制 | 普通 Room 聊天應免費，只有正式 Answer request 才收 Credits |
| Room 背景 AI 是否每條訊息都產生完整答案 | **禁止**；只判斷是否值得介入，必要時顯示中性提示 | 避免每次訊息都增加昂貴推理成本並以既成事實誘導消費 |
| Settings 是 Dash 全站還是 Answer 設定 | **`answer.contact/settings` 是 Answer 專屬；通用設定可繼承 Dash，並可各自在 Answer override** | 明確符合產品邊界，不混淆 Contact 全站帳戶與 Answer 交互偏好 |
| Follow-up Suggestions 是否只是 UI 固定按鈕 | **按當前上下文產生結構化、有意圖的 suggestions；同輪輸出優先、需要時用低成本路徑** | 這是 Answer 的核心交互差異，不能只顯示固定推薦問題 |
| 付款卡 CVC 欄位 | **不儲存、不顯示 CVC**；只顯示品牌／last4／到期日 | 符合安全實作與支付提供商資料模型 |
| 12 個月到期與 Free 月額度月底清零 | **一般購買批次維持 12 個月；Free allowance 使用獨立 batch 並於月底到期** | 特殊月度 promo policy 不應重寫所有 Credit 的 expiry |
| 使用固定 model-call count 定義 Credits | **禁止**；以動態 workflow cost、depth budget、quote 和實際 usage 來計算 | 不同任務成本差異大，固定呼叫數容易吞噬毛利 |
| 舊 Developer quickstart 沒有明確預算授權欄位 | **更新文件與範例**；生產請求需明確 `execute` / `max_credits`，或由帳戶設定已授權的 server-side auto-execute policy | 防止一個簡單的 API request 意外觸發未設定上限的付費計算 |
| Contact Engine 自有 intelligence models | **分階段建立**：先做好評估、路由與可審計資料；有權限的高品質資料累積後才訓練／上線自有模型 | 避免過早訓練低品質語料或侵犯私有資料權益 |

---

## 34. 名詞表

| 名詞 | 定義 |
|---|---|
| Contact Answer | 面向用戶的 Answer 產品，入口為 `answer.contact` |
| Contact Engine | 模型中立的路由、編排、驗證、研究、合成與用量量測基礎設施，入口為 `engine.contact` |
| Verified Answer | 一般深度，多模型獨立分析和基本交叉驗證 |
| Deep Answer | 更深入推理與針對性 challenge 的深度 |
| Research Answer | 多步證據搜尋、來源查核與綜合研究的深度 |
| Consumer Credit | 用戶在 Contact Answer 中購買／獲得的消費者用量單位 |
| Engine API Credit | 開發者呼叫 Contact Engine API 的獨立用量單位 |
| Quote | 執行前的預估用量與授權上限，具有版本和到期時間 |
| Reservation | 執行前暫時保留 Credits，防止並發超支 |
| Settlement | 工作結束後確定消耗額，並釋放未使用保留額度 |
| Provider family | 如 OpenAI、Anthropic、Google 等模型供應商家族 |
| Independent first pass | 尚未看到其他候選答案前，各模型獨立產生初始分析 |
| Challenge | 對候選答案提出反例、假設檢查、反駁與差異分析 |
| Evidence verification | 查核來源是否可信、及時且直接支持特定主張 |
| Synthesis | 將驗證後資訊形成一個可用答案，保留必要不確定性 |
| Follow-up Suggestions | 回答後顯示、使用者點擊即可發起下一個任務的上下文選項 |
| Room background monitor | 對 Room 最新討論進行受限、低成本的「是否值得介入」判斷，不自動生成完整付費答案 |
| Verified Corpus | 帶來源、版本、品質狀態與權限的已驗證資訊資產，可安全用於未來檢索／重用 |

---

## 最終工程原則

Contact Answer 應保持前台極簡，讓用戶只需說明問題、選擇答案深度、確認預估 Credits，並在回答後透過智慧 Follow-up 選項輕鬆延伸討論。Contact Engine 則負責把複雜的分析、路由、成本控制、驗證、證據和資源計費隱藏在可靠、可觀測、可審計的基礎設施內。

**最重要的交付標準不只是「三個模型能回覆」，而是：在每次工作可控制的成本與授權預算內，Engine 能判斷該用什麼、為什麼用、什麼需要被 challenge、證據是否足夠、何時停止，以及最後究竟消耗了多少 Credits；所有結論、權限、成本和付款都能回溯。**

---

*End of specification — Contact Answer & Contact Engine, version 1.0.*
