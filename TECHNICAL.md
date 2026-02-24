# Technical Documentation

## Jupiter Bank Statement Analyzer

**Last Updated:** February 25, 2026 (IST)

---

## 1. Tech Stack

| Layer | Technology |
|-------|------------|
| **Markup** | HTML5 |
| **UI Framework** | Bootstrap 5.3.2 |
| **Icons** | Bootstrap Icons 1.11.1 |
| **Styling** | CSS3 (custom properties, flexbox, grid, theme vars) |
| **Script** | JavaScript ES6+ (modules) |
| **PDF** | PDF.js (Mozilla) v4.0.379 |
| **Excel** | SheetJS (xlsx) v0.20.3 |
| **Charts** | Chart.js 4.4.1 |
| **Font** | Plus Jakarta Sans (Google Fonts) |
| **Hosting** | Static files; served via any HTTP server |

---

## 2. Dependencies

| Dependency | Version | Purpose | Source |
|------------|---------|---------|--------|
| pdfjs-dist | 4.0.379 | PDF parsing, decryption, text extraction | jsDelivr CDN |
| xlsx (SheetJS) | 0.20.3 | Excel .xls/.xlsx parsing | cdn.sheetjs.com |
| @google/genai | 1.x | Gemini AI (Analyse with AI, Chat with AI) | esm.sh |
| Bootstrap | 5.3.2 | UI components, layout, utilities | jsDelivr CDN |
| Bootstrap Icons | 1.11.1 | Icon set | jsDelivr CDN |
| Chart.js | 4.4.1 | Financial charts (bar, pie, line) | jsDelivr CDN |

### CDN URLs

```html
<!-- PDF.js (ESM) -->
https://cdn.jsdelivr.net/npm/pdfjs-dist@4.0.379/build/pdf.min.mjs
https://cdn.jsdelivr.net/npm/pdfjs-dist@4.0.379/build/pdf.worker.min.mjs

<!-- SheetJS (Excel) -->
https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js

<!-- Bootstrap, Icons, Chart.js -->
Bootstrap 5.3.2, Bootstrap Icons 1.11.1, Chart.js 4.4.1
```

---

## 3. PDF.js API Usage

### Initialization

```javascript
import * as pdfjsLib from 'https://cdn.jsdelivr.net/npm/pdfjs-dist@4.0.379/build/pdf.min.mjs';

pdfjsLib.GlobalWorkerOptions.workerSrc = 
  'https://cdn.jsdelivr.net/npm/pdfjs-dist@4.0.379/build/pdf.worker.min.mjs';
```

### Loading Password-Protected PDF

```javascript
const arrayBuffer = await file.arrayBuffer();
const loadingTask = pdfjsLib.getDocument({
  data: new Uint8Array(arrayBuffer),
  password: userPassword || undefined,
  verbosity: 0
});
const pdf = await loadingTask.promise;
```

### Text Extraction

```javascript
const page = await pdf.getPage(pageNumber);
const textContent = await page.getTextContent();

// Each item has: str, transform[4]=x, transform[5]=y, width, height
const items = textContent.items.map(item => ({
  str: item.str,
  x: item.transform[4],
  y: item.transform[5],
  width: item.width,
  height: item.height
}));
```

---

## 4. Table Extraction Logic

### Overview

1. Extract all text items with (x, y) coordinates from every page
2. Group items into rows by Y position (within a threshold)
3. Sort items in each row by X to form cells
4. Classify rows as either personal info or transaction using heuristics
5. Split into Table 1 (personal) and Table 2 (transactions)

### Row Grouping

- **Threshold**: `max(6, avgHeight * 0.5)` — items within this vertical distance belong to the same row
- **Sort**: Rows sorted by Y descending (top to bottom); cells in each row by X ascending

### Table Split Heuristics

**Transaction Header Detection** (`looksLikeTransactionHeader`)

- Row text matches: `date|particulars|narration|description|debit|credit|balance|transaction|ref|value|amount|withdrawal|deposit|dr|cr|chq|cheque`
- All cell lengths < 30 characters

**Transaction Row Detection** (`looksLikeTransactionRow`)

- Contains date pattern: `DD/MM/YYYY`, `DD-MMM-YY`, `DD-Feb-YYYY`, `MMM DD, YYYY`, etc.
- OR contains amount pattern: numeric with optional decimals; `INR 1234`, `Rs 1000`
- OR has numeric cells and ≥2 columns

**Personal Info Detection** (`looksLikePersonalInfoRow`)

- Row text matches: `account|name|holder|customer|address|branch|ifsc|pan|aadhaar|email|phone|statement period|from|to`
- OR row has ≤3 cells and cell length 2–60 characters

**Header/Footer Filter** (`isHeaderOrFooterRow`)

