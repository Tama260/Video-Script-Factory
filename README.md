# 🎬 Video Script Factory

> AI-powered video production system that turns any topic into a complete, production-ready video package — script, AI storyboard, and platform-optimized content — in under 60 seconds, published automatically with zero manual work.

**Live App:** [tama260.github.io/Video-Script-Factory](https://tama260.github.io/Video-Script-Factory/)

---

## What This System Does

Type one topic, pick a format, and get a full video production package back:

- Scene-by-scene script with exact voiceover lines
- AI-generated storyboard image for every scene
- Platform-ready description, tags, hook, and CTA
- A standalone, shareable HTML page for the finished package
- Instant notification with a preview posted to Telegram
- Auto-published to the public Production Archive — sortable by date, with thumbnails, and one-click (or bulk) delete

---

## Supported Formats

| Format | Duration | Scenes |
|---|---|---|
| YouTube Short | 60 seconds | 6 |
| TikTok | 30–60 seconds | 5 |
| Instagram Reel | 30 seconds | 4 |
| LinkedIn Video | 2–3 minutes | 8 |
| YouTube Long Form | 8–10 minutes | 15 |

---

## System Architecture

```
User submits topic + format + audience + tone
        │
        ▼
Pipedream WF1 — Video Script Factory   (HTTP trigger)
  ├─ parse_request          → validates/normalizes the input
  ├─ generate_master_script → Groq (openai/gpt-oss-120b) writes the script as JSON
  ├─ generate_storyboard_images → Pollinations.ai renders one image per scene
  ├─ build_html_dashboard   → assembles everything into a standalone HTML page
  ├─ publish_to_github_pages → commits the page to docs/ in the repo
  └─ (Telegram notification with a preview + link)
        │
        ▼
GitHub Pages serves docs/ as the live site — both the generator
form (index.html) and every published production page live here.

Deleting a production goes through a separate, minimal workflow:

Generator page → Pipedream WF2 — Delete Production (its own HTTP trigger)
  └─ delete_production → verifies a shared secret, then removes the
     file from GitHub via the Contents API. The GitHub token never
     touches the browser.
```

Everything — the generator, the archive, and every generated production — is served from **one place**: the `docs/` folder of this repo via GitHub Pages. There's no separate front-end host to keep in sync; a commit to `docs/` (whether from you or from the pipeline) is live within seconds.

---

## Production Archive Features

- **Thumbnails** — each card shows the first storyboard image from that production
- **Sort by date** — Newest first / Oldest first, read from the real publish date embedded in each page
- **Delete** — a ✕ on each card, or check multiple cards and use **Delete Selected** to remove several at once
- **Resilient images** — storyboard images show a loading skeleton and fall back to a clear "Image unavailable" message if Pollinations.ai fails or hangs (capped at 15s so nothing loads forever)

---

## Tech Stack

| Category | Tool |
|---|---|
| Workflow automation | Pipedream |
| Script generation | Groq AI — `openai/gpt-oss-120b` |
| Image generation | Pollinations.ai |
| Hosting (generator + archive + productions) | GitHub Pages (`docs/`) |
| Notifications | Telegram Bot API |

**Cost: $0/month** — entirely free-tier infrastructure, no separate static host needed.

---

## Environment Variables

**WF1 — Video Script Factory**
```
GROQ_API_KEY        = gsk_xxxx
GROQ_MODEL          = openai/gpt-oss-120b
GITHUB_TOKEN        = ghp_xxxx        (needs write access to the repo)
GITHUB_REPO         = Tama260/Video-Script-Factory
TELEGRAM_BOT_TOKEN  = xxxx:xxxx
TELEGRAM_CHAT_ID    = -100xxxxxxx
```

**WF2 — Delete Production** (separate workflow)
```
GITHUB_TOKEN   = ghp_xxxx        (same token, needs delete permission)
DELETE_SECRET  = vsf-delete-x7k2m9   (must match DELETE_SECRET in index.html)
```

> A secret shipped in front-end JavaScript is visible to anyone who views page source — it stops accidental/casual deletes, it isn't real access control. Fine for a personal project; don't reuse the pattern anywhere that needs real authentication.

---

## Known maintenance items

- **Groq model deprecations** — Groq periodically retires free-tier models (this happened once already with `llama-3.3-70b-versatile`, which silently broke every generation with a 404). Check `console.groq.com/docs/deprecations` occasionally and update `GROQ_MODEL` if `openai/gpt-oss-120b` is ever retired.
- **`$.respond()` in WF2 must be `await`ed** — Pipedream returns a generic "Error in workflow" response if it isn't, even when the deletion itself succeeds.
- **Pollinations.ai reliability** — the free/unauthenticated endpoint occasionally hangs or fails under load; the 15s watchdog in `build_html_dashboard` handles this gracefully, but registering a free key at `auth.pollinations.ai` is worth considering if failures become frequent.
- **Keep `DELETE_SECRET` in sync** between WF2's env var and `index.html` — rotate both together.

---

## Author

**Daffa Novendra Aditama**
AI Automation Engineer | Banten, Indonesia

[![LinkedIn](https://img.shields.io/badge/LinkedIn-daffanovendraaditama-blue)](https://linkedin.com/in/daffanovendraaditama)
[![GitHub](https://img.shields.io/badge/GitHub-Tama260-black)](https://github.com/Tama260)

---

*Built with Groq AI (openai/gpt-oss-120b) · Pipedream · Pollinations.ai · GitHub Pages*
