# 📊 Ledger: Enterprise Multi-Currency Offline Workspace

Ledger is a modern, high-performance, privacy-focused financial logging application designed to run entirely client-side. Built intentionally as an offline-first Single Page Application (SPA), it empowers users to track complex multi-currency portfolios across distinct financial accounts without sending data to an external server.

---

## 🚀 Core Features

- **Smart Multi-Currency Accounting Engine:** Prioritizes localized currency handling (e.g., Singapore Dollar `S$`) with robust custom exchange rate scaling (FX) across dynamic accounts.
- **Interactive Balance Sheet (Net Savings Statement):** A dedicated financial portal organizing income and expenses into side-by-side matrices, offering direct, deep-linked transaction filtering.
- **Bi-Directional Inline Ledger Editor:** Modify accidental transaction errors or shift account structures instantly via a pre-filled structural modal pipeline.
- **Automated Sandbox Footprint Meter:** Displays hardware database allocation, tracking live IndexedDB byte limits and local storage consumption in real-time.
- **Custom Category Designer:** Register bespoke income/expense categories with custom emoji badges, stored persistently in IndexedDB.
- **Receipt & Photo Attachments:** Attach a photo to any transaction directly from the phone camera or gallery, auto-compressed client-side before storage.
- **Dual Account Types — Multi-Currency & Fixed Deposit:** Choose the right account model at creation time: flexible multi-currency wallets for everyday spending, or Fixed Deposit accounts with tenure, interest rate, and automatic maturity-date tracking.
- **Maturity Reminders:** The dashboard surfaces an upcoming/overdue banner for any Fixed Deposit within 30 days of maturity (or already matured), so renewals or withdrawals never get missed.
- **Installable Offline PWA:** Ships with a web manifest, app icons, and a Service Worker cache layer, so the ledger can be installed to a phone home screen and opened with zero network connectivity.
- **Absolute Privacy Preservation:** Zero servers, zero analytics, zero data logging. Data never leaves your physical local storage.

---

## 📱 Architecture & Evolution Log

### [v16.0] - Directional Transfer Colors & FD Placement Status Badges *(Current)*
- **Directional +/− for Transfers:** When viewing a specific account's ledger, transfer entries now show green **+** (money arriving) or red **−** (money leaving) instead of a neutral 🔄 for both directions — so closing a placement for renewal reads unmistakably as money leaving that account, and a new placement or renewal reads as money arriving. (Viewing "All" accounts keeps the neutral 🔄 style, since direction is ambiguous without a specific account in view.)
- **FD Placement Status Badges:** Every FD placement entry now shows its status at a glance: **🟢 Active** (still running), **⏰ Due** (matured, not yet actioned), or **✅ Closed** (already renewed or withdrawn) — so old, already-resolved placements are visually distinct from ones still in progress.

### [v15.0] - Capital vs. Income Fix, FD Reference Numbers, One-Time Data Repair
- **Fixed: Opening Balances & FD Placements Wrongly Counted as Income/Expense.** Opening balances, opening FD placements, FD renewal placements, and FD closures-for-renewal were being saved as `income`/`expense` transactions, which inflated the Total Income / Total Expense report card and cluttered "All Income Log" / "All Expense Log". These are now correctly saved as `transfer` — capital moving into tracking or being reinvested isn't earned income or a spend, so it no longer skews those totals. Genuine interest received (from a withdrawal or a principal-only renewal) is still correctly recorded as income.
- **One-Time Repair Tool for Existing Data:** A new **"🔧 Fix Legacy FD/Opening Balance Entries"** button (next to Export/Import JSON) scans your existing transactions for the affected auto-generated entries and converts them to transfers in place — account balances are unaffected, only their income/expense classification changes. Safe to run more than once; it never touches transactions you entered yourself.
- **Account / Reference Numbers for FD Placements:** You can now record a bank's certificate/account/reference number on any FD placement — at account creation, when logging a deposit, and when renewing. It's shown in the ledger entry and carried through automatically to that placement's closure/renewal/withdrawal transactions, so you can trace which certificate a given entry belongs to.

