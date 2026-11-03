# Tight Arse AI

A standalone, mobile-first, free-tier-first LLM chat and web-search client built as a **single client-side HTML file**. The existing GitHub repository retains its original `Coolkama-AI-local-webclient` name. It can be opened locally or served as a **static webpage**. It is **not part of** the separate PC-hosted Coolkama AI ecosystem.

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

**Confirm that the latest version was actually published:** The bottom of the left-hand chat sidebar displays `Build: 2026-11-03-reasoning-visibility-v3`. If the label is missing, the browser is running an older `index.html`, or GitHub Pages has not updated. Open the Pages website with a fresh query string, e.g. `?version=20261103-provider-adapters-v2`, and reload. Query strings can bypass stale browser HTML caches but cannot publish an unbuilt deployment. The GitHub commit on `main` is not proof of a live Pages deployment; inspect **Settings → Pages** for the published source and status. No GitHub Actions or paid hosting infrastructure are enabled by this repository.

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

**Rate-limit recovery:** if Groq rejects a later model request (for example, HTTP 429 for its tokens-per-minute allowance), any search results and opened-page extracts already attached to the failed assistant message remain usable as untrusted evidence on the next turn. The model can answer a follow-up such as “Summarise the previous results” without losing those sources. This does not raise or bypass Groq's provider rate limits, and the separate automatic Groq retry mechanism may wait for a reported rate-limit reset before resending a rejected request, subject to free-tier safeguards.

## Model-driven page opening (`web_open`)

When Web is enabled, a tool-capable model can now call either `web_search(query, time_range?)` **or** `web_open(index)`. `web_open` reads the 1-based numbered result returned by a search **in the current user turn**, using Tavily's **Basic Extract API**. This avoids the invalid `web_search({index: 2})` calls observed with Groq. Only URLs from actual search results may be opened; the model cannot make arbitrary URL-fetch requests. Previously opened pages are cached for the remainder of the turn, so reopening the same source doesn't spend another Tavily credit.

The model may make at most **three web searches and three new page opens per user turn** (search count remains configurable from one to three). Extracted page text is passed to the model up to 18,000 characters per open, and up to 12,000 characters per opened result is retained with the conversation for follow-ups; Full Trace logs include the raw Tavily extraction response. An open may fail or be incomplete for paywalled or dynamically generated pages. There is no automatic Advanced Extract or paid fallback.

**Free-only accounting:** Tavily's free Researcher plan gives 1,000 API credits/month. A Basic search costs one credit; Basic Extract costs one credit per five *successful* URL extractions. The app deliberately reserves a conservative **full local credit for every extract call**, including failures, and uses the existing combined 900-call-per-month browser counter for both searches and opens. Each model continuation separately consumes a free LLM request. Browser counters do not account for calls from other apps sharing the same provider key.

The collapsed **Searched web** panel displays search queries, opened pages and source links, including which sources were read. The normal/detailed/full local diagnostics record extraction timing, failures, the returned text (Full Trace) and cache hits. The tool schema is made available dynamically when the corresponding per-turn budget permits. Both OpenAI-compatible structured function calls and the supported Dots-style textual calls can request `web_open`; OpenRouter's Any Free Model rotation remains enabled.

## Per-request output length (Automatic by default)

- **Settings → Output length → Automatic** is the default for all providers. The app **omits `max_tokens`** from OpenAI-compatible requests and lets the provider/model determine its own maximum output length. There is no client-imposed 2,048 / 4,096 / 8,192-token cap in this mode.
- **Manual token limit** is an optional positive whole-number override, stored separately for each provider profile. Only explicitly selecting Manual sends `max_tokens: N`. Existing saved 2,048/4,096-token preferences are retained for possible manual use but no longer silently restrict requests following the migration.
- Provider/model output allowances, context constraints, rate-limit caps and free-tier account quotas **still apply**. Groq may reject an oversized single request or later throttle tokens-per-minute; the existing bounded Groq wait/retry and local conservative counters remain active.
- When `finish_reason: "length"` indicates a truncated response, the answer remains incomplete unless a continuation actually finishes it. Diagnostics record `maxTokens: null` in Automatic mode and the actual manual override when selected.

## Continue truncated answers in place

When a free model reaches its output-token allowance (`finish_reason: "length"`) or finishes with an unclosed Markdown code fence, the **last assistant response** is marked **Incomplete**. A **Continue writing** button appears beneath it. The button is also offered for some older stored answers that were previously marked Done but contain an unclosed code fence.

Each explicit button press makes a free-tier-capped LLM continuation request, with tools disabled and a strict continuation instruction. It appends the new text to the **existing assistant message** instead of creating another chat message. The start is buffered, repeated fences and suffix/prefix overlap are aligned before showing new text, and the answer stays incomplete if it reaches the limit again, leaves code unclosed, restarts an earlier code section or fails HTML/JavaScript syntax validation. An interrupted or failed continuation preserves the previous text. Up to **six manual continuations per answer** are permitted. The button is restricted to the latest assistant message to keep conversation history coherent.

