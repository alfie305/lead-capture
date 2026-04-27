# Coral Gables Buyer's Guide — Lead Capture Page

**Live URL:** https://lead-capture-amber.vercel.app/ (Vercel, auto-deploys from `main`)
**Repo:** https://github.com/alfie305/lead-capture

## What This Is
A lead capture page for The Keyes Company. Visitors enter their email and timeline to receive a free Coral Gables Buyer's Guide PDF delivered automatically via Klaviyo.

---

## Status (as of 2026-04-27)

**Working:**
- Email capture → Google Sheets + Klaviyo subscription → DOI confirmation → Klaviyo flow sends PDF
- Confirmation modal reminding users to confirm DOI email + check Promotions/Spam (commit `3a7d073`)

**In progress:**
- Klaviyo DOI confirmation email + flow email being switched to **text-only** in Klaviyo UI to escape Gmail's Promotions tab
- Customizing the DOI confirmation email copy (in Klaviyo, not code)

**Planned but blocked:**
- **SMS backup delivery** — phone-number capture in modal so users can opt into a text with the same PDF link, attached to the same Klaviyo profile.
  - Blocker: **10DLC brand registration rejected by TCR** for `www.keyes.com` ("not a full site / placeholder content").
  - Pending decision: domain strategy — submit `alfredomorejon.keyes.com` directly, buy personal domain + 301 redirect, or buy team brand domain + build mini-site.
  - Implementation plan saved at `~/.claude/plans/klaviyo-allows-for-text-unified-journal.md`.
- **WhatsApp via Klaviyo** — being evaluated as an alternative to SMS. Avoids 10DLC entirely; strong fit for the Coral Gables Latin American buyer demographic.

---

## How It Works (End-to-End)

```
Visitor fills form → clicks "Send Me the Guide"
  ↓
1. Browser → Google Apps Script (logs lead to Google Sheets)
2. Browser → Klaviyo Client API (subscribes to list TqcAat)
  ↓
Confirmation modal appears (reminds to confirm DOI email + check spam folders)
  ↓
User clicks DOI email confirm link → joins list TqcAat
  ↓
Klaviyo Flow fires → sends PDF guide email to visitor
```

---

## Integrations

### Google Sheets (Lead Logging)
- **Script URL:** `https://script.google.com/macros/s/AKfycbzBqItmjE2AKxnpy8jdf7WARyCrXLkxzNEnsa6711dIyzxoKNXUAISiYp1zVvOSgDv0jA/exec`
- **Function:** `doPost` receives email, timeline, source and appends a row
- **Columns:** Timestamp | Email | Timeline | Source
- **Redeploy:** Apps Script → Deploy → Manage deployments → New version → Deploy

### Klaviyo
- **Public API Key (company_id):** `VHFw97`
- **Private API Key:** stored in Apps Script only (not in browser code)
- **List ID:** `TqcAat` (Coral Gables Guide Requests)
- **Flow:** "Coral Gables Guide Delivery" — triggered when profile joins list TqcAat
- **Email subject:** "Your Coral Gables Buyer's Guide is here"
- **PDF link:** https://drive.google.com/uc?export=download&id=1psju1WZFdXda2A75NStpydj1Zj18xNFR
- **Sender:** The Keyes Company / alfredomorejon@keyes.com

---

## Changes Made

### 1. Get the Guide button — scroll fix
- Button now smoothly scrolls to the email form with proper offset
- Email input is auto-focused after scroll so user can start typing immediately
- File: `index.html` (lines ~1637–1648)

### 2. Klaviyo Client API — direct browser integration
- "Send Me the Guide" button now calls Klaviyo's Client API directly from the browser
- No private key exposed — uses public key `VHFw97` via `?company_id=` query param
- Subscribes profile to list `TqcAat` which triggers the Klaviyo Flow
- File: `index.html` (lines ~1570–1610)

### 3. Google Apps Script — simplified to Sheets only
- Removed Klaviyo API calls from Apps Script (browser handles Klaviyo now)
- Apps Script now only logs the lead row to Google Sheets
- Must redeploy as new version after any changes

### 4. Confirmation modal (commit `3a7d073`)
- After form submit, a styled modal pops up with:
  - 3-step instructions to confirm the Klaviyo DOI email
  - "Don't see it?" callout with chips for Promotions / Spam / Updates / Junk folders
  - "Got it — I'll check now" CTA
- Closes via X, CTA, backdrop click, or Esc
- File: `index.html` (HTML at `~1494`, CSS at `~937`, JS at `~1622`)

---

## Apps Script (Current Code)

```javascript
const KLAVIYO_API_KEY = 'pk_238fe284e1913251b940919d0bae7c3e84';
const KLAVIYO_LIST_ID = 'TqcAat';

function doPost(e) {
  const email    = e.parameter.email    || '';
  const timeline = e.parameter.timeline || '';
  const source   = e.parameter.source   || '';

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  sheet.appendRow([new Date(), email, timeline, source]);

  return ContentService.createTextOutput('ok');
}

function doGet(e) {
  return ContentService.createTextOutput('ok');
}
```

---

## Testing
1. Remove a profile from Klaviyo list TqcAat AND delete the profile entirely
2. Submit the form on the live page with that email
3. Check Klaviyo → Lists → TqcAat — profile should appear within seconds
4. Check inbox — PDF email should arrive from alfredomorejon@keyes.com
