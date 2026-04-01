# SiteBook — Complete Project Summary v2
> Read this once at the start of a new chat. Contains everything needed to continue building immediately.

---

## 1. PROJECT IDENTITY

| Item | Detail |
|------|--------|
| **App Name** | SiteBook |
| **Tagline** | Daily Transactions · Vendors · Documents · All in One Place |
| **Live URL** | https://devopsqa0506.github.io/nirmanam/ |
| **GitHub Repo** | devopsqa0506/nirmanam (public) |
| **Current File** | `sitebook_v2.html` — single file, 4974 lines, 668KB |
| **Target User** | Small builder from Kadapa, Andhra Pradesh |
| **Languages** | Telugu + English (bilingual UI) |
| **Platform** | PWA — works on iPhone Safari, Android Chrome, Desktop |

---

## 2. TECH STACK

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML5 + CSS3 + JavaScript ES6 — **single file, no frameworks** |
| Hosting | GitHub Pages (free, auto-deploy on git push, ~15 sec) |
| AI | Google Gemini 2.5 Flash (`gemini-2.5-flash:generateContent`) |
| Backend | Google Apps Script Web App (doPost/doGet) |
| Database | Google Sheets (user's own account, 6 sheets) |
| File Storage | Google Drive via DriveApp in Apps Script |
| Auth | Google Identity Services (GSI) — OAuth 2.0 |
| QR Decode | jsQR 1.4.0 (loaded dynamically from cdnjs) |
| QR Generate | Google Charts API (image-based, no library needed) |
| Fonts | Noto Sans Telugu + JetBrains Mono (Google Fonts) |

**CRITICAL: No npm, no webpack, no React, no Node.js, no Firebase, no separate server.**

---

## 3. HTML STRUCTURE

```
<!DOCTYPE html>
<html lang="te">
<head>
  Meta tags, PWA tags, Google Fonts, Google GSI script
  Single <style> block — ALL CSS here
</head>
<body>
  #login-screen     — Google Sign In screen
  #splash           — App logo + name (2.5s)
  #setup            — 4-step setup wizard
  #app              — Main app
    .hdr            — Header: logo, title, balance pill, user avatar
    .nav            — Bottom nav bar (7 tabs)
    #sc-home        — Expense entry + recent list
    #sc-chat        — AI Chat with Gemini
    #sc-projects    — Construction projects + ROI
    #sc-brain       — AI Brain (Insights + Alerts + Auto Entry)
    #sc-contacts    — Contacts with WhatsApp
    #sc-docs        — Scanned property documents + bills
    #sc-settings    — All configuration
  MODALS (7):
    #guide-modal, #house-modal, #ec-modal, #qr-modal,
    #client-id-modal, #script-url-modal, #script-modal
  #proc             — Processing overlay with timer
  #toast            — Notification (10 second display)
  Single <script>   — ALL JS here (170+ functions)
</body>
```

---

## 4. DESIGN STYLE

```css
--bg: #08090d           /* main background — near black */
--s1: #0f1117           /* surface 1 */
--s2: #161b26           /* surface 2 */
--s3: #1e2535           /* surface 3 */
--s4: #263040           /* surface 4 */
--bd: #2a3347           /* border */
--bd2: #384560          /* border 2 */
--tx: #dce8f0           /* primary text */
--tx2: #7a90a8          /* secondary text */
--tx3: #3a4a5a          /* tertiary text */
--amb: #e8b84a          /* amber — primary accent */
--amb2: #f5d070         /* amber light */
--teal: #3de8c8         /* teal — secondary accent */
--sky: #5aafe8          /* sky blue */
--rose: #e85d7a         /* error/warning red */
--sage: #7ac85a         /* success green */
```

Typography: Noto Sans Telugu (body), Noto Serif Telugu (headings), JetBrains Mono (numbers/codes)
Style: Dark theme, flat design, no gradients, mobile-first

---

## 5. FEATURES IMPLEMENTED

### Authentication
- Google Login via GSI (Google Identity Services)
- Client ID: `849554560698-0a45clrf9sp5ufvsnvufgenivod0sksg.apps.googleusercontent.com`
- JWT decoded client-side — name, email, picture saved to cfg
- User avatar pill in header — click shows dropdown (Sign Out / Settings)
- Sign Out → hides app → shows login screen (proper flow)
- Auto-login on return visits via Google One Tap

### Daily Expense Tracking
- Types: `order` (expense), `payment`, `advance`
- Categories: Sand, Cement, Steel, Labour, Electrical, Hardware, Land, Other
- Fields: vendor, amount, qty, unit, unitPrice, date, notes, project
- Auto vendor creation + balance tracking
- Price history per vendor+category

### AI Document Scanning — UNIVERSAL (Registration + Bills)
- Upload image or PDF (any size)
- **STEP 1**: Gemini auto-detects doc type first (Registration / Agreement / Bill / Other)
- **STEP 2**: Extracts right fields based on detected type

#### Registration/Agreement extraction:
- 24+ fields: docNumber, date, survey, extent, seller, buyer, phones, Aadhaar, boundaries, financials, ownership chain
- **Phone rule**: 10 digits starting 6-9 = phone ONLY
- **Aadhaar rule**: 12 digits = Aadhaar ONLY — NEVER mix
- **Extent rule**: always show sq yards AND cents both
- **Date rule**: from FIRST LINE of document only
- **Ownership chain**: ALL transactions, not just latest

#### Bill/Invoice extraction (NEW in v2):
- `vendorName`, `vendorPhone`, `vendorAddress` — vendor contact details from bill
- `invoiceNumber`, `documentDate`, `amount`, `gstAmount`
- `lineItems[]` — **EVERY line item** extracted:
  - `itemName` (exact as printed)
  - `itemKey` (Gemini-normalized key, e.g. `borewell_drilling`, `tmt_steel_10mm`)
  - `quantity`, `unit`, `unitRate`, `lineTotal`
  - `confidence`

### Bill Intelligence (NEW in v2)
After each bill scan, automatically:

1. **Price History Tracking** — each line item saved to `state.billPriceHistory[itemKey]` with date, vendor, rate, qty, total
2. **Price Change Alerts** — compares new rate vs last known rate for same item:
   - ≥5% increase → red alert card with old/new rate, %, vendor, date
   - ≥5% decrease → green card (good news)
3. **Bill Total Mismatch Detection** — sums all lineItems, compares to printed total. If >2% difference → "⚠️ Bill total mismatch — ₹X vs ₹Y, check for hidden charges"
4. **Vendor Auto-added to Contacts** — bill vendor added to contacts with phone (if on bill), role=vendor
5. **"Add to Ledger" Confirm Card** — after every bill scan, shows confirm card with vendor, amount, line items table → one tap → `addEntry()` → ledger + Google Sheet synced

### Duplicate Document Detection (smart, type-aware)
- **Registration/Agreement**: Check by Document Number, then seller+buyer+date combo
- **Bill**: Check by vendor+amount (not filename)
- **Filename check (all types)**: If same filename exists locally → confirm dialog:
  - "Already exists locally. Drive लो manually delete చేశారా? Re-upload to Drive?" 
  - OK → re-uploads to Drive only (no re-scan, no duplicate local entry)
  - Cancel → skip

### Google Drive Integration
```
SiteBook Docs/
  Properties/
    Survey_620_1/    ← Survey number ONLY (consistent, no location text)
  Bills/
    2026-03/         ← YYYY-MM
  Other/
```
- Images compressed (max 1200px, JPEG 75%) before upload
- `no-cors` POST to Apps Script (fire and forget)
- `driveUploaded` flag set on doc record after upload

### Dynamic Processing Message (based on file size)
```
< 0.5MB  → "15-25 seconds"
0.5-2MB  → "25-40 seconds"
2-5MB    → "40-60 seconds"
5-15MB   → "1-1.5 minutes"
> 15MB   → "1-2 minutes"
```

### AI Brain Module (5 Layers)
1. **AI Insights** — 7 sections including new "Bill Price Trends" which analyzes billPriceHistory cross-vendor
2. **AI Chat** — full project data context, bill data included, answers "last time borewell cost emi?"
3. **Auto Entry** — "Paid 20k to Ramesh for sand" → JSON → confirm → save
4. **Document AI** — scan feature (Registration + Bill)
5. **Smart Alerts** — price hikes >10%, overdue >7 days, budget risks

### Multiple Gemini API Keys
- Stored in `cfg.apiKeys[]`
- Round-robin rotation via `getActiveKey()`
- Auto-rotation on 429 quota error

### QR Code Config Transfer
- Format: `SITEBOOK:` + base64(JSON)
- Contains: sheetId, scriptUrl, geminiKey, all apiKeys
- Generate: Google Charts API → canvas image
- Scan: image upload ONLY (camera removed — unreliable on mobile)

### EC Search (IGRS AP Portal)
- Shows doc number, year, SRO in large font for manual entry
- Opens https://registration.ec.ap.gov.in/ecSearch

### Contacts
- Auto-populated from Registration/Agreement scans (buyer/seller)
- Auto-populated from Bill scans (vendor name + phone)
- Manual add supported
- WhatsApp button per contact

---

## 6. localStorage STRUCTURE

### Key: `nirmit_cfg_v1`
```json
{
  "sheetId": "GOOGLE_SHEET_ID",
  "scriptUrl": "https://script.google.com/macros/s/.../exec",
  "geminiKey": "AIzaSy...",
  "apiKeys": [{"key":"AIzaSy...","label":"Gmail1","addedDate":"25 Mar 2026"}],
  "keyRotateIdx": 0,
  "setupDone": true,
  "googleClientId": "849554560698-...apps.googleusercontent.com",
  "userEmail": "user@gmail.com",
  "userName": "Ravi Kumar",
  "userPicture": "https://...",
  "loggedIn": true
}
```

### Key: `nirmit_data_v1`
```json
{
  "entries": [],
  "vendors": {},
  "priceHistory": {},
  "billPriceHistory": {
    "borewell_drilling": [
      {"date":"15.03.2026","vendor":"XYZ Drilling","unitRate":250,"unit":"ft","lineTotal":45000,"quantity":180,"docId":"d123"}
    ],
    "tmt_steel_10mm": []
  },
  "documents": [],
  "projects": [],
  "contacts": [],
  "nextId": 1,
  "chatHistory": [],
  "lastSync": null
}
```

**NEW in v2**: `billPriceHistory` — key = Gemini-normalized itemKey, value = array of price records per item across all vendors and dates.

---

## 7. GOOGLE SHEETS STRUCTURE

### Entries
`id | date | type | vendor | category | amount | description | unitPrice | priceKey`

### Vendors
`name | category | phone | totalOrdered | totalPaid | balance | lastDate`

### Prices
`priceKey | price | date | unit`

### Documents
`id | date | type | propertyType | summary | fileName`

### Contacts
`id | name | phone | role | address | notes | addedDate`

### Projects
`id | name | landCost | totalCents | housesPlanned | status | location | date | expectedSalePerHouse`

---

## 8. APPS SCRIPT API (v5 — Latest)

Deploy: Web App, Execute as: Me, Who: Anyone (no auth required)
All writes: `method: POST, mode: no-cors` (fire-and-forget)

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var ss = SpreadsheetApp.openById(data.sheetId);
    var action = data.action;
    if (action === "append") {
      var sheet = ss.getSheetByName(data.sheetName) || ss.insertSheet(data.sheetName);
      if (sheet.getLastRow() === 0) sheet.appendRow(data.headers);
      sheet.appendRow(data.row);
      return ok({});
    }
    if (action === "readAll") {
      var result = {};
      ["Entries","Vendors","Prices","Documents","Contacts","Projects"].forEach(function(name) {
        var sh = ss.getSheetByName(name);
        result[name] = (sh && sh.getLastRow() > 1) ? sh.getDataRange().getValues() : [];
      });
      return ok({data: result});
    }
    if (action === "saveFile") {
      var rootFolder = getOrCreateFolder("SiteBook Docs", DriveApp.getRootFolder());
      var folder;
      if (data.fileType === "Registration" || data.fileType === "Agreement" || data.fileType === "EC") {
        var propsFolder = getOrCreateFolder("Properties", rootFolder);
        folder = getOrCreateFolder(sanitize(data.propertyFolder || "Unknown_Property"), propsFolder);
      } else if (data.fileType === "Bill") {
        var billsFolder = getOrCreateFolder("Bills", rootFolder);
        folder = getOrCreateFolder(data.monthFolder || getYearMonth(), billsFolder);
      } else {
        folder = getOrCreateFolder("Other", rootFolder);
      }
      var blob = Utilities.newBlob(Utilities.base64Decode(data.fileBase64), data.mimeType, data.fileName);
      var file = folder.createFile(blob);
      file.setDescription(data.description || "");
      return ok({fileId: file.getId(), fileUrl: file.getUrl(), fileName: file.getName()});
    }
    if (action === "ping") { return ok({}); }
  } catch(err) {
    return ContentService.createTextOutput(JSON.stringify({ok:false,error:err.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
function ok(extra) {
  var obj = Object.assign({ok: true}, extra);
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(ContentService.MimeType.JSON);
}
function getOrCreateFolder(name, parent) {
  var iter = parent.getFoldersByName(name);
  if (iter.hasNext()) return iter.next();
  return parent.createFolder(name);
}
function sanitize(name) {
  return name.replace(/[^a-zA-Z0-9_\-\.\s]/g, "").trim().substring(0, 50);
}
function getYearMonth() {
  var d = new Date();
  return d.getFullYear() + "-" + ("0"+(d.getMonth()+1)).slice(-2);
}
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({ok:true,msg:"SiteBook API v5"}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 9. KEY FUNCTIONS REFERENCE

```javascript
// Core
init()                    // startup: login/splash/setup/app
launchApp()               // show app, hide login/splash
showSc(name)              // switch screen
toast(msg, type)          // 10-second notification
showProc(text)/hideProc() // loading overlay + timer

// Data
addEntry(entry)           // add expense/payment
pushToSheet(name,hdrs,row)// Google Sheets sync (no-cors)
syncFromSheet()           // readAll from Sheet
freshData()               // {entries,vendors,priceHistory,billPriceHistory,documents,...}

// Document Scanning
handlePhoto(e, source)         // upload + scan document (Registration OR Bill)
DOC_PROMPT                     // universal 2-step prompt: detect type → extract fields
processBillLineItems(result, docId)  // saves lineItems to billPriceHistory, returns alerts[]
autoAddBillVendorContact(result)     // adds bill vendor to contacts
buildBillAlertHtml(billAlerts)       // renders price change / mismatch alert cards
buildBillConfirmCard(result, docId)  // "Add to Ledger?" card with line items table
confirmBillEntry(entryJsonStr, btn)  // saves bill amount as expense entry
buildDocResultHtml(r)                // renders bill OR registration result in chat
autoAddContactFromDoc(result)        // adds buyer/seller from Registration to contacts

// AI
callGemini(imageData, imageMime, docPromptOverride)
buildDataSummary()         // includes billPriceHistory + recentBills for AI context
runAIInsights()            // 7-section analysis including Bill Price Trends
runAIAlerts()              // Smart Alerts
processAutoEntry()         // text to entry

// Helpers
fmtIN(n)                  // Indian number format: 12,73,000
rup(n)                    // rupee format: ₹12,73,000
fixAmountStr(str)         // normalize amount string
fv(field)                 // parse Gemini field value (handles null/N/A/object/string)
escH(s)                   // HTML escape
getActiveKey()            // rotate API keys

// Auth
initGoogleLogin()
handleGoogleLogin(resp)
signOut()
updateUserUI()

// Drive
saveToDrive(b64,mime,name,docType,result,docRef)
compressAndUpload(file,docType,result,docRef)

// QR
generateQR()
scanQRFromImage(e)
applyQRConfig(qrText)
```

---

## 10. RULES — NEVER CHANGE THESE

| Rule | Why |
|------|-----|
| Single `index.html` file | Architecture decision — no split |
| localStorage keys `nirmit_cfg_v1` / `nirmit_data_v1` | Backward compatibility |
| Gemini model: `gemini-2.5-flash` | Do not change to pro or 1.5 |
| `no-cors` for Sheet/Drive writes | Required — Apps Script CORS limitation |
| Phone=10 digits, Aadhaar=12 digits | Never mix in DOC_PROMPT |
| Drive folder = Survey Number only | Location text from Gemini varies per scan |
| Toast = 10 seconds | User requested (was 3s, too fast to read) |
| Always verify function is CALLED | Biggest past mistake — wrote function but forgot to call it |
| `billPriceHistory` key in freshData | New field — must stay for bill price tracking to work |
| itemKey from Gemini for bill items | Normalized by Gemini — must stay consistent across bills |

---

## 11. IMPORTANT DECISIONS MADE

| Decision | Reason |
|----------|--------|
| Gemini normalizes `itemKey` for bill items | Prevents "Borewell drilling" vs "BW drill" mismatch — AI is better at normalization than regex |
| Price alert threshold: ≥5% change | 5% meaningful for construction materials, avoids noise |
| Bill total mismatch: >2% difference | Accounts for rounding, flags real errors |
| Filename duplicate check → confirm re-upload | Drive manually deleted → user can re-upload without re-scanning |
| `billPriceHistory` separate from `priceHistory` | `priceHistory` is for manual entry-based prices; bill prices are richer (vendor, qty, rate) |
| Bill vendor contact auto-add | vendorPhone validated (10 digits, starts 6-9) before saving — Aadhaar can't sneak in |
| `buildDataSummary()` includes `billPriceHistory` | AI Chat can now answer cross-vendor price questions |
| Registration/Agreement dedup checks scoped only to those types | Bills don't have doc numbers — prevents false positives |

---

## 12. BUGS FIXED (all sessions combined)

| Bug | Fix |
|-----|-----|
| `c.phone.replace is not a function` | `String(c.phone).replace()` |
| Duplicate Drive folders (spelling varies) | Survey Number only as folder name |
| Duplicate Drive uploads | Dup check stops processing before upload |
| Amount `1,273,000` instead of `12,73,000` | `fmtIN()` + `fixAmountStr()` |
| QR button click — nothing happened | `document.getElementById('qr-file-input').click()` |
| Sign out didn't show login screen | `signOut()` shows `#login-screen`, `launchApp()` hides it |
| Processing message wrong time estimate | Dynamic message based on file size |
| Bill upload got Registration template | Universal DOC_PROMPT that detects type first |
| Same filename hard-blocked re-upload | Changed to confirm dialog → allows Drive re-upload |
| Bill dedup checked filename (wrong) | Bill dedup now checks vendor+amount only |
| `buildDocResultHtml` showed empty financials for bills | isBill branch renders bill-specific layout |
| `renderDocList` showed stamp duty for bills | isBill branch shows vendor, line items, total |
| `buildBillConfirmCard` referenced `file` variable | Fixed — `file` not in scope outside `handlePhoto`, use `result.summary` instead |

---

## 13. PENDING FEATURES (Priority Order)

### Priority 1 — Do Next
- **Expense entry category = "Bill"** — when bill confirm card is tapped, the category should be inferred from `itemKey` (e.g. `borewell_drilling` → "Other", `tmt_steel` → "Steel", `cement_opc` → "Cement")
- **Price history viewer** — a screen or panel in Docs/Brain showing cross-vendor price history per item with a mini trend chart
- **Android APK** — pwabuilder.com (needs PWA manifest added first — currently missing)

### Priority 2 — Near Term
- **iOS** — Safari "Add to Home Screen" already works as PWA
- **Netlify deployment** — `sitebook.netlify.app` free URL
- **Large PDF upload** — files >10MB fail (no-cors payload limit). Fix: OAuth token from Gmail login → Drive API directly with resumable upload

### Priority 3 — Future
- Auto Google Sheet creation after login (Sheets API)
- SaaS model — already works per-user via QR/setup wizard
- AI Insights charts/graphs (billPriceHistory is perfect for this)
- Budget alerts
- Export project report to PDF
- Material price comparison across vendors (foundation already built with billPriceHistory)

---

## 14. USER CONTEXT

- Builder from Kadapa, Andhra Pradesh
- Builds small houses on 3-cent plots
- iPhone (primary) + Laptop
- Telugu + English speaker
- Claude Pro subscriber
- Distribution plan: APK via WhatsApp to Kadapa builders
- Revenue target: ₹99-299/month per builder (SaaS)
- Key frustration: Functions defined but not called, repeated mistakes

---

*SiteBook v2.0 — Summary as of March 2026*
*Current file: sitebook_v2.html (4974 lines, 668KB)*
