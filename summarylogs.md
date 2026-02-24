Summary of changes (v1.4.0):

1. **Excel support (.xls, .xlsx)**
   - Upload bank statements in Excel format
   - Layout: Lines 1–13 = personal info (display only); Line 16 = column headers; Lines 17+ = transactions
   - SheetJS (xlsx) for parsing

2. **Column mapping**
   - Withdrawals = expenses (debit)
   - Deposits = income (credit)
   - Dr/Cr = record type (source of truth)
   - Particulars = description for categorization

3. **Personal info toggle**
   - Eye icon to show/hide account holder details

4. **Charts & summary**
   - Chart period selector: Daily, Weekly, Monthly, Yearly
   - Utilization metrics: Expense/Income %, Savings rate

5. **INR currency**
   - Format: 100.12 = 100 rupees, 12 paisa (1 rupee = 100 paisa)
   - Amounts display up to 2 decimal places

---

Summary of earlier changes:
1. Expanded category rules (regex-based)
Categories now explicitly cover:
Category	Keywords / Merchants
Groceries	Blinkit, BigBasket, Grofers, Dunzo, grocery, supermarket
Food & Dining	Swiggy, Zomato, FoodPanda, Dominos, Pizza, Burger, McDonald's, KFC, restaurant, cafe, food, dine
E-commerce & Shopping	Amazon, Flipkart, Myntra, Meesho, Ajio, Nykaa, Snapdeal, shopping, e-commerce, retail
UPI & Transfers	UPI, Paytm, GPay, PhonePe, BharatPe, VPA, @paytm, @okaxis, @ybl, @axl, NEFT, IMPS, RTGS
Entertainment	Movie, cinema, BookMyShow, games, gaming, Steam, PlayStation, Xbox, ticket
Utilities	Electricity, broadband, mobile, Jio, Airtel, Vi, BSNL, recharge
Transport	Uber, Ola, Rapido, IRCTC, flight, train, bus, metro
Subscriptions	Netflix, Spotify, Prime, Hotstar, Disney, YouTube, Apple One, Google One
Healthcare	Apollo, 1mg, PharmEasy, medical, pharmacy
Income, Investment, Rent & Housing, Insurance, Loan & EMI stay as before.
2. “Categorize with AI” button
New Categorize with AI button next to Analyse with AI.
Uses Gemini to assign a category to each transaction based on narration.
Uses these fixed categories: Income, Investment, Rent & Housing, Groceries, Food & Dining, Utilities, Transport, E-commerce & Shopping, UPI & Transfers, Subscriptions, Insurance, Loan & EMI, Healthcare, Entertainment, Other.
AI handles variants and unclear narrations (e.g. UPI IDs) by inferring category.
After categorization, the summary and charts are refreshed automatically.
Flow: Upload PDF → extract transactions → click Categorize with AI → Gemini categorizes each transaction → summary updates → optionally click Analyse with AI for the written analysis.
Regex rules still run on extraction for basic categorization; Categorize with AI can refine or replace those with AI-assigned categories.