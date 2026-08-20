# Urology AI Survey — Setup & Launch Guide

Same architecture as the Block 40 call sign-up (GitHub Pages static form → Power Automate HTTP trigger → SharePoint Excel). This guide covers only what's specific to this survey; the Block 40 reference doc has the full pattern.

## Files in this folder

| File | Purpose |
|---|---|
| `index.html` | The survey. Self-contained; no dependencies. |
| `sample_payload.json` | Paste into Power Automate "Use sample payload to generate schema". |
| `Urology_AI_Survey_Responses.xlsx` | Upload to SharePoint. Named table `Responses`, 93 columns, 130% zoom. |
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

## Production state (as of 8/20/2026)

- **Live URL:** https://umdunn.github.io/Urology-AI-Survey/ (repo `umdunn/Urology-AI-Survey`)
- **Responses land in:** `Urology_AI_Survey_Responses.xlsx` in Rod's Michigan Medicine
  OneDrive (umhealth-my.sharepoint.com, My files root), table `Responses`
- **Flow:** "AI Survey Response Intake" in Power Automate (Michigan Medicine tenant),
  HTTP trigger (Anyone) → concurrency 1 → Excel "Add a row into a table" with all 93
  columns mapped → email notification to rldunn@med.umich.edu per submission
- The 93 field mappings were injected by exporting the flow package, editing
  `definition.json` (`"item/<col>": "@triggerBody()?['<col>']"`), and re-importing
  with Update — the designer's code view is read-only on this tenant, and Office
  Scripts wouldn't load
- `AI-Survey-Response-Intake_MAPPED.zip` in this folder is the imported package (keep
  as backup; re-import it if the flow is ever damaged)

## If the survey needs edits after launch

1. Edit `index.html`, keeping `WEBHOOK_URL` and `DRAFT_MODE = false` intact
2. If columns change: regenerate `sample_payload.json` + the xlsx template, replace the
   OneDrive file, update the trigger schema, and redo the export/edit/import cycle
3. Upload changed files to the repo via GitHub web UI; Pages redeploys in ~1 min

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
- Payload always includes all 93 columns (empty strings) — Power Automate schema
  validation rejects submissions with missing keys (Block 40 lesson #1)
