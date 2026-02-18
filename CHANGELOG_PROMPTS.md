# Development Prompts & Solutions Log

**Project:** Jupiter Bank Statement Analyzer  
**Format:** Date & Time | Prompt | Solution

---

## February 18, 2026

### Entry: API Key Security Fix
**Prompt:**
> API key was leaked via public GitHub repo. Make API key fully user-entered; remove api-config.js reliance; add info card with links for getting a key.

**Solution:**
- Removed `api-config.js` script tag; API key now comes only from user input and localStorage.
- Removed `window.GEMINI_API_KEY` from `getStoredApiKey()`.
- Added **AI Setup** collapsible card on the page with links to [Google AI Studio API Keys](https://aistudio.google.com/app/api-keys) and [API key docs](https://ai.google.dev/gemini-api/docs/api-key).
- Updated API key modal with both links and clearer instructions.
- Updated README, TECHNICAL.md, ARCHITECTURE.md; deprecated api-config.example.js.

---

## February 19, 2025

### Entry 4  
**Prompt:**
> Add a button called analyse with ai. Lets use gemini ai api key and use our transaction list data to send and receive the feedback analysis from gemini.

**Solution:**
- Added **Analyse with AI** button; integrates with Google Gemini via `@google/genai` SDK.
- Uses `gemini-2.5-flash`; API key from `api-config.js` (gitignored) or modal input.
- Sends full transaction data; displays analysis inline (no modal) with markdown rendering.

---

### Entry 5  
**Prompt:**
> Use these as default and hide this Sensitive Information (API key)

**Solution:**
- Created `api-config.js` with default key; added to `.gitignore`.
- Created `api-config.example.js` template.
- When key in config: password input hidden; Analyse with AI uses it directly.

---

### Entry 6  
**Prompt:**
> Show the analysis information properly with good text UI and not in a pop up window.

**Solution:**
- Moved AI analysis from modal to inline card with `.ai-analysis-prose` styles.
- Added markdown rendering (headings, bullets, bold).
- Scrollable area with `max-height: 480px`.

---

### Entry 7  
**Prompt:**
> SHOW AI Analysis WINDOW PROPERLY, AS I CANT SCROLL INSIDE IT WHEN IT IS DISPLAYING MORE CONTENT.

**Solution:**
- Wrapped content in `.ai-analysis-scroll-area` with `max-height: 480px`, `overflow-y: auto`.
- Custom scrollbar styling for better UX.

---

### Entry 8  
**Prompt:**
> Show logs properly, what is being sent, what is being received when I click the analyse with ai button. Add clear logs, copy logs button also. Make these window panels placement properly.

**Solution:**
- Added **AI API Logs** panel showing REQUEST (sent) and RESPONSE (received).
- Processing Steps and AI API Logs in side-by-side row (col-lg-6 each).
- Clear and Copy buttons on both panels.
- Scrollable log areas with max-height 320px.

---

### Entry 9  
**Prompt:**
> Lets add option to chat with ai with my bank information, and ask few questions with this bank statement information data like what is my usage, what is my monthly, weekly, yearly spending analysis etc questions.

**Solution:**
- Added **Chat with AI** card: input, Send button, message history.
- Suggested questions: Total spending & income, Monthly spending, Weekly spending, Yearly summary, Top expense categories, Biggest spending areas.
- Each question sent to Gemini with full transaction data; responses shown with markdown.
- Clear chat button.

---

### Entry 10  
**Prompt:**
> SHOW CHAT WITH AI DEBUG LOGS ALSO.

**Solution:**
- Chat requests/responses appended to AI API Logs panel.
- `appendAiLogs()` for REQUEST; `appendAiLogResponse()` for RESPONSE/ERROR.
- Processing Steps also logs "Chat: sending question", "Chat: response received".

---

### Entry 11  
**Prompt:**
> Also dont limit the response from ai, as I see very little information being received from ai even though we are sending full bank statement information.

**Solution:**
- Increased `maxOutputTokens` from 2048 to 8192.
- Updated prompts: "Be thorough and detailed" instead of "Be concise"; "Include specific amounts, totals, breakdowns".

---

### Entry 12  
**Prompt:**
> NOW UPDATE ALL DOCUMENTATION WITH NEW FEATURES AND ALL INFORMATION TECHNICAL, FUNCTIONAL AND ALL.

**Solution:**
- **README.md:** v1.2.0; AI features table; Chat with AI; AI API Logs; api-config; security note.
- **PRD.md:** Epic AI Analysis (US-12 to US-15); Epic Debug (US-16); security updates.
- **TECHNICAL.md:** @google/genai; key functions; Gemini config; AI Features section; error handling.
- **ARCHITECTURE.md:** UI components; state; core functions; file structure; security.
- **CHANGELOG_PROMPTS.md:** This document updated with entries 4–12.

---

## February 18, 2025

### Entry 1  
**Time:** ~10:00–10:15  
**Prompt:**
> No table data found in the PDF. The format may not be supported.
> Facing this error when I click unlock and view.
> Let's show the button based on secured pdf vs unlocked pdf:
> - Unlock and view button if secured pdf
> - View button if unlocked pdf

**Solution:**
- **Dynamic button labels:** Show "View" when password is empty (for unlocked PDFs) and "Unlock & View" when password has content.
- Added `updateUnlockButtonLabel()` called on password input change and file select.
- **Robust table extraction:**
  - Expanded `TRANSACTION_HEADER_LABELS` to include `value|amount|withdrawal|deposit|dr|cr|chq|cheque`.
  - Tightened `HEADER_FOOTER_PATTERNS` so valid rows (e.g. "NEFT to Federal Bank") are not filtered out.
  - Broader date/amount patterns in `looksLikeTransactionRow()` for formats like `18-Feb-2026`, `INR 1,234.56`, `Rs 1000`.
  - Fallback: if no transactions found but rows exist, treat all rows as transactions and auto-detect header.

---

### Entry 2  
**Time:** ~10:20–10:30  
**Prompt:**
> THERE IS NO NEED TO SHOW PASSWORD INPUT BOX WHEN IT IS UNLOCKED PDF

**Solution:**
- **PDF probe on file select:** After selecting a file, the app tries to load the PDF without a password.
- **Unlocked PDF:** If load succeeds → hide password input completely; show only "View" button; cache `currentPdfDoc` for reuse.
- **Secured PDF:** If load fails with `PASSWORD_REQUIRED` → show password input; show "Unlock & View" button (disabled until password entered).
- Introduced `pdfIsSecured` and `passwordInputWrapper`; password input wrapped in a div that toggles visibility.
- `unlockAndExtract()` reuses cached PDF for unlocked documents instead of reloading.
- Shows "Checking PDF..." on button while probing.

---

### Entry 3  
**Time:** ~11:00  
**Prompt:**
> Update documentation with all the new features, changes and all.
> Add all prompts and solutions with date and time in a new readme file.

**Solution:**
- **README.md:** Updated version to 1.1.0; added Smart PDF Detection, Conditional Password UI, View/Unlock & View behavior, robust extraction, Summary & Charts, categorization, theme toggle; revised usage flow for unlocked vs protected PDFs; updated tech stack (Bootstrap, Chart.js, Plus Jakarta Sans).
- **docs/PRD.md:** Version 1.1; new user stories US-3, US-4 for conditional UI and auto-detection; renumbered stories; added US-11 for Summary & Charts.
- **docs/TECHNICAL.md:** Updated tech stack; expanded table split heuristics and fallback algorithm; added `handleFile` probe flow, `updateUnlockButtonLabel`, `analyzeTransactions`, `updateCharts`.
- **docs/ARCHITECTURE.md:** Updated data flow with probe branching; new UI components (PasswordInputWrapper, SummarySection, ThemeToggle); new state (currentPdfDoc, pdfIsSecured, analysisData); new functions.
- **docs/CHANGELOG_PROMPTS.md:** New file listing prompts and solutions with dates (this document).

---

## Earlier Development (Summarized)

### Features Implemented Earlier

| Feature | Description |
|---------|-------------|
| **Transaction Analysis** | Parse debit/credit columns; compute totals and categorize |
| **Categorization** | Auto-categorize by narration (income, expenses, investment) |
| **Charts** | Income vs expense bar, category pie, monthly trend |
| **Summary PDF** | Download analysis summary as PDF |
| **Theme Toggle** | Dark, Light, System with localStorage persistence |
| **Copy to Excel/Word** | TSV for Excel; formatted table for Word |
| **Debug Panel** | Collapsible log with timestamps and status |

---

## Document History

| Date | Changes |
|------|---------|
| 2025-02-18 | Initial CHANGELOG_PROMPTS.md with three entries |
| 2025-02-18 | Documentation updates across README, PRD, TECHNICAL, ARCHITECTURE |
| 2025-02-19 | Entries 4–12: AI features, Chat, Logs, prompts; full documentation refresh |
