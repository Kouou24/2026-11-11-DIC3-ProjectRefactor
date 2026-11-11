# LabSteward — Conversation & Research Planning Notes

> 建議儲存位置：`myplan/LabSteward_Conversation.md`  
> 整理日期：2026-11-11  
> 性質：對話整理與研究決策摘要，**非逐字聊天紀錄**。包含待驗證研究線索，不能取代原始文獻或正式法規。

## 1. 專案定位

- **英文題目**：LabSteward: A Compliance-Aware Domain AI Agent for University Laboratory Asset Management and Procurement
- **中文題目**：LabSteward：具法規感知能力之大學實驗室財產管理與採購領域 AI Agent
- **核心構想**：整合 AI Agent、實驗室財產管理、採購法規、數據分析與人工審核，建立可追溯、可驗證的管理／採購輔助系統。
- **研究問題**：結合領域知識、確定性工具與人工審核的混合式 AI Agent，是否比通用 LLM 更可靠地完成受法規約束的實驗室財產管理及採購任務？
- **研究原則**：AI 輔助決策、不自行核准採購；法規判斷、金額計算與重要例外由可測試規則及人員處理。

## 2. A–F 提案敘事

### A. Attention Getter｜動機與引題

研究設備有購置成本高、使用率不透明及跨實驗室共享不足等問題。先前討論列出兩項可作為論證的**待原始來源核對**案例／調查：

- 台灣監察院曾調查中央研究院貴重儀器的使用效益與管理情形；討論提及「252 台、約新臺幣 32.78 億元」。此為歷史案例，不能當作 2026 年現況。
- 2026 年 McKinsey 採購主管調查，討論提及「80% 計畫增加 AI 投資、55% 仍處試點、2% 大規模導入」；不同問項的定義及樣本應核對原文。

論述重心：從研究設備利用效率與採購 AI 趨勢，導向「可靠且可問責的領域 Agent」需求。

### B. But｜挑戰與研究缺口

1. **資訊斷裂**：財產清冊、使用狀態、共享借用與採購流程缺乏串接。
2. **價值不透明**：帳面折舊不等於設備的市場及使用價值。
3. **先買再說**：新購需求不一定先排查可用庫存與替代設備。
4. **法規與程序複雜**：不能讓 LLM 單獨做合規判定。
5. **Agent 穩定性**：通用 LLM 工具調用有幻覺、漏步與多次執行不一致的風險。

之前討論提及 τ-bench 原始實驗中部分情境的 `pass^8 < 25%`，應明確標註模型／情境／論文版本，不可當成所有 Agent 的一般成功率。早前提案提及「32% 幻覺稅」但缺乏足夠可驗證定義，**不建議直接引用**。

**Research Gap 的謹慎表述**：既有研究分別探討資產管理、AI 採購、RAG、Agent 可靠性或人工監督；本計畫旨在整合「台灣大學法規適配 × 庫存優先 × 雙軌殘值 × 可驗證代理流程」，並用實驗評估效益。**不要聲稱世界首創**。

### C. Cure｜解決方案

提出 **LabSteward**：一套具領域知識及法規感知能力的實驗室財產管理與採購 AI Agent。

- 先盤點、查詢與共享／借用，再評估新購。
- LLM Planner 負責需求理解、步驟規劃及工具協調。
- SQLite、Entity Matching、分析模型等確定性／專業工具負責查找和計算。
- RAG 提供法規原文與來源；Rule Engine 負責可執行的合規檢核。
- Human-in-the-loop 處理高風險、例外與正式核准。
- 產出具有來源、檢查紀錄的 Word／Excel 建議或送審文件。

### D. Development｜方法

**八階段概念架構**：

1. User Scenario（使用情境）
2. Query / Input（需求輸入）
3. Planner & Reasoning Agent（規劃及工具調用）
4. Data Acquisition / Retrieval（財產／報價資料擷取）
5. Analytics Engine（估值、比對、比價）
6. Domain Knowledge RAG（法規與領域檢索）
7. Decision Guardrail（規則檢查與人工審核）
8. Output & Interaction（報告、文件與互動）

> 此為概念分工，執行時可反覆查詢與回退，不需固定走完所有步驟。

