# M4L-C Handoff Doc — March 21, 2026

## What is M4L-C?
A personal finance PWA (Progressive Web App) for tracking spending, budgets, goals, and understanding the real cost of credit card debt. Single-file architecture — everything lives in `app.html` (~2500 lines). Hosted at **m4l.app** via Vercel, deployed from GitHub on push to `main`.

## Tech Stack
- **Frontend:** Single HTML file (`app.html`) — vanilla JS, no framework
- **Backend:** Supabase (auth, database for transactions/budgets/goals/settings)
- **Hosting:** Vercel (auto-deploys from GitHub `main` branch)
- **PWA:** Service worker (`sw.js`) with network-first caching. Cache version must be bumped on each deploy (currently `m4lc-v4`)
- **Fonts:** Syne (headers), DM Sans (body), Fraunces (italic accents), Michroma (logo)
- **Design:** Dark theme — deep green/black background, teal (#00D09C) primary, orange (#F5A623) for credit/warnings, red (#FF6B6B) for negative, acid green (#D4E84A) accent

## App Structure (app.html)
- **Lines 1-16:** Head, meta tags, font/script imports
- **Lines 17-250:** CSS styles (all inline in `<style>` block)
- **Lines 251-685:** HTML — auth screen, app shell, 5 page containers, modals (txModal, budModal, goalModal, catDetailModal, filteredTxModal, statementModal), action sheet
- **Lines 686-707:** Statement X-Ray modal HTML
- **Lines 709+:** JavaScript

### Pages (5 tabs)
1. **Home** (`page-home`) — Monthly overview, savings hero, recent transactions
2. **Budget** (`page-budget`) — Category budgets with progress bars
3. **Goals** (`page-goals`) — Savings goals with progress tracking
4. **Interest** (`page-interest`) — Credit card cost visualization, statement upload
5. **History** (`page-history`) — Full transaction list with filters

### Key JavaScript Sections
- **State:** Global `S` object holds user, profile, settings, transactions, budgets, goals, cards
- **Auth:** Supabase email/password + Google OAuth
- **Data loading:** `loadAll()` -> loadTx, loadBudgets, loadGoals, loadCards
- **Rendering:** Each page has a render function (renderHome, renderBudgetList, renderGoalsList, renderInterestPage, etc.)
- **Modals:** open/close pattern using `.modal-bg.open` class toggle
- **Utilities:** `fmtMoney()`, `esc()` (XSS-safe), `calcRealCost()`, `monthKey()`

## What We Built Today (3 commits)

### 1. Statement Upload Feature (`ff31e54`)
- Added "Upload Credit Card Statement" button to Interest page (visible in both empty and populated states)
- Created Statement X-Ray modal with CSV file upload
- Client-side CSV parser that:
  - Auto-detects columns (Description, Amount, Date, Debit/Credit variants)
  - Handles quoted fields
  - Separates purchases from bank charges (interest, fees, penalties — matched by keyword)
  - Shows two summary cards: What You Bought (teal) vs What The Bank Took (red)
  - Percentage bar showing purchase/bank split
- 100% client-side — no data sent to Supabase
- Key functions: `openStatementUploadModal()`, `parseStatementCSV()`, `resetStatementModal()`

### 2. Real-Cost Transaction Breakdown (`88a8e1b`)
- Each purchase shows original price (strikethrough) + real price (with proportional interest share) + markup % badge
- Sorted by biggest hidden cost first
- Math: `interest_share = (tx_amount / total_purchases) * total_bank_charges`
- Summary callout showing overall markup percentage
- Auto-detects date column for display

### 3. Worth It Review Mode (`f3e5712`)
- Interactive one-at-a-time transaction rating exercise
- "Rate Your Decisions" section with "Start Review" button
- Review card shows transaction details with three rating buttons (Worth It / Meh / Not Worth It) + Skip
- Progress bar and counter
- Smooth 0.3s opacity transitions between cards
- Summary screen after completion:
  - Three stat cards with totals per rating
  - Proportional spending bar
  - Contextual insight callouts (regret spending, interest on regrets, investment projection at 10% for 10 years)
  - "Save Insights" copies summary to clipboard
- Ratings appear as emojis in the transaction list after review
- Key functions: `startWorthItReview()`, `showReviewCard()`, `rateTransaction()`, `showReviewSummary()`, `exitReviewMode()`, `copyReviewSummary()`
- Global state: `window._stmtPurchases` (array with rating data), `_reviewIdx` (current position)

## Important Notes for Next Session

### Deployment
- **Changes only go live after `git push origin main`** — the app is hosted, not local
- **Bump `sw.js` cache version** on every push (currently `m4lc-v4`, increment to v5 next time)
- User may need to unregister service worker in DevTools if stale cache persists

### Architecture Patterns
- All HTML is generated via template literals in JS render functions (no templating library)
- Modals use `<div class="modal-bg" id="xxxModal">` with `.open` class to show
- Inline styles everywhere (no CSS classes for most elements)
- State is in the global `S` object, loaded from Supabase on auth
- `esc()` function used for XSS prevention in user-generated content

### File Overview
```
app.html        — The entire app (HTML + CSS + JS, ~2500 lines)
index.html      — Landing/marketing page
sw.js           — Service worker (network-first, bump CACHE version on deploy)
manifest.json   — PWA manifest
vercel.json     — Vercel routing config
icons           — apple-touch-icon.png, icon-192.png, icon-512.png
```

### CSS Variables
```
--bg: #071E1C        (main background)
--bg2: #060F0E       (modal background)
--card: #0C2624      (card background)
--teal: #00D09C      (primary/positive)
--ac: #D4E84A        (accent/gold-green)
--sand: #E4F4F3      (primary text)
--body: rgba(228,244,243,0.75)  (body text)
--muted: rgba(228,244,243,0.5)  (secondary text)
--div: rgba(228,244,243,0.06)   (borders)
--red: #FF6B6B       (negative/warnings)
Orange: #F5A623      (credit card/interest — not a CSS var, used inline)
```
