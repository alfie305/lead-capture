# Coral Gables Buyer's Guide — Lead Capture Page

**Live URL:** https://lead-capture-amber.vercel.app/ (Vercel, auto-deploys from `main`)
**Repo:** https://github.com/alfie305/lead-capture

## What This Is
A lead capture page for The Keyes Company. Visitors enter their email and timeline to receive a free Coral Gables Buyer's Guide PDF, delivered directly via Microsoft Outlook through n8n automation.

---

## Status (as of 2026-04-28)

**Working:**
- Form UI, confirm modal, Vercel deployment

**In progress — needs webhook URL to go live:**
- n8n workflow is built and saved (`n8n-workflow.json`)
- `index.html` submit handler now POSTs to n8n — placeholder `REPLACE_WITH_N8N_WEBHOOK_URL` needs the real URL
- n8n Outlook and Google Sheets credentials need to be connected in the n8n UI
- Once activated and URL is swapped, push to `main` and it's live

**Blocked / deferred:**
- **SMS backup delivery** — 10DLC brand registration rejected by TCR for `www.keyes.com` (DataDome bot protection blocks TCR's crawler). Domain strategy TBD.
- **WhatsApp via Klaviyo** — Meta's link validator also blocked by DataDome. Evaluated as alternative to SMS.
- **Klaviyo** — bypassed entirely. DOI profiles weren't appearing; Outlook deliverability issues on shared Klaviyo IPs. n8n + direct Outlook is the current path.

---

## How It Works (Current Architecture)

```
Visitor fills form → clicks "Send Me the Guide"
  ↓
Browser → n8n webhook (JSON POST: {email, timeline, source})
  ↓
n8n:
  1. Log row to Google Sheets (Timestamp, Email, Timeline, Source)
  2. Send guide email via Microsoft Outlook to visitor
  3. IF timeline == "Within 30 days":
       → Hot lead alert to alfredomorejon@keyes.com
     ELSE:
       → Regular lead notify to alfredomorejon@keyes.com
  4. Respond 200 OK
  ↓
Confirm modal: "Check your inbox" (no DOI step — email arrives immediately)
```

---

## Integrations

### n8n (automation + email delivery)
- **Instance:** `https://alfie85.app.n8n.cloud`
- **Workflow file:** `n8n-workflow.json` (import into n8n if rebuilding)
- **Webhook path:** `coral-gables-guide`
- **Production webhook URL:** `https://alfie85.app.n8n.cloud/webhook/coral-gables-guide` *(activate workflow to enable)*

### Microsoft Outlook (guide + alert emails)
- Sender: `alfredomorejon@keyes.com`
- Guide email: plain text, PDF link `https://drive.google.com/uc?export=download&id=1psju1WZFdXda2A75NStpydj1Zj18xNFR`
- Hot lead subject: `🔥 HOT LEAD — {email}` (timeline: Within 30 days)
- Regular lead subject: `New Lead — {timeline}: {email}`

### Google Sheets (lead logging — via n8n)
- Logging done inside n8n workflow, not from the browser
- Columns: Timestamp | Email | Timeline | Source

### Klaviyo (bypassed)
- Company ID `VHFw97`, List `TqcAat` — still configured but not called
- May revisit for SMS/WhatsApp once domain/10DLC issues are resolved

---

## Changes Made

### 1. Get the Guide button — scroll fix
- Smoothly scrolls to email form with proper offset; auto-focuses input
- File: `index.html`

### 2. Klaviyo Client API — direct browser integration *(now replaced)*
- Was: browser POSTed to Klaviyo directly using public key `VHFw97`
- Now replaced by n8n webhook call

### 3. Google Apps Script — simplified to Sheets only *(now replaced)*
- Was: Apps Script logged to Sheets; Klaviyo call moved to browser
- Now: Sheets logging handled inside n8n; Apps Script no longer called

### 4. Confirmation modal (commit `3a7d073`)
- After submit, modal appears with inbox instructions + spam folder callout
- Updated copy: "On its way / Check your inbox" (no DOI confirmation step)
- File: `index.html` (HTML ~1636, CSS ~941, JS ~1812)

### 5. n8n pivot (2026-04-28)
- Replaced Klaviyo + Apps Script with single n8n webhook call
- n8n handles Sheets logging + Outlook guide email + lead alerts
- Workflow saved as `n8n-workflow.json`
- `.mcp.json` added for Claude Code MCP access to n8n instance

---

## To Finish Going Live

1. In n8n: connect Microsoft Outlook OAuth2 credential to all three Outlook nodes
2. In n8n: connect Google Sheets OAuth2 credential to the Sheets node
3. In n8n: activate the workflow
4. Copy the production webhook URL from the Form Webhook node
5. In `index.html`: replace `REPLACE_WITH_N8N_WEBHOOK_URL` with that URL
6. Push to `main` — Vercel auto-deploys
7. Test: submit form → check Sheets row + guide email in inbox
