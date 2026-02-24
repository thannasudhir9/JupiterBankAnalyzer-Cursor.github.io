# Logs Documentation

**Project:** Jupiter Bank Statement Analyzer  
**Last Updated:** February 25, 2026 (IST)

---

## Overview

The application has two separate logging systems:

| Panel | Purpose | Location |
|-------|---------|----------|
| **Processing Steps** | PDF/Excel extraction, table detection, column mapping | Debug panel (collapsible) |
| **AI API Logs** | Gemini requests and responses (Categorize, Analyse, Chat) | AI API Logs panel |

All timestamps use **IST (Asia/Kolkata)**.

---

## 1. Processing Steps (Debug Panel)

### Format

Each step is shown as:

```
[TIME] ICON MESSAGE
       [optional detail]
```

| Icon | Status | Meaning |
|------|--------|---------|
| ✓ | success | Step completed successfully |
| ▶ | active | Step in progress |
| ○ | pending | Awaiting action or fallback used |
| ✕ | error | Step failed |

**Time format:** `HH:MM:SS.mmm` (24-hour)

### File Upload & Probe

| Step | Status | When |
|------|--------|------|
| PDF file selected | success | User selects PDF; shows filename and size (KB) |
| Excel file selected | success | User selects .xls/.xlsx; View button appears (no password) |
| PDF is unlocked | success | Probe load without password succeeds |
| PDF is password-protected | pending | Probe fails with PASSWORD_REQUIRED; "Enter password to unlock" |

### Extraction Flow

| Step | Status | When |
|------|--------|------|
| Extract initiated | active | Unlock & View clicked |
| Reading PDF file into memory... | active | Loading file |
| File read complete | success | Shows size in KB |
| Loading PDF... | active | Unprotected PDF load |
| Loading password-protected PDF... | active | Protected PDF load with password |
| Using cached unlocked PDF | success | Reusing cached doc |
| PDF unlocked successfully | success | Shows page count |
| Extracting text and tables from all pages... | active | Starting extraction |
| Processing page N of M... | active | Per-page extraction |
| Table split complete | success | Personal info rows, transaction rows |
| Fallback: using all extracted rows as transactions | pending | No header/row heuristics matched |

### Table Detection

| Step | Status | When |
|------|--------|------|
| Table 1 identified: Account holder info (sensitive) | success | Personal info block found; shows row count |
| Table 1: No personal info block detected | pending | No personal info rows |
| Table 2 identified: Transaction list | success | Transaction block found; shows row count |
| Table 2: No transaction data found | error | No transaction rows extracted |
| Extraction failed: No tables detected | error | Both tables empty |
| All processing complete | success | Extraction finished successfully |

### Column Detection

| Step | Status | When |
|------|--------|------|
| Column detection | success | Shows headers; Date/Withdrawals/Deposits/Dr/Cr/Particulars indices; layout |
| Excel table split complete | success | Personal info (lines 1–13), transactions (header line 16, data 17+) |

### AI Operations

| Step | Status | When |
|------|--------|------|
| Sending request to Gemini | active | Analyse with AI; shows prompt size (KB) |
| AI analysis complete | success | Analyse response received |
| AI analysis failed | error | Gemini API error |
| Categorizing transactions with AI | active | Categorize with AI clicked |
| AI categorization complete | success | Categories applied |
| AI categorization failed | error | Parse or API error |
| Chat: sending question to Gemini | active | Chat message sent |
| Chat: response received | success | Chat response received |
| Chat failed | error | Chat API error |

### Error Steps

| Step | Status | When |
|------|--------|------|
| PDF is password-protected | error | Password required |
| Wrong password | error | Incorrect password |
| Error: [message] | error | Generic extraction failure |

### Actions

- **Clear** — Removes all Processing Steps
- **Copy** — Copies steps as plain text to clipboard

---

## 2. AI API Logs

### Format

Each log section has a title and body:

```
[TIME] LABEL — REQUEST
────────────────────
(request/response body)
```

or

```
[TIME] LABEL — RESPONSE
────────────────────
(response body)
```

or

```
[TIME] LABEL — ERROR
────────────────────
(error message)
```