Automatic continuation is enabled by default (see below), while **Continue writing** remains a manual fallback when generation stops incomplete. There is no billing fallback, additional Tavily usage or model pinning. Each manual continuation consumes the current provider's free inference allowance and may still hit provider token-per-minute or context limits. Exact character-perfect continuation is model-dependent; deterministic alignment cannot invent missing expressions or ensure semantic correctness. Full Trace records `continuation-start`, `continuation-finished` and failure/stop events.

## Rolling context window

Long conversations now use a **deterministic, rolling request context** instead of sending the entire saved history to the model every time. Only the API request is windowed; the complete chat, source links, attachments and partial answers remain in local browser storage. There are **no summarisation inference calls** and no additional Tavily requests.

In **Settings → Rolling context window**, select an approximate *input-token* budget per provider: 2,048, 3,072, 4,096, 8,192, 16,384, 32,768 or 65,536. Groq now defaults to **8,192 estimated input tokens** to preserve more of a growing codebase during long answers. Existing Groq profiles still at the old 3,072-token default are upgraded once; deliberately configured other windows remain unchanged, and users can choose 3,072 again. Output length is independently set to Automatic (provider decides) unless the user explicitly switches to Manual. Other providers default to 8,192 estimated input tokens. The client estimates input tokens conservatively from text length and approximates image cost; **it is not a model tokenizer**, doesn't know a rotating model's exact context size and cannot guarantee the provider's TPM limits.

The client always keeps the system instructions and current date, plus the most recent two conversational turns. If these exceed the selected budget, it shortens long excerpts (marking omitted spans). Older *complete* turns are included newest-first only while they fit, and omitted from the request once the budget is exhausted. The structure of model tool calls and their matching results is never separated; shortened tool results remain valid JSON where possible. Research links/indices are preferred over verbose excerpts. Every inference round—including web searches, Groq text-only recovery and explicit code continuation—is re-windowed from the locally retained source context.

When history is omitted or shortened, the chat response displays a small rolling-context indicator. Normal diagnostics emit `context-window` with the estimated input cost, budget, number of older messages/turns omitted and count of shortened excerpts. The original chat is never rewritten by this feature.

This is a **recency-based window, not semantic memory or LLM-generated summarisation**. Information from older dropped turns may be unavailable to the model unless the user restates it or asks to retrieve the sources. Request-token estimates, provider context limits and true token-per-minute consumption vary by model.

## Provider-specific continuation strategies (Groq GPT-OSS)

- A provider-specific continuation router now selects **assistant-message prefilling** for Groq `openai/gpt-oss-20b` and `openai/gpt-oss-120b`, when **GPT-OSS assistant prefilling** is enabled in Groq Settings (default on). The final API message is the **existing assistant response**, once, without an additional user `Continue` request or duplicated code. The model is asked to generate the suffix of that assistant message.
- Groq non-GPT-OSS models, OpenRouter, Gemini and Ollama retain the ordinary assistant-plus-user-follow-up protocol. Disabling Groq prefilling returns to that protocol. This is a capability-specific change, not an assumption that all providers expose identical continuation behaviour.
- For Groq prefilling the exact system/date/user/tool-request prefix of the prior inference round is reused unchanged when it fits the configured input budget; otherwise a conventional follow-up is used. Groq **automatically** caches matching prompt prefixes for supported GPT-OSS models; the app sends no session ID or separate caching API call. Cache hits are opportunistic, not guaranteed.
- Groq still enforces account-specific free-tier limits, output ceilings, and TPM/RPM/RPD quotas. Increasing the input context can increase 429 waits. The client still performs buffered suffix overlap alignment and final code validation; prefilling is not a guarantee of correct code. `auto-continuation-start` and `continuation-start` diagnostics include the selected protocol.
- Groq's API documents assistant prefilling, but we have **only run simulated regression tests** here, not a live free-key request. If your Groq model returns an incompatible-request error, disable prefilling under Groq Settings and export the diagnostics.

