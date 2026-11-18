# Security review — 20-point pen-test checklist (2026-11-19)

AutoRedTeam is a static frontend (vanilla JS, 3D hero via Three.js) plus one
Vercel serverless function (`api/attack.js`) that proxies Gemini, used to
canary-test a user-supplied chatbot system prompt against a fixed library of
prompt-injection attacks. No accounts, no database, no stored user data
(saved attacks live in the visitor's own `localStorage`). Reviewed
`api/attack.js`, `app.js`, and the `esc()`/innerHTML usage across
`scan.html`, `builder.html`, `attacks.html`, `index.html` in full.

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 1 | HTTP/2 Rapid Reset | N/A | Vercel serverless function; no self-managed HTTP server. |
| 2 | bcrypt 72-byte limit | N/A | No passwords/accounts. |
| 3 | SAML signature bypass | N/A | No SAML. |
| 4 | nOAuth | N/A | No OAuth. |
| 5 | GraphQL alias batching | N/A | No GraphQL. |
| 6 | TOTP replay | N/A | No TOTP/2FA. |
| 7 | ReDoS | No | The only regexes are small fixed patterns: `/^```(?:json)?/i`, `` /```$/ ``, `/\{[\s\S]*\}/` (server-side, bounded by the 400/700-token Gemini output cap before it's ever matched), and `/service|GEMINI|model|quota|rate|reach/i` / `/[&<>]/g` client-side — none have nested quantifiers. |
| 8 | JWT algorithm confusion | N/A | No JWTs. |
| 9 | HTTP request smuggling | N/A | Vercel's edge terminates HTTP. |
| 10 | Web cache poisoning | N/A | `/api/attack` sets no cache headers and returns per-request, non-cacheable results; nothing here would be cached by an intermediary in a way that matters. |
| 11 | Spoofed X-Forwarded-For | N/A | No IP-based logic anywhere in this app. |
| 12 | Postgres views bypassing RLS | N/A | No database. |
| 13 | `Math.random()` for security tokens | N/A | No tokens/ids generated — no auth, no session, nothing to secure with randomness. |
| 14 | Timing attacks on secret comparison | N/A | The "canary"/"marker" check (`scan.html`) is a plain substring search on a chatbot's own response text for deterministic grading, not a secret comparison — there's nothing confidential being compared, and the value being searched for is one the tool's own user chose. |
| 15 | Padding oracle | N/A | No encryption. |
| 16 | Server-side prototype pollution | No | `attack.js` reads a handful of named string/number fields off the parsed body (`mode`, `systemPrompt`, `userMessage`, `attackPrompt`, `n`) with explicit `.toString()`/`parseInt()` coercion — no object spread/merge of untrusted input. |
| 17 | Server-side template injection | N/A | No server-side templating; responses are JSON. Client-side HTML is built with manual string concatenation, but every dynamic value (attack prompts, bot responses, error messages) is passed through `esc()` (`app.js`), which escapes `&`, `<`, `>` before insertion into `innerHTML` — confirmed across every `innerHTML`/`insertAdjacentHTML` call in `scan.html`, `builder.html` and `attacks.html` that touches request- or response-derived text. |
| 18 | XXE | N/A | No XML parsing. |
| 19 | Cross-site WebSocket hijacking | N/A | No WebSocket endpoint. |
| 20 | Unicode username normalization | N/A | No usernames/login. |

## Findings outside the 20-point list

1. **Wide-open CORS + no rate limiting on a paid AI endpoint.** `api/attack.js`
   sets `Access-Control-Allow-Origin: *`, so any third-party website can call
   it directly from a visitor's browser, and there is no per-IP/per-visitor
   rate limit on top of it (contrast with `ilaaj-saathi`, `quickresume`,
   `praxis`'s own flagged gap, etc. elsewhere in this portfolio). Combined,
   another site could embed a call to your deployed `/api/attack` and run up
   calls against your `GEMINI_API_KEY` at will. Low severity today since the
   key is described as the free Gemini tier (worst case is hitting Google's
   own rate limit, which the code already handles gracefully), but becomes a
   real cost risk if a paid key is ever used. Suggested fix (not applied —
   pending your approval): restrict `Access-Allow-Origin` to your own deployed
   origin, and add a simple per-IP rate limit in front of the Gemini call.
2. **Self-hosted guardrail test, not a tool against third parties** — worth
   noting explicitly since this project's whole purpose is "attacking" a
   chatbot: the target is always a Gemini call using a system prompt the
   *visitor themselves* supplies in their own browser session, never a
   third-party service reached over the network. There's no SSRF-style
   "attack any URL" primitive here — `userMessage`/`systemPrompt` only ever
   become the `contents`/`system_instruction` of your own server's Gemini
   call, so this tool can't be pointed at an arbitrary external target.

## Summary

No exploitable injection/auth/crypto findings. One real gap worth closing:
`/api/attack` has permissive CORS and no rate limiting, which — on a paid
key — would let any third-party site run up usage against it. No fix has
been applied here — per your standing instructions, this is reported for
approval before any change is made.
