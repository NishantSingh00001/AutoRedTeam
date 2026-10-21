# AutoRedTeam

## Security rules (always apply)

When writing or changing code in this repo, hold to these without being asked:

1. **No secrets in the client.** API keys, tokens, and service/admin keys live only
   in server-side env vars and are read only in server code (API routes, serverless
   functions). Never put a real secret in a `NEXT_PUBLIC_*`, `VITE_*`, `EXPO_PUBLIC_*`
   or `REACT_APP_*` variable, in client files, or committed to git. `.env` stays
   git-ignored; `.env.example` holds placeholders only.
2. **Guard every public endpoint that spends my API key.** Add a per-IP rate limit
   (return 429 when exceeded), cap request body/input length, and restrict CORS to
   this app's own origin — never `Access-Control-Allow-Origin: *` on an endpoint that
   calls a paid provider.
3. **Protect data at the row level.** Any DB table or storage bucket holding user data
   has access rules ON (Supabase RLS owner-only policies / real Firebase rules, never
   test mode). API routes check session + ownership and return only the fields the UI needs.
4. **Cap every LLM call.** Set an explicit timeout and an output-token limit; bound any
   retry/fallback loop. No unbounded agent loops.
5. **Server-side checks for anything paid or privileged.** Never trust a client value
   (`isPro`, a role in localStorage) for access; verify on the server.

Flag, don't silently skip, anything that would break one of these.
