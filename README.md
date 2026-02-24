# Jupiter Bank Statement Analyzer

A client-side web application for uploading, unlocking, and analyzing bank statement PDFs and Excel files. Extracts account holder information and transaction data into spreadsheet-friendly tables with one-click copy to Excel or Google Sheets.

---

## Project Information

| Field | Value |
|-------|-------|
| **Project Name** | Jupiter Bank Statement Analyzer |
| **Version** | 1.4.0 |
| **Type** | Single-page web application |
| **License** | MIT |
| **Environment** | Browser (client-side only, no backend required) |

---

## Project Structure

```
JupiterBankStatementAnalyzer/
├── index.html              # Main application (HTML, CSS, JS)
├── README.md               # Project overview and quick start
├── CHANGELOG.md            # Summary of changes by version
├── LICENSE                 # MIT License
├── .gitignore
├── docs/
│   ├── LOGS.md             # Logs documentation (Processing Steps, AI API Logs)
│   ├── PRD.md              # Product Requirements Document
│   ├── TECHNICAL.md        # Technical architecture and implementation
│   ├── ARCHITECTURE.md     # System design and data flow
│   └── CHANGELOG_PROMPTS.md # Development prompts and solutions log
├── AccountStatement_*.pdf   # (optional) Sample PDF statement for testing
└── AccountStatement_*.xlsx  # (optional) Sample Excel statement for testing
```

---

## Features

### Core Features

| Feature | Description |
|---------|-------------|
| **PDF & Excel Upload** | Drag & drop or click to select bank statement PDFs or Excel (.xls, .xlsx) |
| **Smart PDF Detection** | Auto-detects unlocked vs password-protected PDFs on file select |
| **Conditional Password UI** | Password input shown **only** for protected PDFs; hidden for unlocked PDFs |
| **View / Unlock & View** | **View** button for unlocked PDFs; **Unlock & View** for protected (after entering password) |
| **Two-Table Extraction** | Separates **account holder info** (sensitive) from **transaction list** |
| **Excel Structure** | Lines 1–13 = personal info (display only); Line 16 = headers; Lines 17+ = transactions |
| **Personal Info Toggle** | Eye icon to show/hide account holder details |
| **Robust Table Extraction** | PDF: heuristics; Excel: fixed layout. **Withdrawals** (expenses), **Deposits** (income), **Dr/Cr** (type), **Particulars** (categorization) |
| **Spreadsheet View** | Renders extracted data in HTML tables |
| **Copy to Excel** | One-click copy as TSV for paste into Excel or Google Sheets |
| **Copy for Word/Document** | Copy as formatted table for paste into Word or other documents |
| **Download Unlocked PDF** | Save the decrypted PDF without password (hidden for Excel files) |
| **Download Summary PDF** | Export financial analysis and charts as PDF |
| **Summary & Charts** | Financial analytics: income vs expense, category breakdown, daily/weekly/monthly/yearly views, utilization (Expense/Income %, Savings rate) |
| **Auto-Categorization** | Regex-based categories: Income, Groceries (Blinkit, BigBasket), Food & Dining (Swiggy, Zomato), E-commerce (Amazon, Flipkart), UPI & Transfers, Entertainment, etc. |
| **Categorize with AI** | Use Gemini to assign categories per transaction; improves accuracy for UPI, merchants, cryptic narrations |
| **Theme Toggle** | Dark, Light, and System themes with persisted preference |
| **Analyse with AI** | One-click AI analysis via Gemini; spending patterns, suggestions, financial health summary |
| **Chat with AI** | Ask questions about your statement; monthly/weekly/yearly spending, top categories, usage |
| **AI API Logs** | View request/response for every AI call; Clear and Copy log buttons |
| **Debug Panel** | Processing steps with Clear/Copy; logs PDF extraction and AI requests |

### AI Features (Gemini)

