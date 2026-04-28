# CLAUDE.md — Coral Gables Buyer's Guide Lead Capture

Operational context for Claude when working in this project. Humans should read [README.md](README.md) instead.

---

## What this project is

A single-file static HTML lead capture page for **The Keyes Company** (real estate brokerage, Coral Gables FL). One agent: Alfredo Morejon. Visitors enter email + timeline, receive a Coral Gables Buyer's Guide PDF via email.

**Stack:** plain HTML/CSS/JS in [index.html](index.html). No build step. No framework. ~1700 lines, all in one file.

**Deployed:**
- Production: `https://lead-capture-amber.vercel.app/` (Vercel auto-deploys from `main`)
- Repo: `https://github.com/alfie305/lead-capture`

---

## Current architecture (as of 2026-04-28)

**Klaviyo is bypassed.** After repeated issues (DOI profiles not appearing, Outlook deliverability failures, DataDome blocking TCR/Meta crawlers preventing 10DLC and WhatsApp registration), we pivoted to **n8n + Microsoft Outlook** for direct email delivery.

### How form submission works now

```
Visitor fills form → clicks "Send Me the Guide"
  ↓
Browser → n8n webhook (single JSON POST: {email, timeline, source})
  ↓
n8n workflow:
  1. Log to Google Sheets
  2. Send guide email via Microsoft Outlook to visitor
  3. IF timeline == "Within 30 days" → hot lead alert email to Alfredo
     ELSE → regular lead notify email to Alfredo
  4. Respond 200 OK
  ↓
Confirm modal appears: "Check your inbox" (no DOI step)
```

### Submit handler

[index.html ~1727](index.html) — `N8N_WEBHOOK` constant, single `fetch` POST with JSON body.

**Status:** `N8N_WEBHOOK` is currently set to the placeholder `'REPLACE_WITH_N8N_WEBHOOK_URL'`. Replace with the live URL from n8n (Form Webhook node → copy production URL) then push to deploy.

---

## Integrations

### n8n (primary email delivery)
- **Instance:** `https://alfie85.app.n8n.cloud`
- **Workflow:** "Coral Gables Guide — Lead Capture" (saved as [n8n-workflow.json](n8n-workflow.json))
- **Webhook path:** `coral-gables-guide` — production URL will be `https://alfie85.app.n8n.cloud/webhook/coral-gables-guide`
- **Nodes:** Form Webhook → Log to Google Sheets → [Send Guide Email + Hot Lead? IF branch] → Respond OK
- **Outlook credentials:** need to be connected in n8n UI (Microsoft Outlook OAuth2)
- **Google Sheets credential:** need to be connected in n8n UI (Google Sheets OAuth2)
- **Sheet ID:** user has entered the real Sheet ID in the n8n Google Sheets node
- **MCP access:** `.mcp.json` in project root points at `https://alfie85.app.n8n.cloud/mcp-server/http` — takes effect after session restart

### Google Sheets (lead logging — via n8n, not Apps Script)
- Logging is now done inside the n8n workflow, not from the browser.
- Apps Script endpoint still exists but is no longer called by the page.
- Columns: Timestamp | Email | Timeline | Source

### Microsoft Outlook (email delivery — via n8n)
- Sends guide email to lead from `alfredomorejon@keyes.com`
- Plain text, includes PDF link: `https://drive.google.com/uc?export=download&id=1psju1WZFdXda2A75NStpydj1Zj18xNFR`
- Reply-to: `alfredomorejon@keyes.com`
- Hot lead alert goes to `alfredomorejon@keyes.com` with subject `🔥 HOT LEAD — {email}` when timeline is "Within 30 days"
- Regular lead notify goes to `alfredomorejon@keyes.com` with subject `New Lead — {timeline}: {email}`

