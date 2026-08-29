# 🎬 Video Script Factory

> AI-powered video production system that transforms any topic into a complete, production-ready video package — script, storyboard, voiceover, and platform-optimized content — in under 60 seconds.

**Generator App (create a new production):** [videoscript-factory.netlify.app](https://videoscript-factory.netlify.app/)
**Production Archive (published video packages):** [tama260.github.io/Video-Script-Factory](https://tama260.github.io/Video-Script-Factory/)

---

## What This System Does

Input one topic. Get a complete video production package in under 60 seconds:

- Full scene-by-scene script with voiceover text
- AI-generated storyboard images for each scene
- Platform-optimized descriptions and tags
- Hook, CTA, and viral elements checklist
- Published automatically to GitHub Pages with a public URL

---

## Supported Formats

| Format | Duration | Scenes |
|---|---|---|
| YouTube Short | 60 seconds | 6 scenes |
| TikTok | 30-60 seconds | 5 scenes |
| Instagram Reel | 30 seconds | 4 scenes |
| LinkedIn Video | 2-3 minutes | 8 scenes |
| YouTube Long Form | 8-10 minutes | 15 scenes |

---

## System Architecture

[Generator App — Netlify]
↓
[User submits topic, format, audience, tone] — forwarded as query params, nothing pre-filled or hardcoded
↓
[Pipedream Webhook Trigger] (WF1 - Video Script Factory)
↓
[parse_request] — validates/normalizes the incoming query params
↓
[generate_master_script] — Groq `openai/gpt-oss-120b` writes the scene-by-scene script, hook, tags, CTA as JSON
↓
[generate_storyboard_images] — Pollinations.ai renders an AI image per scene
↓
[build_html_dashboard] — assembles the script, storyboard, and platform-optimized copy into a standalone HTML page
↓
[publish_to_github_pages] — commits the page to `docs/` and returns its public URL
↓
[Production Archive — GitHub Pages] — every published package is listed, sortable by date, and deletable from the generator app

A separate, second Pipedream workflow (its own HTTP trigger) handles deletes — see **Delete workflow setup** below.

---

## Recent Fixes

- **Root cause of "workflow doesn't produce a new package"** — `generate_master_script` was calling Groq with `model: 'llama-3.3-70b-versatile'`, which Groq deprecated on the free tier. Every generation request failed with a `404` at that step, so nothing downstream (`generate_storyboard_images`, `build_html_dashboard`, `publish_to_github_pages`) ever ran — that's why the archive stayed frozen at the same productions for months even though the generator "succeeded" from the front end's point of view. Fixed by switching to **`openai/gpt-oss-120b`** and adding `response_format: { type: 'json_object' }` so Groq returns clean JSON instead of relying only on the prompt instruction (see `generate_master_script.mjs`).
- **Topic mismatch (generated output used a template topic instead of what was typed)** — turned out to be a symptom of the bug above, not a separate prompt issue: because generation was silently failing, what people saw was actually an old cached production from the archive, not a fresh result. The generator form also used to ship with `The Future of AI Agents in 2026` pre-filled directly into the topic textarea's value (not just a placeholder) — that's removed too, so the field is genuinely empty and only shows placeholder text.
- **`Cannot read properties of undefined (reading '$return_value')`** — happens when `generate_master_script` runs without `parse_request`'s output available (e.g. testing the step in isolation instead of triggering the full workflow). Added a guard clause that throws a clear, actionable error instead of a raw `TypeError` if this happens again.
- **Storyboard images not loading** — fixed directly in a real production page (`vid-mpfder6g-fixed.html`) using the skeleton-loading + graceful-fallback pattern already applied to the VICE-AI and Personal BOS dashboards. Apply the same `<img>` markup pattern inside the `build_html_dashboard` step so every *newly generated* production gets it automatically, not just this one file.
- **Model update** — Groq deprecated `llama-3.3-70b-versatile` on the free/developer tier. Script generation now uses **`openai/gpt-oss-120b`** — Groq's officially recommended free-tier replacement, and it benchmarks above the old Llama 3.3 70B on reasoning and writing quality while running faster on Groq's hardware. Badges and footer on the generator page are updated to match.
- **Sort by date + delete** — see the new features section right below.

---

## New: Sort by date & delete productions

The Production Archive on the generator page now:

- **Sorts by date** — a "Newest first / Oldest first" dropdown above the grid. Each production's real publish date is read from the `Production ID: VID-XXXX - <date>` line embedded in its own page (fetched once per page load), not guessed from the filename.
- **Deletes productions** — a small ✕ button on each card. Deleting a production calls a **separate** Pipedream workflow (`delete_production.mjs`) that removes the file from the GitHub repo server-side. The GitHub token never touches the browser.

### Delete workflow setup (one-time)

1. In Pipedream, create a **new** workflow (don't add this to WF1) with an **HTTP / Webhook trigger**.
2. Add one Node.js code step using `delete_production.mjs`.
3. Set env vars on that workflow: `GITHUB_TOKEN` (same token WF1 uses, needs write/delete access to the repo) and `DELETE_SECRET` (any random string you pick).
4. Deploy it, copy its trigger URL.
5. In `index.html`, set `DELETE_WEBHOOK_URL` to that URL and `DELETE_SECRET` to the same string from step 3.

> Heads up: a secret embedded in front-end JavaScript is visible to anyone who views page source — this stops accidental/casual deletes, it isn't real authentication. Fine for a personal portfolio project; don't reuse this pattern for anything that needs real access control.

---

## Tech Stack

| Category | Tools |
|---|---|
| Workflow Automation | Pipedream |
| Script & Prompt Generation | Groq AI (`openai/gpt-oss-120b`) |
| Image Generation | Pollinations.ai |
| Publishing | GitHub Pages (`docs/` folder) |
| Generator Front End | Netlify (static HTML/JS) |

**Cost: $0/month** — 100% free tier infrastructure

---

## Environment Variables

### WF1 — Video Script Factory

GROQ_API_KEY   = gsk_xxxx
GROQ_MODEL     = openai/gpt-oss-120b
GITHUB_TOKEN   = ghp_xxxx
GITHUB_REPO    = Tama260/Video-Script-Factory
TELEGRAM_BOT_TOKEN = xxxx:xxxx
TELEGRAM_CHAT_ID   = -100xxxxxxx

### WF2 — Delete Production (new)

GITHUB_TOKEN   = ghp_xxxx   (same token as above, needs delete permission)
DELETE_SECRET  = any-random-string-you-pick

---

## Known maintenance items to check periodically

- **Groq model deprecations**: check `console.groq.com/docs/deprecations` every few months and re-point `GROQ_MODEL` if `openai/gpt-oss-120b` is ever retired. When this happens, `generate_master_script` fails with a 404 and the whole pipeline silently stops producing new pages — check this first if productions stop appearing again.
- **Topic interpolation**: if generated pages ever again show a topic you didn't type, first confirm generation actually succeeded (check WF1's Live Events) — the last time this happened, it was really the 404 issue above, not a prompt problem.
- **Storyboard image fallback**: apply the skeleton/fallback `<img>` pattern (see `vid-mpfder6g-fixed.html`) inside `build_html_dashboard` so it's baked into every new production automatically.
- **Delete workflow**: keep `DELETE_SECRET` in sync between WF2's env var and `index.html` — if you rotate one, rotate the other.

---

## Author

**Daffa Novendra Aditama**
AI Automation Engineer | Banten, Indonesia

[![LinkedIn](https://img.shields.io/badge/LinkedIn-daffanovendraaditama-blue)](https://linkedin.com/in/daffanovendraaditama)
[![GitHub](https://img.shields.io/badge/GitHub-Tama260-black)](https://github.com/Tama260)

---

*Built with Groq AI (openai/gpt-oss-120b) · Pipedream · Pollinations.ai · GitHub Pages · Netlify*
