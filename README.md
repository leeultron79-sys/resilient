# ResiliLink v3 — secure AI MVP (Personal Mode)

**Status: source code committed; not deployed or connected to paid API.** The `main` branch remains untouched. The default Personal Mode uses a real OpenAI API through a Cloudflare Worker. Community Mode remains an **unverified simulation**, not a real outreach or resource matching system.

## Architecture

GitHub Pages (public `index.html`) → HTTPS Cloudflare Worker (`src/worker.js`) → OpenAI Responses API.

**Never** insert `OPENAI_API_KEY` or the test access code in GitHub, frontend JavaScript, Pages environment variables exposed to the browser, issue comments, or commit history. The frontend only contains the public Worker URL. This is a proof-of-concept and is not a production system for sensitive personal information.

## Deployment steps

1. Create an OpenAI Platform project and fund it with a small prepaid balance; choose a project budget/usage alerts in the Platform dashboard, review model access. This is a separate service from ChatGPT subscriptions.
2. Sign into Cloudflare and install Wrangler (Node.js required). In a local checkout of **this branch**, run `npx wrangler login`. Inspect `wrangler.toml`, including the rate-limit binding. Confirm your Pages origin matches `ALLOWED_ORIGINS`. Only list origins you control; never use `*`.
3. **Set private secrets from your local terminal** (the values will be prompted, not committed):
   ```bash
   npx wrangler secret put OPENAI_API_KEY
   npx wrangler secret put MVP_ACCESS_CODE
   ```
   The access code should be long and unique, distributed privately to testers. This is a lightweight shared-secret barrier, **not user authentication**. Avoid broadly sharing the public Worker URL/code. If an attacker learns the code, they can consume credits, subject to the limits.
4. Run `npx wrangler deploy`. Copy the HTTPS Worker URL returned by Cloudflare and replace only `https://REPLACE_ME.workers.dev` in `config.js` **on this development branch**. Commit that public URL. The API key/code must never be in config.js.
5. Open the GitHub Pages site for the **development branch** (GitHub Pages might still publish `main`; don't change the existing production Pages source without deliberate approval). For isolated testing, serve the branch locally with `python -m http.server 8000`, and add `http://localhost:8000` to `ALLOWED_ORIGINS`. Or deploy a separate preview frontend from the branch.
6. Enter the test access code in the Personal Mode password field, submit *fictional* user input. The backend must reply with an AI answer and at most three suggested actions. Test missing code (401), oversized input (400), repeated requests (429), wrong origin (403), and expired API credit behavior (safe generic error).

## Security, limitations and cost controls

- The API key and shared test access code only live in Cloudflare Worker secrets. The code is entered on the client during use and sent to the Worker in HTTPS request headers, **not embedded in source**. It is kept in React memory only, but visible to someone controlling that browser; use a dedicated private test code.
- CORS is an explicit browser-origin allowlist and **not** a replacement for authentication. The Worker enforces a shared test code, per-IP rate limiting (5/min), 800-character input limit, and a 500-token output ceiling. Abuse can still generate costs: set a low Platform project budget/usage alerts and monitor usage. Project budgets may not be hard spend caps; disable/rotate keys if required.
- Worker uses `store: false` on API requests. Cloudflare/OpenAI may process data according to their policies. Don't submit real client or sensitive personal data, especially financial, health, or identity details.
- The model is not connected to an actual service directory, eligibility service, booking tool, or live weather API. Outputs must be verified with official sources; it does not make medical diagnoses, guarantee eligibility, or perform actions.
- Worker errors intentionally do not expose OpenAI upstream payloads, tokens, or secrets. Logs should not include request bodies, passwords, or authorization headers.
- On public GitHub Pages, all frontend code is publicly visible. Only the backend has secrets.

## Further work

- Introduce authenticated users/short-lived tokens before opening the MVP beyond a small trusted test group.
- Replace Community Mode mocks with a validated consented workflow or hide it for public pilots.
- Add structured safety tests and verified resource inventory before real-world use.
