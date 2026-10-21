# Security review prompt

Run this against **this repo only** before every significant deploy. Paste it
into your AI coding tool (with the project open) in a fresh chat.

> You are a senior application-security reviewer.
>
> Audit MY OWN app's code for 5 problems and help me fix them:
> 1. API keys or secrets in frontend code
> 2. Customer data readable without access rules
> 3. File uploads with no size or rate limits
> 4. Paid features not checked on the server
> 5. AI agent / LLM endpoints with no step, time, cost, or rate cap
>
> How to work:
> - Read the code and config files in this project only. Do not call or test any live system.
> - Never print a secret value. If you find one, show the file and line with the value masked as ••••.
> - First give me a REPORT. Change nothing yet.
>
> For each of the 5 problems, the report has:
> - Status: FOUND / NOT FOUND / NOT SURE (say what you could not see).
> - Where: file path and line numbers.
> - Why it matters: 1–2 simple sentences.
> - Fix: the smallest change that solves it, and which files it touches.
>
> After the report, ask me which fixes to apply. Then apply them one at a time,
> show the diff, and tell me how to test each. End with a checklist of anything
> I must do outside the code (rotate keys, dashboard limits, billing alerts).

## Pre-deploy checklist

- [ ] No secret in client code; `.env` is git-ignored; keys only in server env vars
- [ ] Any key ever exposed is rotated (old one deleted in the provider dashboard)
- [ ] Access rules ON for every DB table / bucket; a user can't read another user's rows
- [ ] Every public endpoint that spends your API key has a per-IP rate limit and tight CORS
- [ ] Uploads (if any) are size/type-limited on the server, not just the form
- [ ] Every paid/admin route checks session + plan on the server
- [ ] Every LLM call has a timeout and an output-token cap
- [ ] Usage limits + budget alerts set in the AI and cloud provider dashboards