### Klaviyo (bypassed — do not re-enable without discussion)
- **Company ID:** `VHFw97`
- **List ID:** `TqcAat` (Coral Gables Guide Requests) — double opt-in still configured
- **Why bypassed:** 202 responses but profiles weren't appearing in All Profiles (DOI strict mode); Outlook deliverability issues due to DataDome bot protection on keyes.com blocking shared Klaviyo IPs; 10DLC/WhatsApp setup blocked by same DataDome issue.
- Klaviyo code is fully removed from [index.html](index.html). Don't add it back without user approval.

### Google Apps Script (no longer used)
- Endpoint still live but page no longer calls it.
- `https://script.google.com/macros/s/AKfycbzBqItmjE2AKxnpy8jdf7WARyCrXLkxzNEnsa6711dIyzxoKNXUAISiYp1zVvOSgDv0jA/exec`

---

## Confirm modal

After form submit, a modal appears reminding the user to check their inbox. **No DOI step** — n8n sends immediately, no confirmation click required.

- **HTML:** [index.html:1636](index.html) (`#confirm-modal`, `.cm-card`)
- **CSS:** [index.html:941](index.html) onwards — class prefix `.cm-*`
- **JS:** [index.html ~1812](index.html) — opens on submit success or fail; closes on X / Got-it / backdrop / Esc
- **Current copy:** "On its way" / "Check your inbox." — from `alfredomorejon@keyes.com`, check Promotions/Spam/Updates/Junk

---

## Immediate next steps (resume here)

1. **Get n8n webhook URL** — in n8n, activate the workflow and copy the production webhook URL from the Form Webhook node.
2. **Replace placeholder** — in [index.html ~1732](index.html), replace `'REPLACE_WITH_N8N_WEBHOOK_URL'` with the real URL.
3. **Connect Outlook credential** — in n8n UI, open each Outlook node and connect Microsoft Outlook OAuth2 account.
4. **Connect Google Sheets credential** — in n8n UI, open the Google Sheets node and connect Google Sheets OAuth2 account.
5. **Activate workflow** — toggle it live in n8n.
6. **Test** — submit form on local or Vercel preview, check Google Sheets row appears and guide email arrives in inbox (not Promotions).
7. **Push to GitHub** — Vercel auto-deploys from `main`.

---

## Blocked / deferred

- **SMS backup delivery** — 10DLC brand registration rejected by TCR for `www.keyes.com`. DataDome bot protection blocks TCR's automated crawler. Implementation plan at `~/.claude/plans/klaviyo-allows-for-text-unified-journal.md`. Pending domain decision.
- **WhatsApp via Klaviyo** — blocked by same DataDome issue (Meta's link validator can't reach keyes.com). Evaluated as alternative to SMS for Coral Gables Latin American buyer demographic.
- **Domain strategy** — options: `alfredomorejon.keyes.com`, personal domain + 301 redirect, team brand domain. No decision yet.

---

## Working with this codebase

**Do:**
- Edit [index.html](index.html) directly. Single file, no build step.
- Test locally: `open index.html` or use Vercel preview after push.
- Commit small, push to `main` for auto-deploy.
- Class prefixes: `.cm-*` for confirm modal, `.form-*` for capture form, `.cm-sms-*` reserved for deferred SMS section.

**Don't:**
- Add frameworks or build tooling.
- Re-introduce Klaviyo or Apps Script calls without explicit approval.
- Add features without user approval — small reviewable changes only.
- Put private API keys or tokens in browser-side code.

---

## Style / preferences

- Typography: **Playfair Display** (serif headlines, italic for emphasis), **Source Sans 3** (body), **JetBrains Mono** (accent chips).
- Color tokens at [index.html:17-33](index.html#L17-L33). Always use CSS variables, never hex literals.
- Animation: subtle, `cubic-bezier(.2,.7,.2,1)`. No bouncy motion.
- Copy: warm but professional. No emojis in product copy. No exclamation marks in CTAs.

---

## Out of scope

- Multi-agent / multi-brokerage support
- Internationalization (US-only, English-only)
- Server-side rendering or any backend beyond n8n
- CRM integration