- Stricter patterns to avoid filtering valid transaction rows (e.g. "NEFT to Federal Bank")
- Matches: corporate office, phone numbers, URLs, toll-free numbers, etc.

### Split Algorithm

1. Find first row matching transaction header → that row and below = Table 2; above = Table 1
2. If no header found, find first row matching transaction row → split there
3. Else, find last row matching personal info → split after it
4. **Fallback**: If no transactions found but rows exist, use all rows as transactions (auto-detect header)

---

## 5. Data Structures

### Extracted Tables

```javascript
// Table 1: Account holder info
personalInfoData = [
  ['Account Number', 'XXXX1234'],
  ['Customer Name', 'John Doe'],
  // ...
];

// Table 2: Transactions (header + rows)
transactionData = [
  ['Date', 'Narration', 'Debit', 'Credit', 'Balance'],
  ['01-Dec-2025', 'NEFT IN', '', '5000', '15000'],
  // ...
];
```

### Clipboard Format (TSV)

```
Date	Narration	Debit	Credit	Balance
01-Dec-2025	NEFT IN		5000	15000
02-Dec-2025	UPI/XXX	100		14900
```

- Tab (`\t`) as column separator
- Newline (`\n`) as row separator
- Cells with tabs/newlines/quotes wrapped in double quotes; inner quotes escaped as `""`

---

## 6. Key Functions

| Function | Purpose |
|----------|---------|
| `handleFile(file)` | Async: detect PDF/Excel; probe PDF or show View for Excel |
| `unlockAndExtract()` | Read file, decrypt (if needed), extract, render; reuses cached doc for unlocked PDFs |
| `updateUnlockButtonLabel()` | Enable/disable Unlock button when password entered (secured PDFs only) |
| `extractTables(pdf)` | Extract and split into personalInfo + transactions; includes fallback logic |
| `groupIntoRows(items)` | Group text items by Y into rows |
| `renderPersonalInfoTable(data)` | Render Table 1 |
| `renderTable(data)` | Render Table 2 |
| `analyzeTransactions(data)` | Parse DR/Cr columns; categorize transactions; compute summary |
| `detectColumnIndices(headers)` | Detect Withdrawals, Deposits, Dr/Cr, Particulars, Date |
| `parseAmount(str)` | Parse INR amounts; preserve decimal (rupees.paisa) |
| `extractFromExcel(file)` | Parse Excel; lines 1-13 personal, line 16 headers, 17+ transactions |
| `categorizeNarration(narration, txType)` | Regex-based category from narration; respects DR/Cr-derived type |
| `runCategorizeWithAI(apiKey)` | Send transactions to Gemini; assign category per row; recompute summary |
| `recomputeAggregatesFromParsed(parsed)` | Rebuild byCategory, daily/weekly/monthly/yearly from parsed |
| `updateCharts()` | Render charts; period selector (Daily/Weekly/Monthly/Yearly) |
| `callGeminiApi(apiKey, prompt)` | Send prompt to Gemini 2.5 Flash; return response text |
| `runAIAnalysis(apiKey)` | Full analysis flow; update AI Analysis section and logs |
| `sendChatMessage(question)` | Chat Q&A; send question + transaction data to Gemini |
| `appendChatMessage(role, text)` | Add user/assistant message to chat UI |
| `updateAiLogs(request, response, error)` | Replace AI API Logs (Analyse with AI) |
| `appendAiLogs(label, request, response, error)` | Append REQUEST section to AI logs |
| `appendAiLogResponse(label, response, error)` | Append RESPONSE/ERROR to AI logs |
| `renderMarkdown(text)` | Convert markdown to HTML for display |
| `debugLog(msg, status, detail)` | Append step to debug panel |
| `debugClear()` | Clear debug panel |
| `escapeHtml(text)` | Sanitize for innerHTML |

### Gemini API Configuration

- **Model**: User-selectable in AI Setup dropdown; default `gemini-2.5-flash`. Options: `gemini-2.5-flash`, `gemini-3-flash-preview`, `gemini-2.5-pro`, `gemini-2.0-flash`, `gemini-1.5-flash`, `gemini-1.5-pro`
- **Storage**: Model choice stored in `localStorage` under `gemini-model`
- **API Key**: User-entered in modal; stored in localStorage only

### PDF Probe Flow (`handleFile`)

1. User selects file → show "Checking PDF..." on button
2. Try `pdfjsLib.getDocument({ data })` (no password)
3. **Success** → `pdfIsSecured = false`; hide password input; show View; cache `currentPdfDoc`
4. **PASSWORD_REQUIRED** → `pdfIsSecured = true`; show password input; show Unlock & View; button disabled until password entered

---

## 7. CSS Architecture

### Design Tokens