**規劃技術**：Python、LLM Tool Calling／ReAct、SQLite、FAISS + BM25、Entity Matching、Playwright、Streamlit、python-docx、openpyxl。

**雙軌殘值（M3）**：

- 帳面價值：依財產／會計規範採直線折舊或適用計算方式。
- 市場估值：探討 log-linear 回歸、Gradient Boosting 與 Bootstrap 區間估計。
- 殘值是輔助決策資料，不等於核定的報廢價值或實際成交價。

**採購流程**：輸入需求 → 缺漏資訊追問 → 優先查庫存 → 有可用設備則建議共享／借用 → 否則蒐集並比對有效報價 → 判斷適用採購程序 → Guardrail → 人工覆核（必要時）→ 文件輸出。

**注意**：「至少三筆報價」應依採購案與校內規定設定，不可概括為每件採購的法定要求；金額門檻亦須核對適用機關及最新公告。

### E. Experiments｜評估

計畫以**去識別化財產資料及合成資料**建立 **50 個測試情境**。建議均分五類（每類 10 題）：

1. 庫存查詢與共享借用
2. 殘值及折舊估計
3. 報價、規格與實體比對
4. 採購法規及流程
5. 跨模組異常與失敗復原

**比較基線（Baselines）**：

- B0：人工 Excel／現行人工流程
- B1：單一 LLM（限制工具）
- B2：純 RAG／檢索增強問答
- B3：完整 LabSteward

**評估指標**：Task Success Rate、pass^k、合規違規率、錯誤放行率、殘值 MAE／MAPE、文件／證據引用正確率、處理時間及人工轉介率。

**消融實驗（建議）**：移除 Guardrail、Inventory、RAG 或 Human Review，評估各模組對錯誤率、穩定性與效率的影響。不可只看表面成功率，也要觀察因人工轉介而產生的工作量。

**可能異常案例**：名稱相似但規格不同、已折舊完畢但仍可使用、資料過期、異常低報價、接近金額門檻、疑似拆單、法規來源衝突、網站讀取失敗。

### F. Findings｜預期產出

預計交付：

- 可展示的領域 AI Agent 原型與操作介面。
- GitHub 程式碼、README、DESIGN.md、法規知識庫及測試資料／腳本。
- 庫存、雙軌殘值、報價及合規分析報告。
- 可追溯的 Word／Excel 文件。
- 比較基線及消融實驗結果，揭示系統效益與限制。

**切勿把預期成果寫成實測成效**。之前討論提出「成功率 ≥90%、作業時間降低 30%、法規引用正確率 ≥95%、高風險測試錯誤放行 0 件」等作為**可討論的目標值**，不是既有結果，需依測試集難度、樣本數與可信區間調整。

## 3. Three Core Innovations｜三大核心創新

### I. Domain-Aware Hybrid AI Agent｜領域感知混合式 Agent

將 LLM 規劃與財產資料、法規 RAG、確定性計算工具分工；不把最終數值與合規裁定交給自由生成。**驗證：**任務成功率、pass^k、規則錯誤率。

### II. Inventory-First Lifecycle Decision Intelligence｜庫存優先的全生命週期決策

把「先查庫存與共享可能 → 比較設備價值 → 決定是否新購」融入單一流程；引入帳面價值和市場價值的雙軌估值。**驗證：**不必要請購辨識能力、殘值預測誤差、決策品質。

### III. Compliance-by-Design & Human-in-the-loop｜內建合規與人類審核

將適用的台灣採購／財產管理規則轉成可檢查條件，附法規出處、版本及稽核紀錄；將高風險決策交回授權人員。**驗證：**違規率、錯誤放行率、引用正確率與人工負擔。

**創新性邊界**：這些技術元素並非各自全新；主張應放在針對特定領域的整合設計以及可重現的實驗驗證。

## 4. Deep Research｜領域專家的思考框架

**五個核心思維模式**：

1. 帳實相符（Record = Reality）
2. 全生命週期（Lifecycle）
3. 先共享、再採購（Utilize before Acquire）
4. 合規與分級授權（Compliance by Threshold）
5. 帳面價值不等於經濟價值（Book vs. Economic Value）