### [v14.0] - Maturity Resolution Workflow & Ledger Filter Fix
- **Fixed: Year Filter Silently Hid Older Transactions.** The ledger's Year dropdown defaulted to a hardcoded "2026," so viewing an account's activity could silently omit transactions dated in other years — most notably, an older Fixed Deposit placement (real commencing dates can predate the current year) simply wouldn't appear, even though it was correctly included in balance and net worth totals. Default changed to **"All Years"**, with the dropdown's year range widened for headroom.
- **Reminder Banners Now Clear Themselves:** Tapping a maturity reminder no longer just navigates to the account — it opens a dedicated **Resolve Maturity** dialog. Once you act on a placement (renew or withdraw), it's flagged internally so the reminder disappears from the dashboard automatically, without deleting or hiding the original placement record.
- **Renew Flow:** Enter the actual interest received (prefilled with the original projected estimate, editable), then choose:
  - **➕ Renew with Interest** — the interest is capitalized straight into the new placement's principal.
  - **🏛️ Principal Only** — only the original principal rolls into the new placement; the interest is paid out as income to an account of your choice.
  Either way, set the new placement's Commencing Date, Tenure, and Interest Rate — the Maturity Date auto-calculates, with a live preview of the new placement.
- **Withdraw & Close Flow:** Enter the actual interest received and pick a destination account — the principal is transferred out and the interest is recorded as income into that same account, closing the placement out entirely.
- **Multiple Placements, Independently Tracked:** Since each FD placement is resolved individually, an account holding several tranches (e.g. 4 separate FDs under one bank name) keeps a separate reminder — and a separate renew/withdraw action — for each one.

### [v13.0] - Opening Balances for Multi-Currency & Fixed Deposit Accounts
- **Multi-Currency Opening Balances:** Account creation for a Multi-Currency account now lets you seed one or more starting currency balances right away (e.g. open "Bank A" already holding both SGD and MYR) via repeatable "+ Add Currency" rows, instead of requiring a separate transfer afterward.
- **Multiple Fixed Deposit Placements at Creation:** Fixed Deposit account creation now supports adding several opening placements in one go — e.g. an account "Bank A FD" that already holds 4 existing certificates, each with its own currency, principal, commencing date, tenure, and interest rate (maturity auto-calculated per placement, with a live payout preview).
- **Backed by Real Transactions:** Each opening balance/placement row is saved as a normal "Opening Balance" (or "Opening Fixed Deposit Placement") transaction under the new account — so they show up in that account's ledger history and are automatically picked up by the maturity reminder banner, with no special-case logic needed elsewhere in the app.
- **Creation-Only, By Design:** These opening rows only appear when creating a brand-new account. Editing an existing account no longer shows them — add further funds or placements the normal way, via Income/Transfer entries, to keep the transaction history accurate.

