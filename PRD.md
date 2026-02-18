# Product Requirements Document (PRD)

## Jupiter Bank Statement Analyzer

**Version:** 1.2  
**Last Updated:** February 19, 2025

---

## 1. Overview

### 1.1 Product Summary

Jupiter Bank Statement Analyzer is a web-based tool that allows users to upload bank statement PDFs (including password-protected ones), unlock them with a user-provided password, and extract tabular data into a spreadsheet-ready format. The primary use case is to copy transaction data to Excel or Google Sheets for analysis.

### 1.2 Problem Statement

- Bank statements are often distributed as password-protected PDFs
- Manually retyping or copying data from PDFs is tedious and error-prone
- Users need a simple way to get transaction data into spreadsheets without specialized software

### 1.3 Solution

A single-page web app that:
- Accepts PDF uploads (drag & drop or file picker)
- Unlocks password-protected PDFs
- Extracts two logical tables: account holder info and transactions
- Displays data in tables and supports one-click copy as TSV (Excel/Sheets compatible)

---

## 2. Goals & Objectives

| Goal | Priority | Success Metric |
|------|----------|----------------|
| Unlock password-protected PDFs | P0 | User can view content after entering correct password |
| Extract transaction table | P0 | Transaction rows appear in table format |
| Copy to Excel/Sheets | P0 | Pasted data preserves columns and rows |
| Separate personal info from transactions | P1 | Two distinct tables with sensitive data clearly labeled |
| Provide debug visibility | P2 | User can see processing steps for troubleshooting |

---

## 3. User Personas

### Primary User

- Individual with bank accounts (e.g., Jupiter, other banks)
- Receives statements as PDF (sometimes password-protected)
- Wants to analyze transactions in Excel or Google Sheets
- Non-technical; expects simple upload → unlock → copy flow

---

## 4. User Stories

### Epic: PDF Upload & Unlock

| ID | Story | Acceptance Criteria | Priority |
|----|-------|---------------------|----------|
| US-1 | As a user, I can upload a PDF by drag & drop | PDF is accepted; file name is shown | P0 |
| US-2 | As a user, I can upload a PDF by clicking to browse | File picker opens; PDF is accepted | P0 |
| US-3 | As a user, I see password input only for protected PDFs | Unlocked PDF: no password box, only View button; Protected: password box + Unlock & View | P0 |
| US-4 | As a user, the app auto-detects if my PDF is locked | On file select, app probes PDF; shows appropriate UI (View vs Unlock & View) | P0 |
| US-5 | As a user, I can unlock protected PDFs with password | Password field appears for protected PDFs; Enter or Unlock & View triggers unlock | P0 |
| US-6 | As a user, I see an error for wrong password | Clear error message; can retry | P0 |

### Epic: Data Extraction & Display

| ID | Story | Acceptance Criteria | Priority |
|----|-------|---------------------|----------|
| US-7 | As a user, I see account holder info in a separate table | Table 1 shows; marked as sensitive | P1 |
| US-8 | As a user, I see transactions in a table | Table 2 shows with headers and rows | P0 |
| US-9 | As a user, I can copy transactions to clipboard | One click; paste works in Excel/Sheets | P0 |
| US-10 | As a user, I do not copy personal info by default | Copy button copies only transactions | P1 |
| US-11 | As a user, I can view financial summary and charts | Income vs expense, category breakdown, monthly trends | P1 |

### Epic: AI Analysis

| ID | Story | Acceptance Criteria | Priority |
|----|-------|---------------------|----------|
| US-12 | As a user, I can get AI-powered analysis of my statement | Analyse with AI sends data to Gemini; shows thorough analysis inline | P1 |
| US-13 | As a user, I can chat with AI about my transactions | Chat with AI; ask questions; get detailed answers with amounts | P1 |
| US-14 | As a user, I see suggested questions in Chat | Quick buttons: monthly spending, top categories, etc. | P1 |
| US-15 | As a user, I can view AI request/response logs | AI API Logs panel shows what was sent and received | P2 |

### Epic: Developer Experience / Debug

| ID | Story | Acceptance Criteria | Priority |
|----|-------|---------------------|----------|
| US-16 | As a developer/user, I can see processing steps | Debug panel shows steps; Clear and Copy buttons | P2 |

---

## 5. Functional Requirements

### 5.1 Must Have (P0)

- [ ] Accept PDF file upload (drag & drop, file picker)
- [ ] Password input for protected PDFs
- [ ] Unlock PDF with correct password
- [ ] Extract text/tables from PDF
- [ ] Display transactions in HTML table
- [ ] Copy transactions as TSV to clipboard
- [ ] Show loading state during extraction
- [ ] Show error for incorrect password

### 5.2 Should Have (P1)

- [ ] Identify and separate account holder info (Table 1)
- [ ] Identify and separate transaction list (Table 2)
- [ ] Label sensitive data section with warning
- [ ] Exclude personal info from copy

### 5.3 Nice to Have (P2)

- [ ] Collapsible debug panel with step-by-step log
- [ ] AI API Logs with Clear and Copy
- [ ] Chat with AI for Q&A on transaction data

---

## 6. Non-Functional Requirements

### 6.1 Performance

- PDF processing completes within reasonable time (< 30s for typical statements)
- No server round-trips; all processing in browser

### 6.2 Security & Privacy

- No PDF content sent to external servers (except transaction data to Gemini API for AI features)
- Password used only for local decryption
- Sensitive data displayed only in user’s browser

- API key stored in gitignored config file or localStorage

### 6.3 Compatibility

- Modern browsers: Chrome, Firefox, Safari, Edge (latest 2 versions)
- Requires HTTP server (ES modules); `file://` not supported

### 6.4 Usability

- Minimal steps: upload → password → unlock → copy
- Clear error messages
- Responsive layout for common screen sizes

---

## 7. Constraints

- Client-side only (no backend)
- PDF format must be parseable by PDF.js
- Table extraction is heuristic-based; layout may vary by bank

---

## 8. Out of Scope (v1)

- Server-side processing
- User authentication or persistence
- PDF generation or editing
- Support for scanned/image-based PDFs (OCR)

---

## 9. Success Metrics

- User can complete full flow (upload → unlock → copy) in under 2 minutes
- Copy-paste into Excel/Sheets produces correct columns and rows
- No user data leaves the browser
