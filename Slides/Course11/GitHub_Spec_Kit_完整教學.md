# GitHub Spec Kit 完整教學：從 Vibe Coding 到 Spec-Driven Development

> 本教學根據 Spec Kit 維護者 **Dan Delimarsky** 的[官方教學影片](https://www.youtube.com/watch?v=a9eR1xsfvHg)以及 [GitHub 官方 Repo](https://github.com/github/spec-kit) 整理而成。

---

## 目錄

1. [什麼是 Spec-Driven Development？](#1-什麼是-spec-driven-development)
2. [Spec Kit 簡介](#2-spec-kit-簡介)
3. [安裝方式](#3-安裝方式)
4. [核心工作流程總覽](#4-核心工作流程總覽)
5. [Step 1：初始化專案（specify init）](#5-step-1初始化專案specify-init)
6. [Step 2：建立 Constitution（憲章）](#6-step-2建立-constitution憲章)
7. [Step 3：定義 Specification（規格）](#7-step-3定義-specification規格)
8. [Step 4：制定 Plan（技術計畫）](#8-step-4制定-plan技術計畫)
9. [Step 5：拆解 Tasks（任務清單）](#9-step-5拆解-tasks任務清單)
10. [Step 6：Implementation（實作）](#10-step-6implementation實作)
11. [成果展示](#11-成果展示)
12. [支援的 AI Agent 一覽](#12-支援的-ai-agent-一覽)
13. [進階功能：Extensions 與 Presets](#13-進階功能extensions-與-presets)
14. [最佳實踐與注意事項](#14-最佳實踐與注意事項)
15. [常用 CLI 參數參考](#15-常用-cli-參數參考)
16. [總結](#16-總結)

---

## 1. 什麼是 Spec-Driven Development？

Spec-Driven Development（SDD，規格驅動開發）是一種將「規格」（Specification）從傳統的文件輔助角色，提升為**可執行的核心產出物**的開發方法論。

傳統開發流程中，你可能先寫一份 PRD（產品需求文件），然後交給工程師去實作——但 PRD 和最終程式碼之間常常存在巨大鴻溝。SDD 的核心理念是：

> **規格不是為程式碼服務的——程式碼才是為規格服務的。**

這個「權力反轉」意味著：當需求變更時，你只需要更新規格，就能系統性地重新生成程式碼，而不是在既有程式碼上打補丁。

與「Vibe Coding」的對比：

| | Vibe Coding | Spec-Driven Development |
|---|---|---|
| **方法** | 給 AI 模糊指令，期望得到正確結果 | 撰寫結構化規格，AI 依規格實作 |
| **可重現性** | 低，每次結果可能不同 | 高，同一規格可生成多個實作變體 |
| **可維護性** | 難以追蹤設計決策 | 規格即文件，易於團隊共享 |
| **迭代方式** | 不斷修補程式碼 | 更新規格後重新生成 |

---

## 2. Spec Kit 簡介

**Spec Kit** 是 GitHub 開源的工具包，幫助開發者快速上手 Spec-Driven Development。它的目標是取代「從零開始的 vibe coding」，提供一套結構化的工作流程。

![Spec Kit GitHub Repo 頁面](screenshots/01_github_stars.jpg)
*▲ Spec Kit 在 GitHub 上的 Repo，發布一週即獲得超過 16,000 顆星。*

Spec Kit 的核心組成：

- **specify CLI**：命令列工具，自動化 scaffolding 整個 SDD 流程
- **Slash Commands**：與 AI Agent 搭配使用的提示指令（如 `/speckit.specify`、`/speckit.plan` 等）
- **Templates**：結構化模板，約束 LLM 產出更高品質的規格文件
- **Helper Scripts**：輔助腳本（Shell / PowerShell），處理 Git 分支、JSON 元資料等確定性操作

---

## 3. 安裝方式

### 方式一：使用 uvx 安裝（推薦）

```bash
# 安裝指定穩定版本
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z

# 或安裝最新 main 分支版本
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

安裝後可直接使用：

```bash
specify init <PROJECT_NAME>
specify init . --ai claude
specify check
```

### 方式二：一次性使用（uvx）

```bash
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init <PROJECT_NAME>
```

如影片中 Dan 所示，他直接從 GitHub Repo 複製 uvx 指令在終端機執行。

### 方式三：手動下載（不需安裝任何工具）

如果你完全不想安裝 CLI，可以直接到 [GitHub Releases](https://github.com/github/spec-kit/releases) 頁面，下載對應你 AI Agent 和作業系統的 zip 檔案，解壓縮後放入專案資料夾即可。

![GitHub Repo 檔案結構](screenshots/02_github_repo.jpg)
*▲ Spec Kit 的 GitHub Repo 結構，包含 templates、scripts 等核心資料夾。*

> **影片重點提示**：Dan 強調「CLI 只是便利層（convenience layer），你完全不需要安裝它——只需下載 Release 中的 zip 放進專案即可。」

---

## 4. 核心工作流程總覽

Spec Kit 的 SDD 流程分為五個階段：

```
Constitution → Specify → Plan → Tasks → Implement
  （憲章）     （規格）   （計畫）  （任務）    （實作）
```

每個階段對應一個 Slash Command：

| 階段 | Slash Command | 說明 |
|---|---|---|
| **Constitution** | `/speckit.constitution` | 建立專案不可妥協的原則 |
| **Specify** | `/speckit.specify` | 定義「做什麼」和「為什麼」（產品視角） |
| **Plan** | `/speckit.plan` | 制定「怎麼做」（技術視角） |
| **Tasks** | `/speckit.tasks` | 拆解為可執行的任務清單 |
| **Implement** | `/speckit.implement` | AI Agent 逐一實作任務 |

另外還有可選指令：

- `/speckit.clarify`：釐清規格中不明確的部分
- `/speckit.analyze`：跨文件一致性分析
- `/speckit.checklist`：生成品質驗證清單

---

## 5. Step 1：初始化專案（specify init）

在影片中，Dan 以建立一個「Podcast 網站」為範例，示範整個流程。

### 執行初始化

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init pod-site
```

![CLI 初始化畫面](screenshots/03_cli_init.jpg)
*▲ 在終端機執行 `specify init pod-site` 開始建立新專案。*

### 選擇 AI Agent

CLI 會提示你選擇要搭配的 AI Agent：

![Agent 選擇畫面](screenshots/04_agent_selection.jpg)
*▲ specify CLI 提供多種 AI Agent 選擇，包括 GitHub Copilot、Claude Code、Cursor、Gemini CLI 等。*

你可以用方向鍵選擇 Agent。在影片中 Dan 選擇了 **GitHub Copilot**。

### 選擇腳本類型

接著選擇 Helper Scripts 的類型：
- **PowerShell**：適用於 Windows 原生環境
- **Shell (sh)**：適用於 Linux / macOS / WSL2

CLI 會根據你的作業系統智慧預設。

### 初始化完成後的專案結構

```
pod-site/
├── .github/
│   └── prompts/           # Slash Commands 提示檔
│       ├── speckit.plan.prompt.md
│       ├── speckit.specify.prompt.md
│       └── speckit.tasks.prompt.md
└── .specify/
    ├── memory/            # 憲章與記憶檔案
    │   └── constitution.md
    ├── scripts/           # Helper Scripts (PowerShell / sh)
    └── templates/         # 規格、計畫、任務的模板
```

![VS Code 中的專案結構](screenshots/05_vscode_project.jpg)
*▲ 在 VS Code 中打開初始化後的專案，可以看到 `.github` 和 `.specify` 資料夾。*

> **注意**：必須在專案根目錄（例如 `pod-site/`）下開啟 VS Code，Slash Commands 才能正確被辨識。如果你在上一層目錄開啟，輸入 `/specify` 時不會看到任何指令。

---

## 6. Step 2：建立 Constitution（憲章）

### 什麼是 Constitution？

Constitution 是專案的「憲法」，定義一組**不可妥協的原則**。不管後續如何迭代，AI Agent 在每個步驟都必須遵守這些原則。

例如：
- 「必須始終有測試」
- 「必須使用 Next.js 14 以上版本」
- 「不得使用伺服器端執行」

### 如何填寫

Constitution 檔案位於 `.specify/memory/constitution.md`，初始時是空白模板。你可以：

1. **手動填寫**：直接編輯 Markdown 檔案
2. **讓 LLM 協助填寫**：在 Agent 聊天中輸入指示

在影片中，Dan 輸入：

```
Fill the constitution with the bare minimum requirements for a static web app based on a template
```

![Constitution 填寫結果](screenshots/06_constitution.jpg)
*▲ LLM 正在根據模板填寫 Constitution 檔案。*

### 填寫結果

![Constitution 完成](screenshots/07_constitution_filled.jpg)
*▲ 完成的 Constitution 包含三條核心原則。*

GPT-5 為這個靜態網站專案產出了三條憲章原則：

1. **Article I - Static-First Delivery**：不使用伺服器端執行，網站以 HTML/CSS/JS 和靜態資源透過 CDN 發布
2. **Article II - Simplicity Over Tooling**：優先使用原生 HTML/CSS/JS，只在確實需要時才引入框架
3. **Article III - Accessibility & SEO Baseline**：使用語意化 HTML、提供 meta 描述、robots.txt 和 sitemap.xml

> **影片重點提示**：Dan 特別移除了 LLM 額外產出的「Performance Budget」和「Quality Gates」等條款，因為這是原型階段不需要的。這展示了**人工介入編輯**的重要性——你不必照單全收 LLM 的產出。

---

## 7. Step 3：定義 Specification（規格）

### 核心概念

Specification 階段的重點是定義**「做什麼」**和**「為什麼」**，完全不涉及技術實作細節。這就像一位產品經理（PM）在撰寫需求文件。

**規格與實作的分離**是 SDD 最大的優勢之一：同一份規格可以用不同的技術棧（Next.js、Hugo、Astro 等）來實作，無需重寫需求。

### 執行 /speckit.specify

在 Agent 聊天中輸入 `/specify`，然後描述你的需求：

```
I am building a modern podcast website to look sleek, something that would stand out.
Should have a landing page with one featured episode.
There should be an episodes page, an about page, and an FAQ page.
Should have 20 episodes and the data is mocked.
You do not need to pull anything from any real feed.
```

![Specify 指令執行](screenshots/08_specify_command.jpg)
*▲ 使用 `/specify` 指令，輸入產品需求。注意此處完全不提技術細節。*

### Specify 的運作機制

1. 讀取 `/speckit.specify` 的提示檔案
2. 執行 Helper Script（建立 Git 分支、設定元資料）
3. 讀取 Spec Template 作為基底
4. 根據你的描述和 Constitution 產出完整規格

### 規格產出內容

![Specification 結果](screenshots/09_spec_result.jpg)
*▲ LLM 產出的 Specification 包含使用者故事、驗收條件、功能需求等。*

產出的規格文件（`spec.md`）包含：

- **Quick Guidelines**：規格撰寫指南（聚焦使用者需求而非技術細節）
- **User Scenarios & Testing**：使用者故事與驗收情境
- **Edge Cases**：邊界情況（如集數排序未指定時的預設行為）
- **Functional Requirements**：功能需求清單
- **Review & Acceptance Checklist**：驗收清單

### 處理「需要釐清」的項目

規格中可能會有 `[NEEDS CLARIFICATION]` 標記，表示 LLM 對某些需求有疑問。在影片中，Dan 直接讓 LLM 自行判斷：

```
Use the best guess you think is reasonable. Update acceptance checklist after.
```

對於**正式專案**，你應該自己回答這些問題，確保需求精確。但在原型階段，讓 LLM 做出合理假設是可接受的。

---

## 8. Step 4：制定 Plan（技術計畫）

### 核心概念

Plan 階段才是定義**「怎麼做」**的時候。這裡你指定技術棧、架構約束、效能目標等技術細節。

### 執行 /speckit.plan

```
Use Next.js with static site configuration. No databases.
Make sure that the site is responsive and ready for mobile.
```

![Plan 指令執行](screenshots/10_plan_command.jpg)
*▲ 使用 `/plan` 指令，指定技術需求。*

### Plan 的運作機制

1. 讀取 Specification（確認產品需求）
2. 讀取 Constitution（確認不可違反的原則）
3. 讀取 Plan Template
4. 產出技術計畫文件

> **關鍵觀察**：影片中可以看到 Agent 確實讀取了 Constitution，確保技術計畫不違反核心原則。

### 計畫產出內容

![Plan 結果](screenshots/11_plan_result.jpg)
*▲ 完成的技術計畫包含語言版本、依賴套件、目標平台等詳細資訊。*

產出的計畫文件包含：

- **Execution Flow**：執行流程圖
- **Technical Context**：語言版本（JavaScript/TypeScript + Node.js）
- **Primary Dependencies**：Next.js + Static Export (SSG)
- **Testing**：Lighthouse 和其他測試工具
- **Target Platform**：Static hosting via CDN（Vercel / GitHub Pages / Azure Static Web Apps）
- **Constitution Checks**：確認各項原則是否通過
- **Architecture Gates**：架構檢查（使用框架、單一資料模型、避免反模式等）
- **Data Contract**：資料模型定義（Episode 的欄位結構）
- **Research Artifacts**：技術研究筆記

### 資料夾結構

每個功能（Feature）有自己的子資料夾：

```
.specify/specs/
└── 001-building-modern-podcast-website/
    ├── spec.md          # 規格
    ├── plan.md          # 技術計畫
    ├── contracts.md     # 資料契約
    ├── research.md      # 研究紀錄
    └── quickstart.md    # 快速入門
```

---

## 9. Step 5：拆解 Tasks（任務清單）

### 執行 /speckit.tasks

```
Break this down.
```

![Tasks 指令執行](screenshots/12_tasks_command.jpg)
*▲ 使用 `/tasks` 指令拆解工作項目。*

### 任務清單產出

![Tasks 結果](screenshots/13_tasks_result.jpg)
*▲ 產出的任務清單將工作拆解為多個階段性步驟。*

產出的 `tasks.md` 將整個實作拆解為多個階段：

1. **Setup**：初始化 Next.js App 骨架
2. **Test-First**：先撰寫測試（TDD），確認測試失敗
3. **Core Implementation**：在測試失敗的前提下實作功能
   - Landing Page
   - Episodes Page
   - About Page
   - FAQ Page
4. **Integration & Refinement**：整合測試（如 Lighthouse）
5. **Polish**：響應式設計、圖片最佳化、文件撰寫、無障礙性優化

每個任務都是獨立的、可審查的工作單元，類似 TDD 的精神。

---

## 10. Step 6：Implementation（實作）

### 執行實作

在影片中，Dan 先切換模型到 **Claude Sonnet 4**（他個人認為程式碼生成品質最佳），然後輸入：

```
/implement
Implement the tasks for this project and update the task list as you go.
```

![Implementation 執行](screenshots/14_implement.jpg)
*▲ 使用 `/implement` 指令，AI Agent 開始逐一實作任務。*

### 實作過程

AI Agent 會：
1. 依照 `tasks.md` 的順序逐一執行
2. 完成每個任務後更新任務狀態
3. 遇到問題時標記（如 Lighthouse 測試因未安裝 Chrome 而失敗）

### 建置與預覽

實作完成後：

```bash
npm run build    # 建置靜態網站
npm run dev      # 啟動開發伺服器
```

---

## 11. 成果展示

![最終網站 - Landing Page](screenshots/15_final_website.jpg)
*▲ 最終產出的 Podcast 網站首頁，包含導覽列、主視覺和精選集數。*

![最終網站 - Episodes Page](screenshots/16_final_episodes.jpg)
*▲ Episodes 頁面，每集都有標題、描述、標籤和「Listen Now」按鈕。*

網站包含了所有規格中定義的頁面：
- **Home**：首頁有主視覺「Master the Art of Podcasting」和精選集數
- **Episodes**：20 集模擬資料，含標籤分類
- **About**：關於頁面
- **FAQ**：常見問題

> **Dan 的評價**：「看起來不錯！不是完美的，但作為原型已經很好了。重要的是——現在我有了完整的規格，我可以輕鬆地修改需求並重新生成。」

---

## 12. 支援的 AI Agent 一覽

Spec Kit 支援超過 20 種 AI 編碼 Agent：

| Agent | 說明 |
|---|---|
| **GitHub Copilot** | VS Code Agent Mode 中使用 Slash Commands |
| **Claude Code** | 終端機使用，擅長研究與程式碼生成 |
| **Cursor** | Cursor Agent，新增支援 |
| **Gemini CLI** | Google 的命令列 AI Agent |
| **Windsurf** | Codeium 的 AI IDE |
| **Qwen Code** | 阿里巴巴的 AI 編碼工具 |
| **Kiro CLI** | AWS 的 AI 開發工具 |
| **JetBrains Junie** | JetBrains IDE 中的 AI Agent |
| **Generic** | 通用選項，支援未列出的 Agent |

> **影片中 Dan 的建議**：他個人偏好用 **GPT-5** 進行規格相關的步驟（Specify、Plan、Tasks），因為產出較為穩定；而用 **Claude Sonnet 4** 進行實作（Implement），因為程式碼品質較佳。但他強調「唯一知道哪個模型最好的方式就是實驗」。

---

## 13. 進階功能：Extensions 與 Presets

Spec Kit 提供可擴展的架構：

### 優先層級（由高到低）

1. **Project-Local Overrides**：`.specify/templates/overrides/` — 專案級覆寫
2. **Presets**：`.specify/presets/<preset-id>/templates/` — 預設組合
3. **Extensions**：`.specify/extensions/<ext-id>/templates/` — 擴充功能
4. **Spec Kit Core**：`.specify/templates/` — 核心模板

### 使用場景

- **Extensions** 新增功能（例如新增安全審查步驟）
- **Presets** 客製化核心行為（例如「海盜風格」的提示語）
- **Project-Local Overrides** 覆寫上述兩者，無需修改原始檔案

---

## 14. 最佳實踐與注意事項

根據影片和官方文件，整理出以下最佳實踐：

### 規格撰寫

- **Specify 階段不要提及技術細節**：語言、框架、API 等留到 Plan 階段
- **善用 Constitution**：把跨功能的共通原則（如「所有內容必須支援繁體中文」）寫在 Constitution
- **手動編輯是必要的**：LLM 產出的規格只是起點，你應該以人工審查並修改

### 模型選擇

- 規格 / 計畫階段：推薦使用推理能力強的模型（如 GPT-5、Claude Opus）
- 實作階段：推薦使用程式碼生成品質高的模型（如 Claude Sonnet 4）
- **多做實驗**：不同模型在不同場景下表現差異很大

### Git 工作流程

- Spec Kit 自動建立 Git 分支來隔離每個功能的工作
- 實驗完成後再合併到 main branch
- 如果結果不滿意，可以刪除分支重來，規格文件仍然保留

### 迭代策略

- **同一規格、不同實作**：可以用不同模型或技術棧重新實作
- **增量開發**：用 `/speckit.specify` 新增功能規格，每個功能有獨立子資料夾
- **Brownfield 專案**：已有的專案也可以用 Spec Kit 來管理新功能

### 企業環境注意事項

- 在受控環境中，手動指定精確需求比讓 LLM 猜測更重要
- Constitution 可以編碼組織級的規範（如安全政策、合規要求）
- 可搭配 MCP 工具（如 Figma MCP）讓 AI 參考實際設計稿

---

## 15. 常用 CLI 參數參考

### `specify init` 完整參數

| 參數 | 類型 | 說明 |
|---|---|---|
| `<project-name>` | 位置參數 | 新專案目錄名稱（或使用 `.` 代表當前目錄） |
| `--ai` | 選項 | AI Agent 類型（`claude`、`copilot`、`cursor`、`gemini` 等） |
| `--script` | 選項 | 腳本類型：`sh` 或 `ps`（PowerShell） |
| `--here` | 旗標 | 在當前目錄初始化 |
| `--force` | 旗標 | 強制合併，不需確認 |
| `--no-git` | 旗標 | 跳過 Git 初始化 |
| `--ai-skills` | 旗標 | 以 Agent Skills 形式安裝 |
| `--branch-numbering` | 選項 | `sequential`（預設）或 `timestamp` |
| `--debug` | 旗標 | 啟用除錯輸出 |

### 環境變數

| 變數 | 說明 |
|---|---|
| `SPECIFY_FEATURE` | 覆寫 feature detection（用於非 Git 環境） |

---

## 16. 總結

Spec Kit 提供的 Spec-Driven Development 流程，將 AI 輔助開發從「隨意的 vibe coding」提升到「結構化的規格驅動」。關鍵價值在於：

1. **意圖即真相（Intent as Source of Truth）**：規格取代程式碼成為專案的核心
2. **可重現性**：同一規格可用不同技術棧、不同模型重新實作
3. **團隊協作**：規格文件是 Markdown 格式，任何人都能閱讀和編輯
4. **增量迭代**：每個功能有獨立的規格 → 計畫 → 任務 → 實作流程
5. **品質把關**：Constitution 確保核心原則不被違反，Checklist 確保規格完整性

如 Dan 在影片結尾所說：Spec Kit 仍是實驗性專案，團隊非常歡迎社群回饋。前往 [github.com/github/spec-kit](https://github.com/github/spec-kit) 下載使用，並在 Issues 中分享你的經驗！

---

*本教學最後更新：2026-03-25*
*資料來源：[YouTube 影片](https://www.youtube.com/watch?v=a9eR1xsfvHg) ｜ [GitHub Repo](https://github.com/github/spec-kit) ｜ [GitHub Blog](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)*