```css
:root {
  --bg-primary: #0f1419;
  --bg-secondary: #1a2332;
  --bg-card: #1e2a3a;
  --accent: #3b82f6;
  --accent-hover: #2563eb;
  --text-primary: #f1f5f9;
  --text-secondary: #94a3b8;
  --border: #334155;
  --success: #22c55e;
  --error: #ef4444;
}
```

### Layout

- Single column, max-width ~1000px
- Flexbox for headers and actions
- Sticky table headers for long lists
- Debug panel: collapsible, max-height with scroll

---

## 8. Browser Compatibility

| Browser | Min Version | Notes |
|---------|-------------|-------|
| Chrome | 80+ | Full support |
| Firefox | 80+ | Full support |
| Safari | 14+ | Full support |
| Edge | 80+ | Full support |

**Requirements**

- ES modules support
- `async`/`await`
- `navigator.clipboard.writeText`
- `ArrayBuffer`, `Uint8Array`

---

## 9. Build & Deployment

No build step. Static files only. Serve the project directory over HTTP (ES modules require it).

### How to Run Locally

**Python:**
```bash
cd JupiterBankStatementAnalyzer
python3 -m http.server 8080
# Open http://localhost:8080
```

**Node.js:**
```bash
npx serve .
# Open the URL shown
```

**VS Code Live Server:** Right-click `index.html` → Open with Live Server

---

## 10. AI Features (Gemini)

### Transaction Analysis (DR/Cr & Categorization)

- **Column detection**: `detectColumnIndices()` matches Indian bank headers:
  - **Withdrawals** = expenses (debit); **Deposits** = income (credit)
  - **Dr/Cr** = record type (source of truth when present): Dr = debit, Cr = credit
  - **Particulars** = narration for categorization
  - Also: `Chq.No/Dr`, `Chq.No/Cr`, `Value Date`, etc.
- **Amount parsing**: `parseAmount()` handles `₹`, `Rs`, `INR`, commas; preserves decimal (INR: 100.12 = 100 rupees, 12 paisa).
- **Categories**: Income, Groceries (Blinkit, BigBasket), Food & Dining (Swiggy, Zomato), E-commerce & Shopping (Amazon, Flipkart, Myntra), UPI & Transfers, Entertainment, Utilities, Transport, Subscriptions, Insurance, Loan & EMI, Healthcare, Investment, Other.
- **DR/CR logic**: Credit (Cr) = money in (income); Debit (Dr) = money out (expense). Categorization respects transaction type from amounts.

### Excel Extraction

- **Layout**: Lines 1–13 = personal info (display only, not used in calculations)
- **Line 16** = column headers (Value Date, Particulars, Dr/Cr, Withdrawals, Deposits, Balance, etc.)
- **Lines 17+** = transaction data
- **SheetJS**: `XLSX.read(arrayBuffer, { type: 'array' })`; `sheet_to_json` with `header: 1`

### Categorize with AI

- Sends parsed transactions (date, narration, amount, type) to Gemini with a fixed category list.
- AI returns one category per line; `normalizeAICategory()` maps variants to allowed names.
- `recomputeAggregatesFromParsed()` rebuilds byCategory and monthly/weekly; summary re-renders.

### Analyse with AI

- Sends full transaction data + analysis prompt to Gemini
- Prompt requests: spending patterns, income vs expenses, notable observations, suggestions, summary
- Response rendered as markdown in inline scrollable section

### Chat with AI

- User question + transaction data sent to Gemini per message
- Suggested questions: Total spending & income, Monthly spending, Weekly spending, Yearly summary, Top expense categories, Biggest spending areas
- Responses rendered with markdown; chat history preserved

### Timestamps & Timezone

- Footer "Last updated" and AI log timestamps use `timeZone: 'Asia/Kolkata'` (IST).
- `toLocaleTimeString('en-IN', { timeZone: 'Asia/Kolkata', hour12: true })` for chat and logs.

### AI API Logs

- **Categorize with AI**: Appends Categorize REQUEST and RESPONSE/ERROR.
- **Analyse with AI**: Replaces logs with REQUEST and RESPONSE
- **Chat with AI**: Appends each exchange (REQUEST, then RESPONSE/ERROR)
- Clear and Copy buttons for both Processing Steps and AI API Logs panels

---

## 11. Error Handling

| Scenario | User Message |
|----------|--------------|
| Wrong/missing password | "Incorrect or missing password. Please try again." |
| Invalid/corrupt PDF | "Failed to open PDF. [error details]" |
| Invalid Excel / parse error | "Failed to open Excel file. [error details]" |
| No tables found | "No table data found in the PDF. The format may not be supported." |
| Copy failed | "Could not copy. Please select and copy manually." |
| Gemini API error | "AI analysis failed: [error message]" |
