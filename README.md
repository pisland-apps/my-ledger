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

### [v11.0] - Fixed Deposit Accounts & Maturity Reminders *(Current)*
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
