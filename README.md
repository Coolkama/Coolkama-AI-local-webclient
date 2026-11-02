# Coolkama AI Local Web Client

A standalone, mobile-first LLM workbench and chat client built as a **single client-side HTML file**. It can be opened locally or served as a **static webpage**. It is **not part of** the separate PC-hosted Coolkama AI ecosystem.

> **Project requirements:** [Scope and guardrails](docs/PROJECT_SCOPE.md). Read these before making changes.

## What it does

- Multiple saved chats, streamed model responses, Markdown/code highlighting and code snippet downloads.
- Free-only remote model selection, conservative browser-side usage limits, and optional local Ollama or offline test mode.
- Opt-in, model-driven web search: tool-capable LLMs can call a browser-owned Tavily Basic `web_search` function, receive sources and refine queries in the same turn (up to 3 searches per user message). Each search and each model round trip consumes the corresponding free-tier allowance and local counter. Models that do not support tools may reject web-enabled requests; no paid fallback is used.
- User-selected PDF, image, text and code-file attachments.
- Browser-local persistence for chats, settings and API keys.

## File access rules

**Users choose files to upload/process. Users can download generated outputs. The app must not edit, overwrite, delete, move or otherwise modify files already on their device.** File input is through browser-controlled selection, and saving is through normal browser downloads—not direct filesystem writes.

Selected content may be sent to the configured external AI provider when the user submits a request. When web search is enabled, model-generated queries (which may include conversation details) are also sent to Tavily. Files and chats are not automatically uploaded to GitHub. Browser localStorage is origin-specific, not a synchronised account, and API keys stored there are not encrypted.

## Running

Open `index.html` in a modern browser, or publish it with a static website host. CDN dependencies require internet access unless separately cached. Configure a supported free-tier provider key (or use the offline test mode); direct remote API calls and LAN Ollama access depend on the browser's CORS/network permissions.

Client-side rate counters are **not** a billing guarantee: use genuine free provider plans and provider-side billing protections. No paid model/search fallback is intended.

## Repository policy

Source code and documentation only. GitHub Actions have been disabled by the owner; do not enable Actions or add workflows, CI, cache/artefact storage, Packages, Codespaces, Git LFS or other chargeable infrastructure without explicit approval.

## GitHub Pages

For static hosting, select **Settings → Pages → Deploy from a branch → main → /(root)**. The root-level `.nojekyll` file disables Jekyll processing; this repository needs no application build step. If the public site is not published yet, review the Pages publishing status and GitHub's deployment messages. The expected URL is https://coolkama.github.io/Coolkama-AI-local-webclient/ (only after successful publication).

## Diagnostics (local-only)

Open **Diagnostics** using the activity icon in the navigation bar or **Settings → Open logs**. Recording defaults to **Normal**; use **Detailed** to see search queries, tool arguments and web results. **Full Trace** is opt-in and persists complete API request/response payloads in the browser's **IndexedDB**, including conversation text, uploaded document extracts, images, the Tavily response and raw SSE messages. The trace records model IDs (when reported by the provider), finish reasons, token usage (when provided), timings, errors and each inference/tool round.

**API keys and authentication values are not recorded.** Best-effort credential redaction is applied to all diagnostic levels; do not assume exports are free of other private data. Full traces can be large and use device storage. Log records are bounded to the latest 400 events, with a 650 KB compact metadata index in localStorage; IndexedDB holds large Full Trace payloads. Browser storage quota and eviction rules still apply.

**Save chat log** downloads records for the current conversation; **Save all logs** includes unrelated events such as model discovery. **Clear** removes both the compact index and full-trace database records. Logs are never automatically sent to GitHub or another service. Inspect JSON downloads before sharing them.

If diagnosing a search failure, enable Detailed or Full Trace, reproduce the question and save that conversation's log. Existing conversations are unaffected by changing diagnostic level; earlier events cannot retroactively acquire payloads.

## Research reliability and citation checks

- **Current date on every inference round:** the browser supplies the current local date/time, timezone and UTC time as a system message. This still works when `openrouter/free` intentionally selects a *different* free model for each request; model rotation is preserved.
- **Better search guidance:** the model is asked to search broadly before narrowing research, to prioritise credible dated sources, and not to mistake navigation snippets for actual articles.
- **Free Tavily Basic publication filtering:** queries about latest/recent news or research default to a `month` publication filter, while the tool may specify `day`, `week`, `month` or `year`. Explicit historical timeframes override the automatic filter. Evergreen cinema/showtimes and other live-listings queries are not publication-date-filtered. These options do **not** enable Tavily Advanced search or increase searches beyond the configured 1–3 per message.
- **Stable source references:** result IDs `[1]`, `[2]` etc. stay aligned with the combined retrieved-source list across repeated searches, with duplicated URLs merged. Publication dates appear where actually supplied by the search provider.
- **Citation URL audit:** completed web answers are checked against the URLs *actually retrieved* by the tool. Unknown numbered references and external URLs absent from retrieved evidence trigger a visible warning and diagnostic events. This is a provenance check, **not full fact-checking**, and does not imply that a linked article was opened or read in full.
- Full Trace records the injected date, search filter, tool-source map and citation audit, so future problematic replies can be investigated directly.

Suggested manual tests with Web enabled: ask for recent black-hole research, today's Liverpool cinema showtimes, a historical academic result from 2024, and an answer with citations. With Diagnostics → Full Trace enabled, inspect the requests for correct date awareness, recency filtering, stable source IDs and citation warnings. Provider behaviour remains variable because automatic free-model rotation is intentional.
