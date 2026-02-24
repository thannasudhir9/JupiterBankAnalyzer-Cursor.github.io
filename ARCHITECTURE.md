# Architecture & System Design

## Jupiter Bank Statement Analyzer

**Last Updated:** February 25, 2026 (IST)

---

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         BROWSER                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    index.html (SPA)                          │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │ │
│  │  │   Upload    │  │  Password   │  │   Results Section    │  │ │
│  │  │     UI      │  │     UI      │  │  • Personal Info     │  │ │
│  │  └──────┬──────┘  └──────┬──────┘  │  • Transactions      │  │ │
│  │         │                │         │  • Copy Button       │  │ │
│  │         └────────────────┼─────────┴──────────────────────┘  │ │
│  │                          │                                    │ │
│  │  ┌──────────────────────▼──────────────────────────────────┐ │ │
│  │  │                   Application Logic                       │ │ │
│  │  │  handleFile → unlockAndExtract → extractTables → render   │ │ │
│  │  └─────────────────────────┬────────────────────────────────┘ │ │
│  │                            │                                  │ │
│  │  ┌─────────────────────────▼────────────────────────────────┐ │ │
│  │  │                    PDF.js / SheetJS (CDN)                 │ │ │
│  │  │  getDocument, getPage, getTextContent                     │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                               │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │                   Debug Panel                             │ │ │
│  │  │  debugLog() → steps with timestamp & status                │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

     NO BACKEND — All processing happens in the browser
```

---

## 2. Data Flow

```
User selects PDF or Excel file
        │
        ▼
┌───────────────────┐
│   handleFile()     │  • Store file ref; detect type (PDF vs Excel)
│   (async)          │  • PDF: probe load without password
└────────┬──────────┘  • Excel: show View immediately (no password)
         │
         ├── PDF Success ──► pdfIsSecured=false; show View; cache pdf
         ├── PDF PASSWORD_REQUIRED ──► show password input + Unlock & View
         └── Excel ──► currentFileType='excel'; show View; hide Download Unlocked PDF
                  │
                  │  User clicks View / Unlock & View
                  ▼
┌───────────────────┐
│ unlockAndExtract() │  • PDF: pdfjsLib or decrypt; extractTables(pdf)
└────────┬──────────┘  • Excel: extractFromExcel(file) — lines 1-13 personal, 16+ transactions
         │
         ├─► PDF: pdfjsLib.getDocument({ data, password })
         ├─► Excel: XLSX.read(arrayBuffer); slice by EXCEL_PERSONAL_INFO_ROWS, EXCEL_HEADER_ROW
         │
         ▼
┌───────────────────┐     PDF
│  extractTables()   │ ◄────────
│  extractFromExcel()│ ◄──── Excel (lines 1-13, 16+)
└────────┬──────────┘  Split: personalInfo, transactions
         │
         ▼
┌───────────────────┐
│  renderPersonal   │  → Table 1 (if any)
│  InfoTable()       │
└────────────────────┘

┌───────────────────┐
│   renderTable()   │  → Table 2
└────────┬──────────┘
         │
         ▼
User clicks "Copy Transactions as Excel"
         │
         ▼