| Feature | Description |
|---------|-------------|
| **Categorize with AI** | Assigns categories per transaction via Gemini; uses Income, Groceries, Food & Dining, E-commerce & Shopping, UPI & Transfers, Entertainment, etc. |
| **Analyse with AI** | Sends transaction data to Gemini; returns thorough analysis with amounts and breakdowns |
| **Chat with AI** | Q&A interface; suggested questions (monthly spending, top categories, etc.) or free-form |
| **Model Selector** | Choose model in AI Setup: Gemini 2.5 Flash (default), Gemini 3 Flash Preview, Gemini 2.5 Pro, Gemini 2.0 Flash, Gemini 1.5 Flash, Gemini 1.5 Pro |
| **API Key** | Enter in app when prompted; get free key at [Google AI Studio](https://aistudio.google.com/app/api-keys) or [docs](https://ai.google.dev/gemini-api/docs/api-key) |

### Security & Privacy

- **Client-side only** — no PDF data is sent to any server
- **In-memory processing** — PDF is read in browser memory only
- **Sensitive data handling** — Account holder info displayed separately with warning; excluded from clipboard copy
- **AI features** — Transaction data is sent to Google Gemini API; API key entered by user and stored in `localStorage` only

---

## How to Start the Application

> **Important:** The app uses ES modules and must be served over HTTP. Opening `index.html` directly (`file://`) will not work.

### Prerequisites

- A modern web browser (Chrome, Firefox, Safari, or Edge)
- One of: **Python 3**, **Node.js**, or **VS Code** with Live Server

---

### Option 1: Using Python (Recommended)

1. **Open Terminal** (macOS/Linux) or **Command Prompt/PowerShell** (Windows).

2. **Navigate to the project folder:**
   ```bash
   cd path/to/JupiterBankStatementAnalyzer
   ```
   Replace `path/to/JupiterBankStatementAnalyzer` with your actual project path.

3. **Start the HTTP server:**
   ```bash
   python3 -m http.server 8080
   ```
   (Use `python` instead of `python3` if that's how Python is installed on your system.)

4. **Open the app in your browser:**
   - Go to: **http://localhost:8080**
   - Or: **http://127.0.0.1:8080**

5. **Stop the server** when done: Press `Ctrl + C` in the terminal.

---

### Option 2: Using Node.js

1. **Open Terminal** in your project folder.

2. **Install serve** (one-time, if not already installed):
   ```bash
   npm install -g serve
   ```
   Or run without installing:
   ```bash
   npx serve .
   ```

3. **Start the server:**
   ```bash
   npx serve .
   ```
   Or, if you installed globally:
   ```bash
   serve .
   ```

4. **Open the URL** shown in the terminal (e.g. `http://localhost:3000`).

---

### Option 3: Using VS Code Live Server

1. **Install the Live Server extension:**
   - Open VS Code → Extensions (`Cmd/Ctrl + Shift + X`)
   - Search for **"Live Server"** by Ritwick Dey
   - Click **Install**

2. **Start the server:**
   - Open the project folder in VS Code
   - Right-click `index.html` in the Explorer
   - Select **"Open with Live Server"**

   Or use the status bar: click **"Go Live"** at the bottom right.

3. The app will open in your default browser automatically.

---

### AI Analysis Setup (Optional)

When you click **Categorize with AI**, **Analyse with AI**, or **Chat with AI**, you'll be prompted for your Gemini API key. Get a free key at [Google AI Studio → API Keys](https://aistudio.google.com/app/api-keys) or see the [API key docs](https://ai.google.dev/gemini-api/docs/api-key). Your key is stored in your browser (`localStorage`) and never sent to our servers.

### Verification

- You should see **"Bank Statement Analyzer"** with an upload zone.
- If the page is blank or shows module errors, ensure you are using `http://localhost` (or similar) and not a `file://` URL.

---

### Usage Flow

**PDF (unlocked):**
1. Upload your bank statement PDF
2. App auto-detects it is unlocked (shows "Checking PDF..." briefly)
3. Only **View** button appears (no password input)
4. Click **View** to extract and display data

**PDF (password-protected):**
1. Upload your bank statement PDF
2. App detects password protection
3. Password input and **Unlock & View** button appear
4. Enter password and click **Unlock & View**
5. Review the extracted tables

**Excel (.xls, .xlsx):**
1. Upload your bank statement Excel file
2. No password needed — **View** button appears immediately
3. Click **View**; first 13 lines show as personal info (eye icon to hide/show)
4. Lines 14–16+: column headers at line 16 (Value Date, Particulars, Dr/Cr, Withdrawals, Deposits), transaction data from line 17

**After extraction:**
- Use **Copy to Excel** / **Copy for Word** to paste into spreadsheets or documents
- Use **Download Unlocked PDF** to save a password-free copy of the statement
- Use **Summary & Charts** and **Download Summary PDF** for financial analysis
- Use **Categorize with AI** to improve transaction categories (Blinkit, UPI, Amazon, etc.)
- Use **Analyse with AI** for automated analysis (spending patterns, suggestions)
- Use **Chat with AI** to ask questions (e.g. "What is my monthly spending?")

---

## Documentation

| Document | Description |
|----------|-------------|
| [CHANGELOG.md](CHANGELOG.md) | Summary of changes by version (Added, Changed, Fixed) |
| [LOGS.md](docs/LOGS.md) | Logs documentation — Processing Steps, AI API Logs, format, troubleshooting |
| [PRD.md](docs/PRD.md) | Product Requirements Document — goals, user stories, acceptance criteria |
| [TECHNICAL.md](docs/TECHNICAL.md) | Technical documentation — stack, APIs, extraction logic |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | Architecture — data flow, component diagram, file structure |
| [CHANGELOG_PROMPTS.md](docs/CHANGELOG_PROMPTS.md) | Development prompts and solutions with timestamps |

---

## Tech Stack

- **Frontend**: Vanilla HTML, CSS, JavaScript (ES6 modules)
- **UI Framework**: Bootstrap 5.3.2
- **Icons**: Bootstrap Icons 1.11.1
- **PDF Engine**: PDF.js (Mozilla) v4.0.379 via CDN
- **Excel Engine**: SheetJS (xlsx) v0.20.3 via CDN
- **Charts**: Chart.js 4.4.1
- **AI**: Google Gemini 2.5 Flash via `@google/genai` SDK (ESM)
- **Styling**: Custom CSS with CSS variables, dark/light themes
- **Font**: Plus Jakarta Sans (Google Fonts)

---

## Supported Formats

### PDF
- Password-protected and unprotected PDFs
- Multi-page statements
- Bank statements with Table 1 (account holder info) and Table 2 (transactions)

### Excel (.xls, .xlsx)
- Fixed layout: Lines 1–13 = personal info; Line 16 = column headers; Lines 17+ = transactions
- **Column mapping**: Value Date, **Particulars** (description for categorization), **Dr/Cr** (debit/credit type), **Withdrawals** (expenses), **Deposits** (income)

### Currency (INR)
- Amounts in Indian Rupees (₹). Format: `100.12` = 100 rupees, 12 paisa (1 rupee = 100 paisa)

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

---

## License

MIT License. See [LICENSE](LICENSE) for details.