### [v12.0] - Account Model Rework: Normal vs. True Multi-Currency vs. Fixed Deposit
- **Three Distinct Account Types:** Account creation is now a clean three-way choice — **🏛️ Normal** (single fixed currency, chosen once at creation — unchanged from earlier behavior), **💱 Multi-Currency** (one account name holding several independent currency balances side by side, e.g. "Bank A" with its own SGD and MYR baskets that never mix), and **🏦 Fixed Deposit** (a term-deposit account whose currency and terms are captured per placement, not at creation).
- **Minimal Account Creation:** Creating a Multi-Currency or Fixed Deposit account now only asks for a name — no currency, no opening balance, no term fields. Everything else is captured naturally as you log transactions.
- **True Currency Baskets:** Multi-Currency and Fixed Deposit accounts no longer auto-convert incoming funds into one native currency. Each currency you transact in in gets its own running balance ("basket") shown side by side in the accounts list — depositing SGD into "Bank A" only ever affects its SGD basket, never its MYR basket. Normal accounts keep the original behavior (everything converts into their one fixed currency).
- **FD Terms Moved to the Transaction:** Fixed Deposit placement details — Commencing Date, Tenure, Interest Rate, and the auto-calculated Maturity Date — now appear directly in the transaction form whenever you log a deposit into an FD account, instead of at account creation. This naturally supports multiple tranches/placements in the same FD account, each with its own maturity date.
- **Reminders Now Track Every Placement:** The dashboard's maturity reminder banner scans all FD deposit transactions (not just one date per account), so if an FD account has several placements, each one gets its own upcoming/overdue reminder.
- **Account Dropdowns Clarified:** Transaction form account pickers now show 💱/🏦 prefixes and omit the currency suffix for Multi-Currency/Fixed Deposit accounts (since they don't have one fixed currency), while Normal accounts still show their currency as before.

### [v11.0] - Fixed Deposit Accounts & Maturity Reminders
- **Account Type Selector:** Account creation/edit now starts with a choice — **💱 Multi-Currency Account** (today's flexible wallet/bank account behavior, unchanged) or **🏦 Fixed Deposit Account** (a new dedicated type for term deposits).
- **Fixed Deposit Terms:** FD accounts capture Commencing Date, Tenure (months), and Annual Interest Rate; the Maturity Date is auto-calculated from commencing date + tenure (and recalculated live as you edit the form), alongside a projected payout preview using simple interest.
- **Maturity Dashboard Banner:** Any FD within 30 days of maturity — or already past it — surfaces a color-coded reminder banner at the top of the dashboard (amber = upcoming, red = overdue), tapping it jumps straight to that account's edit screen.
- **FD Badges Everywhere:** FD accounts are labeled with a `🏦 FD · rate% · maturity date` badge in the accounts list, the account management list, and prefixed in transaction account dropdowns, so they're never confused with regular wallets.
- **Transfers Still Work Normally:** FD accounts remain fully usable as transfer source/destination (e.g. moving funds in to open a new FD, or out at maturity/withdrawal) — no changes needed to how you log deposits or payouts.

### [v10.0] - Receipt & Photo Attachments
- **Camera Capture & Gallery Picker:** The transaction form now has "Take Photo" and "Gallery" buttons, using native `<input type="file">` capture on Android to open the camera app or the OS photo picker directly — no extra permissions dialog or plugin required.
- **Client-Side Image Compression:** Selected photos are downscaled (max 1024px on the long edge) and re-encoded as JPEG (~70% quality) via an off-screen canvas before being stored, keeping the IndexedDB footprint reasonable even for high-resolution phone camera photos.
- **Inline Preview & Removal:** A thumbnail preview appears in the form immediately after capture/selection, with a one-tap ✕ to remove and reselect before saving.
- **Receipt Badge in Ledger List:** Transactions with an attached photo show a small 📎 icon next to the date; tapping it opens a fullscreen image viewer without needing to open the edit form.
- **Backup-Compatible:** Attached images travel with their transaction record automatically in Export/Import JSON backups — no separate export step needed.
- **Graceful Storage-Limit Handling:** If a device runs out of local storage while saving a photo, the app surfaces a clear "not enough storage space" message instead of silently failing.

### [v9.0] - Installable Offline PWA & GitHub Pages Deployment
- **True Offline Support:** Added `sw.js` (Service Worker) implementing a cache-first strategy for the app shell, so the ledger opens and functions with airplane mode on, once installed.
- **Web App Manifest:** Added `manifest.json` + PNG icon set (192×192 / 512×512), enabling a proper "Install" prompt on Android Chrome — full standalone window, no browser chrome, instead of a plain bookmark shortcut.
- **Static Hosting:** Restructured the project for deployment on GitHub Pages (`index.html` at repo root) so the app has a stable `https://` origin — a requirement for Service Worker registration, which local `file://` pages cannot satisfy.
- **Origin-Aware Bootstrapping:** Service Worker registration is feature-detected and protocol-guarded (`https:`/`http:` only), so the same `index.html` still degrades gracefully and runs fine when opened locally without a server.

### [v8.0] - Data Integrity, Validation & Reliability Hardening
- **Fixed Silent Write Failures:** `writeDB` / `readAllDB` / `deleteDB` previously had no error channel — a failed IndexedDB transaction (e.g. a missing object store after a schema change) would hang forever with no feedback. All database helpers now reject on `tx.onerror`/`request.onerror`, and every call site surfaces a clear alert on failure instead of freezing silently.
- **Fixed Missing Object Store on Upgrade:** Bumped `DB_VERSION` and made `onupgradeneeded` idempotent (`objectStoreNames.contains` checks per store), fixing a bug where users who had opened an earlier build of the file never received the `categories` object store, silently breaking "Add Category."
- **Fixed Orphaned Transactions on Account Deletion:** Deleting an account previously only removed the account record — transactions referencing it as source/destination were left behind as orphaned data. Account deletion now cascades and removes all linked transactions, with the confirmation dialog wording updated to state this explicitly.
- **Added Input Validation:** Transaction amount, account initial balance, and FX rate inputs are now checked for `NaN` / non-positive values before being written, rejecting garbage input (empty strings, negative numbers, non-numeric text) with a clear message instead of silently persisting invalid records.
- **Added Import Confirmation Guard:** Restoring a JSON backup now requires an explicit confirm step before it wipes and replaces all current accounts/transactions/categories — previously a backup file selection immediately overwrote all data with no warning.
- **Fixed Non-Awaited Store Clearing:** Backup import's "clear existing data" step called `.clear()` on an `IDBRequest` directly inside an `await`, which does not actually wait for completion. Replaced with a properly Promise-wrapped `clearStoreDB` helper.
- **Consistent Confirm UI:** Replaced native browser `confirm()` popups (delete account / delete category / delete transaction / import backup) with an in-app modal styled to match the rest of the workspace.
- **Ledger List Pagination:** Long transaction histories are now rendered in pages of 50 with a "Load more" button, instead of building and mounting the full DOM list on every render — keeps scrolling smooth as transaction count grows.

### [v7.0] - Android WebView & Tap-Latency Optimization
- **Touch-Safe Handlers:** Converted structural form execution from standard HTML inputs to explicit programmatic `button` nodes to isolate virtual keyboard interference on mobile.
- **Double-Tap Mitigation:** Applied native CSS `touch-action: manipulation` overrides to drop mobile browser delays by 300ms across Android Chrome.
- **Target Area Scaling:** Standardized touch assets to a strict minimum layout footprint of `48px` to guarantee physical taps execute seamlessly on phone devices like the Xiaomi 14T.

### [v6.0] - Cross-Platform Security & Theming
- Deployed system preference event hooks allowing the ledger workspace to smoothly mirror device Dark Mode settings dynamically.
- Finalized local hardware quota tracking localization.

### [v5.0] - Structural Modal Architecture
- Rebuilt item creation workflows into independent modal layering sheets to safeguard document scale layouts while inputs are active.
- Integrated dashboard counters tracking structural counts inside active tags.

### [v4.0] - Advanced Editing & Financial Deep-Linking
- Upgraded static metric widgets into live portals to filter ledger tables automatically.
- Deployed global account reconfiguration logic to allow post-creation asset updates.

---

## 🛠️ Tech Stack & Local Execution

This application operates entirely inside a single isolated context using native browser technologies:

- **UI & Layout:** Pure HTML5 structure accented by optimized modern semantic flex grids.
- **Database Layer:** Asynchronous `IndexedDB` pipeline handling persistence and structural record lookups.
- **Offline Layer:** `Service Worker` (`sw.js`) cache-first strategy + `Web App Manifest` (`manifest.json`) for installability.
- **Media Layer:** Native `<input capture>` for camera/gallery access + `Canvas` API for client-side JPEG compression of receipt photos.
- **Dependencies:** 100% dependency-free. Zero external libraries, packages, or network calls are required.

### How to Run Locally (offline, no install):
1. Copy or download the `index.html` file to your machine or phone.
2. Open the file inside any browser (Chrome, Safari, or mobile Android WebView environments).
3. Note: without HTTPS hosting, the Service Worker cache won't register — the app still runs fine, it just won't get the installable "Add to Home Screen → Install" experience.

### How to Run as an Installable App (recommended, full offline PWA):
1. Host `index.html`, `manifest.json`, `sw.js`, `icon-192.png`, and `icon-512.png` together at the root of a static site (e.g. this repo's GitHub Pages).
2. Open the deployed URL on an Android phone in Chrome.
3. Tap the menu (⋮) → **Install app** (not "Create shortcut" — that only bookmarks the page and skips the Service Worker/standalone window).
4. Once installed, the app opens in its own window and works fully offline (try it in airplane mode).

---

## 💾 Data Backups & Redundancy

Because data storage is tied explicitly to your browser's internal local database (and to the specific origin — `file://` vs. a hosted `https://` URL are treated as separate databases), clearing browser data, switching hosting origins, or reinstalling can affect records.

- Use the built-in **Export JSON** backup payload option regularly to save physical ledger copies to your phone or computer.
- Restore historical financial tracking at any moment on a new device or new hosting origin using the **Import JSON** node (a confirmation prompt now guards against accidental overwrites — see v8.0 above).