**Time format:** 12-hour IST (e.g. `10:30 PM`)

### Categorize with AI

| Section | Content |
|---------|---------|
| Categorize — REQUEST | Prompt with transaction list + allowed categories |
| Categorize — RESPONSE | AI-returned category names (one per line) |
| Categorize — ERROR | Failure message if API or parse fails |

### Analyse with AI

| Section | Content |
|---------|---------|
| REQUEST | Full analysis prompt + transaction data |
| RESPONSE | Gemini's markdown analysis |
| ERROR | API error if failed |

**Note:** Analyse with AI **replaces** the entire AI logs (does not append).

### Chat with AI

| Section | Content |
|---------|---------|
| Chat — REQUEST | User question + transaction data |
| Chat — RESPONSE | Gemini's answer |
| Chat — ERROR | API error if failed |

**Note:** Each chat exchange **appends** a new REQUEST and RESPONSE/ERROR pair.

### Actions

- **Clear** — Resets AI logs to placeholder text
- **Copy** — Copies all log sections (title + body) as plain text

---

## 3. Log Text Format (Copy Output)

### Processing Steps (copied)

```
[14:32:01.123] ✓ PDF file selected
  statement.pdf (245.3 KB)
[14:32:02.456] ▶ Extract initiated
  Using unlocked PDF
[14:32:03.789] ✓ PDF unlocked successfully
  4 page(s) found
...
```

### AI API Logs (copied)

```
[10:30 PM] Categorize — REQUEST

You are a transaction categorizer. Assign exactly ONE category...
Transactions (numbered 1 to N):
1. 01-Dec-2025 | NEFT IN | ₹5000 | income
...

[10:30 PM] Categorize — RESPONSE

Food & Dining
UPI & Transfers
Income
...
```

---

## 4. Technical Reference

| Function | Purpose |
|----------|---------|
| `debugLog(msg, status, detail)` | Append to Processing Steps |
| `debugClear()` | Clear Processing Steps |
| `getDebugLogsText()` | Get Processing Steps as plain text |
| `updateAiLogs(request, response, error)` | Replace AI logs (Analyse) |
| `appendAiLogs(label, request, response, error)` | Append REQUEST to AI logs |
| `appendAiLogResponse(label, response, error)` | Append RESPONSE/ERROR to AI logs |
| `getAiLogsText()` | Get AI logs as plain text |

---

## 5. Troubleshooting

| Issue | Check |
|-------|-------|
| No Processing Steps | Ensure PDF was selected and View/Unlock clicked |
| "No tables detected" | PDF format may not match heuristics; check Debug for page-processing steps |
| Column detection wrong | See "Column detection" step; verify header names in your PDF |
| AI logs empty | Click Categorize with AI, Analyse with AI, or Chat with AI first |
| Copy fails | Browser may block clipboard; try selecting and copying manually |



Local Logs

