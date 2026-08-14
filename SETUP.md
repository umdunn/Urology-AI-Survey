# Urology AI Survey — Setup & Launch Guide

Same architecture as the Block 40 call sign-up (GitHub Pages static form → Power Automate HTTP trigger → SharePoint Excel). This guide covers only what's specific to this survey; the Block 40 reference doc has the full pattern.

## Files in this folder

| File | Purpose |
|---|---|
| `index.html` | The survey. Self-contained; no dependencies. |
| `sample_payload.json` | Paste into Power Automate "Use sample payload to generate schema". |
| `Urology_AI_Survey_Responses.xlsx` | Upload to SharePoint. Named table `Responses`, 92 columns, 130% zoom. |
| `SETUP.md` | This guide. |

## Draft → production switch

Two constants at the top of the `<script>` block in `index.html`:

```js
var DRAFT_MODE  = true;   // true = red banner, submissions NOT posted
var WEBHOOK_URL = "";     // Power Automate HTTP trigger URL
```

- **Team review (now):** `DRAFT_MODE = true`. Red banner shows, submit shows a
  confirmation but posts nothing (payload logged to browser console for QA).
- **Faculty launch:** set `DRAFT_MODE = false` and paste the webhook URL. Banner
  disappears automatically.

## Launch checklist

1. **SharePoint:** upload `Urology_AI_Survey_Responses.xlsx` to the same SharePoint
   library used for Block 40.
2. **Power Automate** (Premium license, same as Block 40):
   - Instant cloud flow → trigger "When an HTTP request is received"
   - Trigger ⋯ → Settings → **Concurrency Control ON, Degree of Parallelism = 1**
   - "Use sample payload to generate schema" → paste contents of `sample_payload.json`
   - Action: Excel Online (Business) → "Add a row into a table" → map all 92 fields
     (tedious but one-time; every payload key matches its column name exactly)
   - Action: "Send an email (V2)" → notify Rod on each submission.
     Suggested subject: `AI survey response received` (responses are anonymous — no name in subject)
   - Save; copy the HTTP URL into `WEBHOOK_URL`
3. **GitHub Pages:** push to the hosting repo (decision pending — new repo vs.
   subdirectory of `umdunn/Urology-Call`). Wait ~1 min for redeploy.
4. **End-to-end test:** submit once through the live page, verify the row lands in
   Excel and the email arrives, then delete the test row.
5. **Send faculty email** with the link. Reminder 2 days before the close date.

## Design decisions (from team feedback, Aug 2026)

- **10 questions, mostly checkboxes** (Todd: ≤8–10; everyone: minimize burden)
- **Dropped:** rank/years demographics, relative-to-peers Likert, payment question
  (Kristian: not actionable / miscalibrated / no enterprise-rate premise)
- **Split** "concerns" (ethical, Q6) from "barriers" (capability, Q7) — Kristian
- **Added:** comfort-with-AI-tasks (Q5, Kristian's list verbatim), disclosure
  contexts (Q8, covers Todd's policy/governance list), "what do you want from the
  retreat" (Q9)
- **Gating:** clinical detail + ambient AI only shown if clinically active;
  research detail only if research user (Kristian's screening idea)
- **"Other" options** on concerns and most check-alls (Chrouser)
- **Anonymous** — no name field, so no dedupe possible; acceptable for planning data
- Payload always includes all 92 columns (empty strings) — Power Automate schema
  validation rejects submissions with missing keys (Block 40 lesson #1)