**三項爭論**：

1. 法定／會計折舊年限 vs. 實際經濟壽命
2. 集中共享 vs. 實驗室自有管理
3. 全自動採購 vs. Human-in-the-loop

**十個深度理解問題**：

1. 為什麼財產認定可能同時涉及金額與使用年限？
2. 超過折舊年限但正常使用的設備如何處理？
3. 採購金額改變時，哪些作業流程需要切換？
4. 借用登記嚴格度如何平衡效率與稽核？
5. 如何偵測不合理的低報價？
6. 如何提出可驗證的採購必要性證明？
7. 直線折舊、log-linear 與 Gradient Boosting 應如何比較？
8. 哪些判斷應由規則引擎做，哪些留給 LLM？
9. 成員離校時如何處理財產移交與責任？
10. 如何辨識疑似拆單規避採購程序的行為？

## 5. 規劃的儲存庫目錄

```text
2026-11-11-DIC3-ProjectRefactor/
├── myplan/
│   └── LabSteward_Conversation.md
├── agent/        # LLM Planner 與 Agent 流程
├── tools/        # 資料庫、比價、估值、規則等工具
├── knowledge/    # 法規及領域知識
├── eval/         # 測試案例與評估腳本
├── app/          # Streamlit 介面
├── DESIGN.md     # 架構、資料流與設計決策
└── README.md
```

## 6. 十週工作規劃（提案草案）

| 週次 | 工作 |
| --- | --- |
| W1 | 需求、文獻、限制與風險定義 |
| W2 | 資料蒐集、去識別化與庫存資料庫 |
| W3 | 法規知識庫、檢索及來源追蹤 |
| W4 | 財產查詢與報價工具 |
| W5 | LLM Planner 與 Agent 工作流程 |
| W6 | 雙軌殘值模型 |
| W7 | Guardrail、人工審核及稽核紀錄 |
| W8 | 系統整合及 Streamlit 介面 |
| W9 | Baseline、消融、錯誤與可靠性測試 |
| W10 | 結果整理、GitHub、DESIGN.md 與報告 |

## 7. 文獻與查證清單

以下是**本次對話提及的研究線索**；提交前請核對原始來源、頁碼、年份、樣本與統計定義：

- 監察院：中研院貴重儀器採購／使用管理相關調查（歷史案例）。
- McKinsey（2026）：*Agentic AI in Strategic Sourcing: Hype or Hero?* — https://www.mckinsey.com/capabilities/operations/our-insights/agentic-ai-in-strategic-sourcing-hype-or-hero
- τ-bench：LLM Agent 工具使用、政策遵循及重複執行可靠性研究。
- ICML 2026：*Towards a Science of AI Agent Reliability* — https://proceedings.mlr.press/v306/rabanser26a.html
- 台灣行政院公共工程委員會：採購金額門檻及法規公告 — https://www.pcc.gov.tw/
- 韓國 ZEUS：研究設施設備管理／共享相關年度調查。
- 高等教育政府採購 AI 風險研究（2026）；需核對期刊與 DOI 後正式引用。

## 8. 對話確立的撰寫原則與待辦

- 資訊圖表偏好：**A4 橫式、繁體中文、高密度、NUMBERING 編號、深藍＋青綠系統化版面**，可加入圖示、架構圖、表格與流程圖。
- 專業摘要採 **A–F 六段故事線融入四段文字**、約 500 中文字：動機與挑戰 → 解法與方法 → 評估 → 預期產出。
- 不要誇大「世界首創」、「完全零錯誤」或「已降低成本」等未驗證主張。
- 對引用數字要分清楚：**外部研究事實、提案設計、假設目標及實測結果**。
- 優先開發最小可行版本（MVP）：需求解析 → 庫存查詢 → 規則檢查 → 人工審核 → 產出報告，後續再整合殘值與比價。
- 實驗前先定義標準答案、成功條件、法規版本、失敗處理及資料來源。

---

> 此檔案為這次 ChatGPT 對話的結構化整理，方便放進 `myplan/` 給後續開發者或 Coding Agent 讀取；不包含所有對話的逐字原文，也不表示已連線或提交到 GitHub。