cd /Users/sthanna/Downloads/VSCode-NewMac/JupiterBankStatementAnalyzer && python3 -m http.server 8765
::1 - - [18/Feb/2026 09:54:41] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 09:54:42] code 404, message File not found
::1 - - [18/Feb/2026 09:54:42] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 09:57:55] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 09:57:55] code 404, message File not found
::1 - - [18/Feb/2026 09:57:55] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:04:04] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:11:16] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:11:17] code 404, message File not found
::1 - - [18/Feb/2026 10:11:17] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:19:48] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:19:49] code 404, message File not found
::1 - - [18/Feb/2026 10:19:49] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:21:11] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:21:12] code 404, message File not found
::1 - - [18/Feb/2026 10:21:12] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:23:32] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:23:32] code 404, message File not found
::1 - - [18/Feb/2026 10:23:32] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:27:56] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:27:57] code 404, message File not found
::1 - - [18/Feb/2026 10:27:57] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:30:42] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:30:43] code 404, message File not found
::1 - - [18/Feb/2026 10:30:43] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:30:43] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:30:44] code 404, message File not found
::1 - - [18/Feb/2026 10:30:44] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:33:41] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:33:42] code 404, message File not found
::1 - - [18/Feb/2026 10:33:42] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:36:23] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:36:23] code 404, message File not found
::1 - - [18/Feb/2026 10:36:23] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:41:28] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:41:28] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:41:30] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:41:30] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:41:30] code 404, message File not found
::1 - - [18/Feb/2026 10:41:30] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:43:49] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:43:49] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:43:50] code 404, message File not found
::1 - - [18/Feb/2026 10:43:50] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:48:50] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:48:50] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:48:51] code 404, message File not found
::1 - - [18/Feb/2026 10:48:51] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 10:53:24] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:53:24] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 10:53:24] code 404, message File not found
::1 - - [18/Feb/2026 10:53:24] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 11:04:48] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 11:04:48] "GET /api-config.js HTTP/1.1" 200 -
::1 - - [18/Feb/2026 11:04:50] code 404, message File not found
::1 - - [18/Feb/2026 11:04:50] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 22:58:31] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 22:58:33] code 404, message File not found
::1 - - [18/Feb/2026 22:58:33] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:00:24] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:00:26] code 404, message File not found
::1 - - [18/Feb/2026 23:00:26] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:04:05] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:04:05] code 404, message File not found
::1 - - [18/Feb/2026 23:04:05] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:06:32] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:06:32] code 404, message File not found
::1 - - [18/Feb/2026 23:06:32] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:06:33] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:06:33] code 404, message File not found
::1 - - [18/Feb/2026 23:06:33] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:08:05] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:08:05] code 404, message File not found
::1 - - [18/Feb/2026 23:08:05] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:09:03] code 404, message File not found
::1 - - [18/Feb/2026 23:09:03] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:19:42] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:19:44] code 404, message File not found
::1 - - [18/Feb/2026 23:19:44] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:19:50] "GET / HTTP/1.1" 304 -
::1 - - [18/Feb/2026 23:20:50] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:20:50] code 404, message File not found
::1 - - [18/Feb/2026 23:20:50] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:21:12] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:12] code 404, message File not found
::1 - - [18/Feb/2026 23:21:12] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:21:13] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:14] code 404, message File not found
::1 - - [18/Feb/2026 23:21:14] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:21:14] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:14] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:14] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:14] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:21:15] code 404, message File not found
::1 - - [18/Feb/2026 23:21:15] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:22:31] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:25:59] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:26:00] code 404, message File not found
::1 - - [18/Feb/2026 23:26:00] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:26:01] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:26:02] code 404, message File not found
::1 - - [18/Feb/2026 23:26:02] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:28:28] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:28:28] code 404, message File not found
::1 - - [18/Feb/2026 23:28:28] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:28:29] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:28:29] code 404, message File not found
::1 - - [18/Feb/2026 23:28:29] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:31:44] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:31:46] code 404, message File not found
::1 - - [18/Feb/2026 23:31:46] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:39:17] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:39:18] code 404, message File not found
::1 - - [18/Feb/2026 23:39:18] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:42:29] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:42:30] code 404, message File not found
::1 - - [18/Feb/2026 23:42:30] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:45:16] code 404, message File not found
::1 - - [18/Feb/2026 23:45:16] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:45:32] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:45:32] code 404, message File not found
::1 - - [18/Feb/2026 23:45:32] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:45:34] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:45:34] code 404, message File not found
::1 - - [18/Feb/2026 23:45:34] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:01] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:03] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:03] code 404, message File not found
::1 - - [18/Feb/2026 23:46:03] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:03] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:03] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:04] code 404, message File not found
::1 - - [18/Feb/2026 23:46:04] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:04] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:04] code 404, message File not found
::1 - - [18/Feb/2026 23:46:04] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:36] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:36] code 404, message File not found
::1 - - [18/Feb/2026 23:46:36] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:36] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:37] code 404, message File not found
::1 - - [18/Feb/2026 23:46:37] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:37] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:37] code 404, message File not found
::1 - - [18/Feb/2026 23:46:37] "GET /favicon.ico HTTP/1.1" 404 -
::1 - - [18/Feb/2026 23:46:49] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:46:50] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:47:24] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:47:29] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:47:31] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:49:05] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:53:00] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:53:26] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:53:27] "GET / HTTP/1.1" 200 -
::1 - - [18/Feb/2026 23:53:54] "GET / HTTP/1.1" 200 -
