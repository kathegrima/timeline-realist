# ⏱ Timeline Realist

> Your timeline is wrong. Find out by how much.

**Timeline Realist** is a free tool for hardware PMs. Enter your project phases with estimated durations, add context about your team and constraints, and get an honest assessment from an AI trained to think like a veteran hardware PM — including the risks you forgot, the phases you skipped, and the schedule slip you didn't want to admit.

🌐 **Live:** [kathegrima.github.io/timeline-realist](https://kathegrima.github.io/timeline-realist)

🌍 **Available in:** Italian 🇮🇹 · English 🇬🇧

---

## Why this exists

Hardware is hard. Schedules slip. EVT spins fail. Suppliers ghost you at DVT. Compliance testing takes three times longer than planned.

This tool doesn't apply magic multipliers — it reasons from context, identifies project-specific risks, and tells you what a seasoned PM would say in a honest 1:1. The Hofstadter's Law quote at the top is not decorative.

> *"It always takes longer than you expect, even when you take into account Hofstadter's Law."*
> — Douglas Hofstadter, Gödel, Escher, Bach (1979)

---

## How it works

1. Enter your project phases and estimated durations
2. Add optional context (team size, supplier constraints, target market, etc.)
3. Click **"Give me the truth"**
4. The frontend sends the data to a **Cloudflare Worker**
5. The Worker calls **Google Gemini** with a hardware-PM-specific prompt
6. The analysis is returned in the selected language

```
Browser (GitHub Pages) → Cloudflare Worker → Google Gemini API
```

The API key is never exposed to the browser — it lives in Worker environment variables.

---

## Methodological note

Revised estimates are AI-generated based on common patterns in the hardware industry, **not certified statistical studies**. The EVT/DVT/PVT framework is real and widely used in consumer electronics and robotics. The Hofstadter's Law is real. Everything else is shared experience — use the output as a starting point for a conversation with your team, not as absolute truth.

---

## Stack

| Layer | Technology |
|---|---|
| Hosting | GitHub Pages |
| Frontend | Vanilla HTML + CSS + JS (zero dependencies) |
| Backend / Proxy | Cloudflare Workers (free tier) |
| AI | Google Gemini 2.5 Flash-Lite |
| Font | Space Mono (Google Fonts) |

---

## Repository structure

```
/
└── index.html    # Full frontend — single file, no build step
```

---

## Deploy

### 1. Fork and enable GitHub Pages
1. Fork this repository
2. Go to **Settings → Pages** → Source: `main` / `root`
3. The site is live at `https://yourusername.github.io/timeline-realist`

### 2. Create the Cloudflare Worker
1. Sign up at [cloudflare.com](https://cloudflare.com)
2. Go to **Workers & Pages → Create → Start with Hello World**
3. Name it `timeline-realist`
4. Click **Deploy**, then **Edit code**
5. Replace all content with the Worker code below
6. Click **Save and deploy**

### 3. Add the Gemini API key
1. Get a free key at [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. In your Worker: **Settings → Variables and Secrets → Add**
   - Name: `GEMINI_API_KEY`
   - Type: **Secret**
   - Value: your `AIza...` key
3. Click **Deploy**

### 4. Connect the Worker to the frontend
In `index.html`, find and replace:

```javascript
const WORKER_URL = 'https://timeline-realist.your-account.workers.dev/';
```

---

## Worker code

```javascript
export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') return cors(new Response(null, { status: 204 }));
    if (request.method !== 'POST') return cors(new Response('Method not allowed', { status: 405 }));

    try {
      const { userMessage, systemPrompt } = await request.json();
      if (!userMessage?.trim()) return cors(new Response(JSON.stringify({ error: 'No input.' }), { status: 400 }));

      const res = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-lite:generateContent?key=${env.GEMINI_API_KEY}`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ role: 'user', parts: [{ text: `${systemPrompt}\n\nProject phases:\n${userMessage}` }] }],
            generationConfig: { temperature: 0.8, maxOutputTokens: 2048, thinkingConfig: { thinkingBudget: 0 } }
          })
        }
      );

      const data = await res.json();
      const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
      if (!text) return cors(new Response(JSON.stringify({ error: JSON.stringify(data) }), { status: 200 }));
      return cors(new Response(JSON.stringify({ text }), { headers: { 'Content-Type': 'application/json' } }));

    } catch (e) {
      return cors(new Response(JSON.stringify({ error: e.message }), { status: 500 }));
    }
  }
};

function cors(response) {
  const r = new Response(response.body, response);
  r.headers.set('Access-Control-Allow-Origin', '*');
  r.headers.set('Access-Control-Allow-Methods', 'POST, OPTIONS');
  r.headers.set('Access-Control-Allow-Headers', 'Content-Type');
  return r;
}
```

---

## Free tier limits

| Service | Free limit |
|---|---|
| GitHub Pages | Unlimited |
| Cloudflare Workers | 100,000 req/day |
| Gemini 2.5 Flash-Lite | 1,000 req/day |

---

## Privacy

User input is processed by Google Gemini via API. On the free tier of Google AI Studio, data may be used for model improvement. Timeline Realist does not store any data. To opt out of data training, enable billing on Google AI Studio.

---

## EVT / DVT / PVT — quick reference

| Phase | What it is |
|---|---|
| **EVT** | Engineering Validation Test — verifies the design works to specification |
| **DVT** | Design Validation Test — validates reliability, compliance and manufacturing requirements |
| **PVT** | Production Validation Test — first factory run, validates the manufacturing process |
| **MP** | Mass Production — serial production. Always comes after PVT issues |

---

## License

MIT — free to use, modify and share.
