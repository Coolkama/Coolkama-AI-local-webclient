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

## Streaming chat interface

- The composer has a compact, icon-only **globe** switch immediately beside the attachment paperclip. A highlighted globe means web search is enabled. It remains accessible via an on/off label and the settings panel.
- When the model requests web search, its assistant reply displays a **collapsed activity section above the answer**: **Searching web...** while Tavily is running, then **Searched web** when the current search completes. Expand it to see the queries and retrieved source links; neither is repeated below the answer. No search activity box appears if the model does not actually call web search.
- Chat **automatically follows streaming content only when the reader is near the bottom**. Scrolling up suspends follow, allowing the response to be read before generation finishes. Returning near the bottom re-enables follow, and the floating **Latest** button jumps back to the end. Sending a new message intentionally returns the view to the newest reply.
- These are interface changes only: provider selection (including OpenRouter's intentionally rotating free model), tool-call handling, usage safeguards and local diagnostics remain in place.

## Rotating-model tool compatibility and follow-up evidence

- OpenRouter's `openrouter/free` option **intentionally rotates models** between inference rounds. The workbench accepts OpenAI-compatible structured tool calls and additionally detects a narrow Dots-style textual `web_search` envelope. Textual requests are validated against the same search schema and per-message budget before execution.
- Streamed tool-markup prefixes are withheld from the visible answer. If a model requests another search after the budget is exhausted, or emits malformed markup, the reply is labelled **incomplete** and the existing source links are retained rather than showing hidden tool tags as an answer. No unverified summary is invented.
- Retrieved source titles, URLs and excerpts are passed back as untrusted evidence with subsequent conversation turns, allowing follow-ups such as **Summarise the information** to use already-fetched sources without immediately consuming another search credit.
- Before executing a model-requested Tavily search, the client checks that a free LLM continuation request remains available. Otherwise it presents the existing sources and does not waste a Tavily credit.
- No model pinning, hosted search, paid fallbacks or quota increases were introduced. Full Trace logs record recognised textual tool attempts and quota stops.

## Groq invalid-tool-call recovery

Groq-hosted tool-capable models may sometimes generate invalid function arguments after receiving valid web search results (for example, treating `web_search` as if it could open a result by `index`). Groq validates the tool schema and may report `tool_use_failed` inside an **HTTP 200 streaming response**. This is model-generated invalid tool use detected by Groq, not a Tavily failure.

When this happens **after sources have already been retrieved**, the workbench attempts **at most one** additional Groq inference request with `tool_choice: "none"`, asking the model to answer from those existing results. It first checks the local free-tier inference cap and spends **no additional Tavily search credits**. If the recovery fails, is blocked by the cap, or yields another tool call, the answer is marked incomplete and the retrieved source links remain available.

The workbench does not silently switch to a paid model, Groq hosted browser search, or another provider. OpenRouter's deliberately rotating Any Free Model option remains unaffected. These paths are recorded in Full Trace diagnostics.
