# AutoRedTeam

**Attack your chatbot before someone else does.** Automated, canary-safe adversarial testing for AI assistants — with a live 3D hero, a scored findings report, and a mutation fuzzer.

AutoRedTeam runs a battery of prompt-injection, instruction-override and jailbreak attacks against a chatbot **you configure and control**, then shows you — line by line — where the guardrails held and where they broke, with a robustness grade and reproduction steps.

---

## Defensive by design — "canary-safe"

Real red-teaming can mean coaxing a model into genuinely harmful output. **AutoRedTeam deliberately does not do that.** It swaps the harmful goal for a **harmless planted secret** (a "canary" word) and a forbidden marker. The attacks only ever try to make the bot reveal the canary or trip a rule — so you measure whether a guardrail *holds* without ever producing dangerous content. Test only bots you own.

---

## Features

| Feature | What it does |
|---|---|
| **Attack battery** | 8 vectors — injection, override, role-play jailbreak, system-prompt leak, encoding/translation smuggling — fired at your target. |
| **Scored report** | Robustness score /100, an A–F grade, held/bypassed counts, and the worst finding. |
| **Watch it fail** | Expand any finding to see the exact attack sent and the bot's actual response (canary redacted). |
| **Mutation fuzzer** | Auto-generates reworded variants of one attack to hunt for a phrasing that slips through. |
| **Patch & re-scan** | Edit the system prompt, re-run, and see the robustness delta vs the last scan. |
| **Attack library** | A field guide to every technique: how it works, an example, and the defense. |
| **Attack builder** | Write your own canary-safe attack, fire it, and save it to your battery (stored locally). |
| **Export report** | Print/save the findings report. |
| **Live 3D hero** | A real-time WebGL scene (Three.js): attacks strike the guardrail, most deflect, one breaches. |

---

## How it works

- The **target** is a Gemini model running **your system prompt** (via `system_instruction`). The attack is sent as the user message.
- The serverless function `/api/attack` runs the target and returns its response. The client **judges deterministically** — did the response contain the canary or the forbidden marker? — so results are reliable and free.
- The **mutation fuzzer** uses the same endpoint in `mutate` mode to generate safe variants.
- The AI key stays **server-side** as `GEMINI_API_KEY`.

---

## Pages

```
autoredteam/
├── index.html      # Home — live 3D hero
├── scan.html       # Configure target · run battery · report · fuzzer · patch & re-scan
├── attacks.html    # Attack library (field guide)
├── builder.html    # Custom attack builder
├── app.js          # Shared: targeting cursor, attack battery, backend helpers
├── style.css       # Shared design system
├── api/
│   └── attack.js   # Serverless → Gemini (target run + mutations)
└── README.md
```

---

## Deploying to Vercel (free)

1. Push this folder to a GitHub repo.
2. Vercel: **Add New → Project → Import** → Framework **Other**, build settings empty.
3. Get a free Gemini key at **[Google AI Studio](https://aistudio.google.com/app/apikey)**.
4. Vercel → **Settings → Environment Variables** → add `GEMINI_API_KEY` (Production + Preview).
5. Deploy. Vercel auto-detects `api/attack.js`.

The scan/fuzzer/builder need the deployed backend. In a static/local preview, use **“See a sample report”** on the Scan page, and the 3D hero + attack library work without any key.

---

## Honest notes

- Robustness grades are a **signal, not a guarantee** — a pass means these specific probes held, not that the bot is unbreakable.
- Judging is deterministic on the canary/marker; subtle partial leaks may need a human eye (the full response is shown).
- The 3D hero is real-time **stylized** WebGL, not pre-rendered film art.
- Free Gemini is rate-limited; a full scan is several calls, so you may hit limits — wait a minute and retry.

---

*Adversarial testing for AI. Canary-safe. For bots you control.*
