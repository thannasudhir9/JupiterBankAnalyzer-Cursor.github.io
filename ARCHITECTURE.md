# Architecture & System Design

## Jupiter Bank Statement Analyzer

**Last Updated:** February 19, 2025

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
│  │  │                    PDF.js (CDN)                           │ │ │
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
User selects PDF file
        │
        ▼
┌───────────────────┐
│   handleFile()     │  • Store file ref
│   (async)          │  • Probe: try load without password
└────────┬──────────┘
         │
         ├── Success ──► pdfIsSecured=false; hide password; show View; cache pdf
         │
         └── PASSWORD_REQUIRED ──► pdfIsSecured=true; show password input + Unlock & View
                  │
                  │  User enters password
                  ▼
┌───────────────────┐
│ unlockAndExtract() │  • Unlocked: use cached pdf (skip load)
└────────┬──────────┘  • Secured: load with password
         │
         ├─► Read file.arrayBuffer()
         ├─► pdfjsLib.getDocument({ data, password })
         ├─► pdf (decrypted)
         │
         ▼
┌───────────────────┐
│  extractTables()   │
│  • For each page:  │
│    - getTextContent
│    - groupIntoRows
│    - collect cells
│  • Split by heuristics:
│    - Table 1: personalInfo
│    - Table 2: transactions
└────────┬──────────┘
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
│  PersonalInfoSection │  Table 1 container (sensitive label)       │
│  TransactionsSection │  Table 2 container + loading state          │
│  SummarySection      │  Financial charts, metrics, categories      │
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
│  currentPdfDoc       │  Cached PDF doc (when unlocked)            │
│  pdfIsSecured        │  true = password UI; false = View only     │
│  personalInfoData    │  Array of rows for Table 1                  │
│  transactionData     │  Array of rows for Table 2                  │
│  extractedData       │  Copy of transactionData (for clipboard)   │
│  analysisData        │  Categorized transactions, summaries        │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│                       Core Functions                              │
├────────────────────────────────────────────────────────────────┤
│  handleFile()        │  Async: probe PDF; show View or Unlock UI   │
│  unlockAndExtract()  │  Async: decrypt + extract + render          │
│  extractTables()     │  Parse PDF → { personalInfo, transactions } │
│  groupIntoRows()     │  Cluster text items by Y position           │
│  callGeminiApi()     │  Send prompt to Gemini; return text         │
│  runAIAnalysis()     │  Full analysis; update UI and logs         │
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
├── api-config.js              # Gemini API key (gitignored)
├── api-config.example.js      # Template
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
| API key | api-config.js in .gitignore; optional localStorage |
| XSS | `escapeHtml()` used for user-generated content in DOM |
| Sensitive data in copy | Only transactions copied; personal info excluded |

---

## 7. Extensibility

- **New banks/formats**: Extend `PERSONAL_INFO_LABELS` and `TRANSACTION_HEADER_LABELS` regexes
- **Additional tables**: Extend split heuristics in `extractTables()`
- **Export formats**: Add CSV/Excel binary export alongside TSV
- **Backend**: Could add optional server endpoint for heavy PDFs or OCR