┌───────────────────┐
│  Build TSV string  │  • Tab-separated
│  clipboard.write  │  • Transactions only
└────────────────────┘
```

---

## 3. Component Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                         UI Components                            │
├────────────────────────────────────────────────────────────────┤
│  UploadZone          │  Drop target + file input                   │
│  PasswordSection     │  Password input + Unlock button              │
│  PersonalInfoSection │  Table 1; eye icon to show/hide details   │
│  TransactionsSection │  Table 2 container + loading state          │
│  SummarySection      │  Financial charts, metrics, categories      │
│  CategorizeWithAIBtn │  Button to run AI categorization            │
│  AiAnalysisSection   │  Inline AI analysis (scrollable)            │
│  ChatWithAISection   │  Chat UI; suggestions; message history     │
│  DebugPanel          │  Processing steps; Clear/Copy               │
│  AiLogsPanel         │  AI API request/response logs; Clear/Copy   │
│  Toast               │  "Copied!" notification                     │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│                       State Variables                             │
├────────────────────────────────────────────────────────────────┤
│  currentPdfFile      │  File object or null                        │
│  currentPdfDoc       │  Cached PDF doc (when unlocked)             │
│  currentFileType     │  'pdf' or 'excel'                           │
│  pdfIsSecured        │  true = password UI; false = View only     │
│  personalInfoData    │  Array of rows for Table 1                  │
│  transactionData     │  Array of rows for Table 2                  │
│  extractedData       │  Copy of transactionData (for clipboard)   │
│  analysisData        │  Categorized transactions, summaries        │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│                       Core Functions                              │
├────────────────────────────────────────────────────────────────┤
│  handleFile()        │  Async: probe PDF or accept Excel; show View/Unlock UI │
│  unlockAndExtract()  │  Async: decrypt/extract (PDF or Excel) + render       │
│  extractTables()     │  Parse PDF → { personalInfo, transactions }          │
│  extractFromExcel()  │  Parse Excel (lines 1-13 personal, 16+ transactions) │
│  groupIntoRows()     │  Cluster text items by Y position           │
│  callGeminiApi()     │  Send prompt to Gemini (model from selector); return text │
│  runAIAnalysis()     │  Full analysis; update UI and logs         │
│  runCategorizeWithAI()│  AI categorization; update parsed & summary │
│  sendChatMessage()   │  Chat Q&A with Gemini                      │
│  updateAiLogs()      │  Update/replace AI API Logs                 │
│  appendAiLogs()      │  Append chat request to AI logs             │
│  renderTable()       │  Build HTML table from 2D array              │
│  debugLog()          │  Append to debug panel                      │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. File Structure

```
JupiterBankStatementAnalyzer/
│
├── index.html                 # Single file app
│   ├── <head>
│   │   ├── <style>             # All CSS (~300 lines)
│   │   └── metadata
│   │
│   └── <body>
│       ├── .container
│       │   ├── header
│       │   ├── .upload-card
│       │   │   ├── .upload-zone
│       │   │   └── .password-section
│       │   └── .results-section
│       │       ├── .personal-info-table
│       │       ├── .transactions-section
│       │       ├── .summary-section
│       │       ├── .ai-analysis-section
│       │       └── .chat-with-ai-section
│       │
│       ├── .logs-panels-row (Debug + AI API Logs)
│       ├── .toast
│       └── <script type="module">
│           ├── PDF.js import & worker setup
│           ├── DOM refs
│           ├── Event handlers
│           ├── handleFile, unlockAndExtract
│           ├── extractTables, groupIntoRows
│           ├── renderPersonalInfoTable, renderTable
│           ├── handleFile, unlockAndExtract, extractTables
│           ├── callGeminiApi, runAIAnalysis, sendChatMessage
│           ├── debugLog, debugClear, updateAiLogs, appendAiLogs
│           └── copy button handler
│
├── .gitignore
├── README.md
└── docs/
    ├── PRD.md
    ├── TECHNICAL.md
    ├── ARCHITECTURE.md
    └── CHANGELOG_PROMPTS.md
```

---

## 5. Processing Pipeline

```
PDF File (binary)
       │
       ▼
┌──────────────────┐
│ ArrayBuffer       │  file.arrayBuffer()
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Uint8Array        │  Wrapped for PDF.js
└────────┬─────────┘
         │
         ▼
┌──────────────────┐     password
│ PDF.js Decrypt    │ ◄──────────────
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ PDFDocument       │  pdf.numPages, pdf.getPage()
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ TextContent       │  page.getTextContent()
│ items[]           │  { str, transform, width, height }
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Rows (grouped)    │  groupIntoRows() by Y
│ cells per row     │  sort by X
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Heuristic Split   │  Transaction header / row detection
└────────┬─────────┘
         │
         ├──────────────────┬──────────────────┐
         ▼                  ▼                  ▼
   personalInfo        transactions      Debug log
   (Table 1)           (Table 2)        (steps)
         │                  │
         ▼                  ▼
   renderPersonalInfo   renderTable
   (HTML table)         (HTML table)
                             │
                             ▼
                       Copy as TSV
                       (clipboard)
```

---

## 6. Security Considerations

| Concern | Mitigation |
|---------|------------|
| PDF content exposure | Processed only in browser; never sent to server |
| AI (Gemini) | Transaction data sent to Google; API key local only |
| Password handling | Used only for PDF.js decrypt; not stored or transmitted |
| API key / model | User-entered key; model selector (default gemini-2.5-flash); both in localStorage |
| XSS | `escapeHtml()` used for user-generated content in DOM |
| Sensitive data in copy | Only transactions copied; personal info excluded |

---

## 7. Extensibility

- **New banks/formats**: Extend `PERSONAL_INFO_LABELS` and `TRANSACTION_HEADER_LABELS` regexes
- **Additional tables**: Extend split heuristics in `extractTables()`
- **Export formats**: Add CSV/Excel binary export alongside TSV
- **Backend**: Could add optional server endpoint for heavy PDFs or OCR
