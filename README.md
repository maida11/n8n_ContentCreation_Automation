# TikTok Script Generator — n8n Automation

Turn any product page URL into a ready-to-shoot TikTok script, complete with hashtags, hooks, and a voiceover audio file — all in one click.

---

## What it does

Paste a product or store URL into the web interface and the automation:

1. **Scrapes the product page** — fetches the page content via HTTP request
2. **Generates a structured script** — sends the scraped content to an LLM (Groq) to write a multi-shot TikTok script with labelled sections (Hook, Problem, Solution, Benefits, Social Proof, Call to Action)
3. **Generates hashtags** — produces 10 relevant hashtags for the niche
4. **Generates hooks** — writes 5 alternative opening hooks to test
5. **Synthesizes a voiceover** — sends the script to a TTS model and returns an audio file URL
6. **Returns everything to the UI** — the frontend displays the script, hashtags, hooks, and an audio player in a clean tabbed interface

Total turnaround: ~30–50 seconds per URL.

---

## Stack

| Layer | Tool |
|---|---|
| Automation | n8n (self-hosted) |
| LLM | Groq (via API) |
| Text-to-speech | Qwen TTS (Hugging Face Space) |
| Frontend | Vanilla HTML/CSS/JS |
| Scraping | n8n HTTP Request node |

---

## n8n Workflow nodes

```
Webhook → HTTP Request (scrape) → API call → Code (parse) →
Basic LLM Chain (script) → Basic LLM Chain (hashtags/hooks) →
Code (format) → HTTP Request (TTS) → HTTP Request (TTS poll) →
Code (assemble) → Merge → Respond to Webhook
```

---

## Setup

### 1. Import the workflow
Import the workflow JSON into your n8n instance and activate it.

### 2. Configure credentials
- Add your **Groq API key** to the Groq Chat Model nodes
- The TTS node uses the public Hugging Face Gradio API — no key needed

### 3. Serve the frontend
Open a terminal in the folder containing `script-generator.html` and run:

```bash
npx serve .
```

Then open `http://localhost:3000/script-generator.html` in your browser.

> Do not open the HTML file directly from the filesystem (`file://`). Browsers block fetch requests from `file://` origins.

### 4. Set the webhook URL
In `script-generator.html`, update this line with your n8n production webhook URL:

```javascript
const WEBHOOK_URL = 'http://localhost:5678/webhook/YOUR-WEBHOOK-ID';
```

### 5. Enable CORS
n8n blocks cross-origin requests by default. Start n8n with:

```bash
N8N_CORS_ALLOWED_ORIGINS=* n8n start
```

Or add to your `.env` file:

```
N8N_CORS_ALLOWED_ORIGINS=*
WEBHOOK_URL=http://localhost:5678/
```

---

## Output format

The webhook returns a JSON array with two objects:

```json
[
  {
    "audioUrl": "https://..."
  },
  {
    "script": [
      { "shot": 1, "label": "Hook", "text": "..." },
      { "shot": 2, "label": "Problem", "text": "..." }
    ],
    "hashtags": ["#SkinCare", "#GlowingSkin"],
    "hooks": ["Get ready for glowing skin", "Tired of clogged pores"]
  }
]
```

---

## Frontend features

- **Script tab** — numbered shots with labelled sections, ready to read on camera
- **Hashtags tab** — click any hashtag to copy it individually
- **Hooks tab** — click any hook card to copy it
- **Copy all** — copies the full script, hashtags, and hooks as plain text
- **Voiceover player** — inline audio player for the generated TTS file

---

## Notes

- Works best with product collection pages or individual product pages that have descriptive copy
- If a page blocks scraping, the LLM will receive limited content and the script quality may suffer
- The TTS audio URL is temporary (hosted on Hugging Face's Gradio file server) — download it if you need to keep it
- To use in production beyond localhost, expose n8n with a tunnel (ngrok, Cloudflare Tunnel) and update the webhook URL in the frontend
