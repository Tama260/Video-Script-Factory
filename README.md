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
[Pipedream Webhook Trigger]
↓
[generate_video_script] — Groq `openai/gpt-oss-120b` writes the scene-by-scene script + voiceover text from the submitted topic
↓
[generate_storyboard] — Pollinations.ai renders an AI image per scene
↓
[build_production_page] — assembles the script, storyboard, and platform-optimized copy into a standalone HTML page
↓
[publish_to_github_pages] — commits the page to `docs/` and returns its public URL
↓
[Production Archive — GitHub Pages] — every published package is listed and browsable

---

## Recent Fixes

- **Topic mismatch (generated output used a template topic instead of what was typed)** — the generator form used to ship with `The Future of AI Agents in 2026` pre-filled directly into the topic textarea's value (not just a placeholder). That's fixed: the field is now genuinely empty with only placeholder text, so there's no default value that could accidentally get submitted. The front end already forwards whatever you type as the `topic` query param correctly — if a generated page still shows a different topic than what you entered, the mismatch is happening **inside the Pipedream `generate_video_script` step**, most likely a hardcoded example topic left in the Groq prompt (e.g. a few-shot example that isn't being overridden by the actual `{{topic}}` variable). Worth opening that step and confirming the prompt interpolates the incoming topic parameter rather than a fixed example.
- **Storyboard images not loading** — the broken/blank image icons appear on the *generated production pages* (the per-video pages published to `docs/`, like the "AI Storyboard" scene cards), not on this generator front end. That template lives inside the Pipedream `build_production_page` step and wasn't part of the files shared for this round of fixes, so it couldn't be patched directly here. The same pattern already applied to the VICE-AI and Personal BOS dashboards should be dropped into that step's `<img>` markup:

```html
<div class="scene-image-wrap">
  <div class="scene-image-skeleton" id="skeleton_UNIQUE_ID"></div>
  <img id="UNIQUE_ID" class="scene-image" style="display:none"
       src="POLLINATIONS_IMAGE_URL" alt="Scene description" loading="lazy" referrerpolicy="no-referrer"
       onload="document.getElementById('skeleton_UNIQUE_ID').style.display='none'; this.style.display='block';"
       onerror="document.getElementById('skeleton_UNIQUE_ID').outerHTML='&lt;div class=&quot;scene-image-fallback&quot;&gt;Image unavailable&lt;/div&gt;'; this.remove();">
</div>
```
  Send over the `build_production_page` template (or the Pipedream step code) next time and it can be wired in directly rather than just documented here.
- **Model update** — Groq deprecated `llama-3.3-70b-versatile` on the free/developer tier. Script and storyboard-prompt generation now reference **`openai/gpt-oss-120b`** — Groq's officially recommended free-tier replacement, and it benchmarks above the old Llama 3.3 70B on reasoning and writing quality while running faster on Groq's hardware. Badges and footer on the generator page are updated to match; the actual model string still needs to be updated inside the `generate_video_script` Pipedream step (see Environment Variables below).

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

GROQ_API_KEY   = gsk_xxxx
GROQ_MODEL     = openai/gpt-oss-120b
GITHUB_TOKEN   = ghp_xxxx
GITHUB_REPO    = Tama260/Video-Script-Factory
TELEGRAM_BOT_TOKEN = xxxx:xxxx
TELEGRAM_CHAT_ID   = -100xxxxxxx

---

## Known maintenance items to check periodically

- **Groq model deprecations**: check `console.groq.com/docs/deprecations` every few months and re-point `GROQ_MODEL` if `openai/gpt-oss-120b` is ever retired.
- **Topic interpolation**: if generated pages ever again show a topic you didn't type, check the prompt inside `generate_video_script` first — that's the most likely place a stale/example topic is getting baked in.
- **Storyboard image fallback**: the `build_production_page` template doesn't yet have the skeleton/fallback image pattern used elsewhere in this project suite — share that template file to get it patched.

---

## Author

**Daffa Novendra Aditama**
AI Automation Engineer | Banten, Indonesia

[![LinkedIn](https://img.shields.io/badge/LinkedIn-daffanovendraaditama-blue)](https://linkedin.com/in/daffanovendraaditama)
[![GitHub](https://img.shields.io/badge/GitHub-Tama260-black)](https://github.com/Tama260)

---

*Built with Groq AI (openai/gpt-oss-120b) · Pipedream · Pollinations.ai · GitHub Pages · Netlify*
