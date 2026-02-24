# Changelog

All notable changes to Jupiter Bank Statement Analyzer are documented here.

---

## [1.4.0] — 2026-02-25 (IST)

### Added
- **Excel support** — Upload bank statements in .xls or .xlsx format in addition to PDF
- **Excel layout** — Lines 1–13 = personal info (display only); Line 16 = headers; Lines 17+ = transactions
- **Personal info toggle** — Eye icon to show/hide account holder details
- **Chart period selector** — Daily, Weekly, Monthly, Yearly views for income/expense charts
- **Utilization metrics** — Expense/Income ratio and Savings rate in summary
- **INR paisa support** — Amounts display up to 2 decimal places (100.12 = ₹100 and 12 paisa)

### Changed
- **Column detection** — Withdrawals = expenses, Deposits = income, Dr/Cr = type, Particulars = categorization
- **Dr/Cr as source of truth** — When present, used to determine debit vs credit per row
- **Amount parsing** — Fixed decimal preservation for INR (rupees.paisa); handles Excel numeric values

### Fixed
- `INR_FORMAT` self-reference that caused initialization error
- Excel date serial numbers and Date objects in `parseDateFromCell`

---

## [1.3.0] — 2026-02-18 (IST)

### Added
- **Categorize with AI** — Use Gemini to assign categories per transaction; improves accuracy for UPI, Blinkit, Amazon, etc.
- Expanded category rules: **Groceries** (Blinkit, BigBasket), **Food & Dining** (Swiggy, Zomato), **E-commerce & Shopping** (Amazon, Flipkart, Myntra), **UPI & Transfers**, **Entertainment**, and more

### Changed
- **DR/Cr detection** — Improved column detection for Indian bank PDFs (Dr, Cr, Chq.No/Dr, Chq.No/Cr, Withdrawals, Deposits)
- **Amount parsing** — Handles ₹, Rs, INR, commas, dashes; Indian number format
- **Timezone** — All timestamps (footer, AI logs, chat) now use IST (Asia/Kolkata)
- Categorization respects DR/Cr-derived transaction type (income vs expense)

### Fixed
- Transaction analysis for Indian bank statement formats
- Blank or dash cells in amount columns now parse correctly

---

## [1.2.0] — 2025-02-19

### Added
- **Analyse with AI** — One-click financial analysis via Gemini
- **Chat with AI** — Ask questions about your statement (monthly spending, top categories, etc.)
- **AI API Logs** — View request/response for every AI call
- **Model Selector** — Choose Gemini model (2.5 Flash, 3 Flash Preview, 2.5 Pro, etc.)

### Changed
- API key now user-entered only (no api-config.js)
- AI Setup card with links to Google AI Studio

---

## [1.1.0] — 2025-02-18

### Added
- **Smart PDF detection** — Auto-detects unlocked vs password-protected PDFs
- **Conditional password UI** — View button for unlocked; Unlock & View for protected
- **Summary & Charts** — Income vs expense, category breakdown, monthly trends
- **Theme toggle** — Dark, Light, System themes
- **Copy to Excel/Word** — TSV for Excel; formatted table for Word

### Changed
- Robust table extraction with broader header/pattern detection
- Fallback logic for varied bank statement formats

---

## [1.0.0] — Initial release

- PDF upload (drag & drop, file picker)
- Password unlock for protected PDFs
- Two-table extraction (personal info + transactions)
- Copy transactions to clipboard
- Basic categorization

---