Official references: [Groq assistant prefilling](https://console.groq.com/docs/prefilling), [prompt caching](https://console.groq.com/docs/prompt-caching), [Groq rate limits](https://console.groq.com/docs/rate-limits).

## Reasoning-only streaming diagnostics

Certain OpenRouter free-router models (observed with `dots-studio/dots-3-note-preview:free`) can stream thousands of `delta.reasoning` / `reasoning_details` chunks before emitting any visible `delta.content`. This is **not** an inactive network request. The chat now shows a lightweight **Model reasoning** activity counter and the actual selected model name while waiting for the first visible answer. It never displays private reasoning text in the chat. The stream records total reasoning characters/chunks in `llm-round-end` diagnostics; Stop continues to abort the request immediately. Reasoning-heavy free models may take a long time or never produce a final response within the chosen timeout/allowance; no automatic paid switch is introduced.

The 3 November diagnostics also showed GPT-OSS 120B returning `finish_reason: length` after 3,072 total completion tokens on each request even though the client sent no `max_tokens`. This is **observed provider-chosen behaviour**, not evidence that the model supports no higher output limit. To experiment with larger single completions, select a positive **Manual output token limit** under Settings, keeping provider free-tier token budgets and rate limits in mind. No automatic output limit is silently imposed.

## Provider adapters, streaming metadata and caching integrity

- **Groq:** All web-enabled inference rounds in a single user answer advertise the *same* two Tavily Basic function schemas and `tool_choice: auto`. Browser-side search/open quotas remain enforced at execution. Cache-friendly GPT-OSS prefilling reuses the exact previous model request prefix and tool configuration; if the context budget cannot accommodate the prefix, it uses a normal assistant-plus-user continuation. The original prompt is not recreated or shortened in the prefill path. Cache tokens are read from the trailing SSE usage event when the provider emits them. `cachedTokens: null` means the API did not report a figure, not that a cache miss is proved.
- **Groq prefilling is experimental:** Models may still restart an HTML document even with a preserved prefix. Automatic completion can retry once using the ordinary follow-up protocol after a detected restart or incompatible prefill error, counting that *second* inference request toward both the free request counter and the **six total automatic continuation requests**. Manual **Continue writing** sends **one model request per press**; a misaligned result leaves the code intact and allows the next press to use normal follow-up. Manual continuations do not advertise Tavily tools.
- **Gemini:** Streamed function-call `extra_content.google.thought_signature` is retained on the original call and echoed with the assistant tool-call turn, before its corresponding tool results. Assistant-level Gemini extra metadata is retained in completed chats and replayed only to Gemini. This prevents losing provider-required opaque state while keeping the existing OpenAI-compatible endpoint.
- **OpenRouter:** Free fixed-model requests include a stable per-chat `session_id` for provider routing, plus `usage: { include: true }` for in-stream accounting. `openrouter/free` deliberately omits the sticky session routing key so it can continue rotating free models. Streamed `reasoning_details` blocks are preserved in order and returned intact with subsequent tool results and same-provider conversation turns.
- **Every provider:** The normal model API remains free-tier-only, and no paid fallback, hosted web tool, background service or user filesystem permission is added. Provider-controlled caching is opportunistic and these changes are not a guarantee of complete HTML or a cache hit. The client retains diagnostics for protocol choice, cache fields, failed boundaries and fallbacks.

## Seamless automatic completion and Groq rate-limit waiting

**Seamless long answers** is enabled by default and can be turned off under Settings. When a model response hits `finish_reason: "length"` or ends with an unclosed fenced-code block, the workbench automatically requests **only the missing suffix**, appends it to the **same assistant message**, and continues normal streaming without displaying a separate continuation message. Continuation starts are buffered and checked using exact suffix/prefix overlap up to 12,000 characters, duplicate-code-fence handling and restart/declaration guards; ambiguous starts are not displayed until aligned. Up to **six additional inference requests** may be made per answer, bounded by provider free-tier caps; no automatic paid provider switch, hosted tool, or Tavily search is triggered. If completion remains impossible, the saved answer is marked incomplete and the older manual Continue button remains available. The full trace records auto-continuation starts, completions and stops; normal chat UI does not expose individual continuation rounds.

Groq HTTP 429 / `rate_limit_exceeded` responses are handled with an **abortable wait-and-retry loop**, using `Retry-After`, Groq's “please try again in” message (including decimal seconds or minute/second combinations), or its token reset header. Retry waits add a short guard margin. The workbench makes at most **three rate-limit retries per inference call**, with **15 minutes maximum per provider-requested wait**; waits longer than this and daily exhaustion stop instead of trying indefinitely. The conservative local Groq per-minute counter may independently require waiting up to one minute, at most twice per request. **Every attempted request is counted**, including rejected 429 requests. During the wait the same assistant turn remains active, and **Stop** aborts the wait. If the browser tab is closed, reloaded, suspended by the phone, or loses connectivity, this browser-only application cannot guarantee the wait and automatic retry will finish.

**Continuation integrity:** For continuation rounds, a shortened previous assistant message now preserves the latest code tail rather than the older head. The previous code is included **once**, as an assistant message, not copied again into the user instruction. A generated fenced HTML document is checked for unclosed scripts, duplicate named functions and JavaScript syntax errors before the turn is marked Done. An invalid join or script is marked Incomplete with a visible explanation and diagnostic event; no automatic extra model-based repair calls are spent. This is a syntax safeguard, not a full application runtime/quality test. Small configured input windows may still omit earlier implementation detail.

Large input/output settings can cause a Groq request to exceed its limit on a *single request* (such as HTTP 413). That cannot be fixed by waiting and is not retried. Other providers' 429 responses are not automatically retried. Provider output length and model context remain subject to individual limits.
