# CLAUDE.md — Coral Gables Buyer's Guide Lead Capture

Operational context for Claude when working in this project. Humans should read [README.md](README.md) instead.

---

## What this project is

A single-file static HTML lead capture page for **The Keyes Company** (real estate brokerage, Coral Gables FL). One agent: Alfredo Morejon. Visitors enter email + timeline, receive a Coral Gables Buyer's Guide PDF via Klaviyo email flow.

**Stack:** plain HTML/CSS/JS in [index.html](index.html). No build step. No framework. ~1700 lines, all in one file.

**Deployed:**
- Production: `https://lead-capture-amber.vercel.app/` (Vercel auto-deploys from `main`)
- Repo: `https://github.com/alfie305/lead-capture`
- Old/possibly-stale: `https://alfie305.github.io/lead-capture/` (referenced in README; verify before relying on it)

---

## Integrations

### Klaviyo (browser-side opt-in)
- **Company ID (public):** `VHFw97`
- **List ID:** `TqcAat` (Coral Gables Guide Requests) — double opt-in enabled
- **API:** `/client/subscriptions/` endpoint, revision `2024-10-15`, `custom_source: "Coral Gables Guide"`
- **Submit handler:** [index.html:1547](index.html) — POST to Klaviyo + Google Sheets, then opens confirm modal
- **Klaviyo flow:** "Coral Gables Guide Delivery" — triggered on list join, sends PDF link via email
- **PDF location:** Google Drive `https://drive.google.com/uc?export=download&id=1psju1WZFdXda2A75NStpydj1Zj18xNFR`

### Google Apps Script (Sheets logging)
- **Endpoint:** `https://script.google.com/macros/s/AKfycbzBqItmjE2AKxnpy8jdf7WARyCrXLkxzNEnsa6711dIyzxoKNXUAISiYp1zVvOSgDv0jA/exec`
- Append-only row (Timestamp, Email, Timeline, Source). No Klaviyo logic in Apps Script — that moved to the browser.

---

## Confirm modal (added commit `3a7d073`)

After form submit, a modal appears reminding the user to confirm DOI email and check Promotions/Spam folders.

- **HTML:** [index.html:1494](index.html) (`#confirm-modal`, `.cm-card`)
- **CSS:** [index.html:937](index.html) onwards — class prefix `.cm-*`, reuses existing color tokens (`--ivory`, `--clay`, `--green`, etc.)
- **JS:** [index.html:1622](index.html) onwards — opens on submit success or fail; closes on X / Got-it / backdrop / Esc

---

## Current state / blockers (as of 2026-04-27)

- **Email opt-in:** working end-to-end. Klaviyo returns 202; profile lands in `TqcAat` after DOI confirmation.
- **Klaviyo flow email → text-only:** in progress (in Klaviyo UI, not code) to escape Gmail Promotions tab. Switch is via Klaviyo email editor → three-dot menu → "Switch to text only editor."
- **Double opt-in customization:** the system DOI confirmation email lives in Klaviyo (Templates or list Consent settings, depending on account UI version). Edit there, not in code.
- **SMS backup channel:** planned but **blocked**. Implementation plan saved at `~/.claude/plans/klaviyo-allows-for-text-unified-journal.md`. Blocker: 10DLC brand registration was **rejected by TCR** for site `www.keyes.com` ("not a full site / placeholder content"). Domain strategy decision pending — options: submit `alfredomorejon.keyes.com` directly, buy a personal domain + 301 redirect, or buy team brand domain + build mini-site.
- **WhatsApp via Klaviyo:** being evaluated as alternative to SMS. Strong fit for Coral Gables Latin American buyer demographic. Avoids 10DLC entirely. No code written yet.

---

## Working with this codebase

**Do:**
- Edit [index.html](index.html) directly. Single file, no build step.
- Test by opening the file locally (`open index.html`) or via the Vercel preview after push.
- Commit small, push to `main` for auto-deploy.
- For UI work, the codebase uses semantic class prefixes (`.cm-*` for confirm modal, `.form-*` for the capture form, `.cm-sms-*` reserved for the deferred SMS section).

**Don't:**
- Add frameworks or build tooling — keep it static.
- Re-introduce private API keys in browser code. Klaviyo private key lives in Apps Script only.
- Edit Klaviyo flows/templates from code — those are managed in Klaviyo's UI and changes are out-of-band.
- Add features without explicit user approval. The user prefers small, reviewable changes.
- Touch the existing email submit path in `index.html` (lines ~1547–1620) when adding the SMS/WhatsApp section — additive only.

---

## Style / preferences

- Page typography: **Playfair Display** for serif headlines (italic for emphasis), **Source Sans 3** for body, **JetBrains Mono** for accent chips.
- Color tokens defined at [index.html:17-33](index.html#L17-L33). Always use CSS variables, not hex literals.
- Animation: subtle, cubic-bezier `(.2,.7,.2,1)` is the project standard. No bouncy or playful motion.
- Copy: warm but professional. No emojis in product copy. No exclamation marks in CTAs.

---

## Out of scope

- Multi-agent / multi-brokerage support
- Internationalization (US-only, English-only)
- Server-side rendering or any backend beyond Apps Script + Klaviyo
- CRM integration beyond the Klaviyo profile
