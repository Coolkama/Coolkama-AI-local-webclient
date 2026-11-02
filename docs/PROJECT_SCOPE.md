# Project scope and guardrails

**Project:** Tight Arse AI (repository: `Coolkama/Coolkama-AI-local-webclient`)  
**Agreed:** 2 November 2026  
**Status:** Product requirements — use when proposing or implementing features

## Purpose and project boundary

This is an **independent, mobile-friendly, browser-based LLM workbench and chat client**. The GitHub repository retains its original Coolkama name, but the product is **Tight Arse AI** and it is **not** part of the separate PC-hosted Coolkama AI ecosystem. Do not merge their architectures, roadmaps, assumptions or requirements.

The application should be deployable as a **static webpage**, in the same general manner as Sciwrix: the website serves the application, and the browser performs its client-side work. It must not require a custom server, desktop companion, build service or local agent to operate.

It may call supported remote LLM/search APIs, and it may optionally connect directly to a user-configured Ollama endpoint, subject to browser security requirements, permissions and CORS. Ollama is an optional inference provider, **not** a dependency on the separate PC-hosted AI project.

## Local-file boundary (explicit requirement)

**Allowed**

- The user may explicitly select/upload a file through a browser file picker or equivalent browser-controlled selection interface.
- The client may read and parse **only those user-selected files** for the requested task (e.g. PDF text, plain text, code and images).
- Processed content may be supplied to the **selected** AI provider as part of a user-initiated request. The UI must explain that externally hosted providers receive submitted content; parsing locally does not mean inference stays local.
- Generated text, code, and other outputs may be **saved locally by triggering an ordinary browser download**, leaving the destination and any overwrite decision to the browser/operating system.
- Browser-managed local storage may retain chat history, selected attachments, configuration, model selection, usage counters and API keys.

**Not planned / prohibited unless these requirements are explicitly changed**

- Directly modifying, overwriting, deleting, renaming, moving or organising files already on the user's device.
- Background access to local folders, file scanning or arbitrary filesystem access.
- Persistent filesystem write permissions, file-system handles or a local daemon for editing files.
- Treating an uploaded document as permission to change the source document.
- Assuming that saving an output means modifying an existing input file: **save = download a new output via the browser**.

When adding file features, prefer the browser File API for explicit selection and Blob/download mechanisms for outputs. No unrestricted filesystem API is needed.

## Browser-first deployment and data boundaries

- Single-file `index.html` is the current application packaging goal. Use client-side React/JavaScript and CDN dependencies as appropriate; no server-side compilation or application backend is required.
- Responsive/mobile-first UI must work in normal modern browsers.
- Chat history/settings are stored **in the current browser and site origin**, not synchronised across devices. localStorage is browser storage, not GitHub storage or a server database.
- Browser restrictions (CORS, mixed content, local network permissions) may prevent direct access to certain endpoints. Explain such limits instead of bypassing browser security.
- API keys in localStorage are **not encrypted**. Never commit real credentials or hard-code shared secrets into the hosted HTML. External API providers receive the request data the user elects to submit.
- No automatic upload of selected local files to GitHub, the website host or unrelated services.

## Providers, search and costs

- The remote-model selection policy is **free-only**: no paid model fallback or automatic upgrade.
- Web search must use user-approved, free-tier-compatible approaches; do not silently enable chargeable search features.
- The browser exposes a `web_search` function to a tool-capable LLM when the user enables Web mode. The model uses conversation history to choose self-contained queries and may make follow-up searches. The browser executes Tavily Basic searches, returns structured evidence to the model, and displays the source links plus search queries in the chat. No automatic pre-search using the raw newest user message.
- Each external search and each LLM round-trip consumes one unit of the respective local request counter. Allow at most **three Tavily searches per user message**, with a selectable smaller maximum. No paid search, silent fallback or unbounded search loop.
- Some models lack OpenAI-compatible function calling; report unsupported tool use clearly rather than switching to a paid model or silently reverting to a less reliable method.
- Model-created Tavily search queries may include user conversation details, so explain this in the UI; treat retrieved web content as untrusted evidence, never instructions.
- Surface provider quotas and client-side usage stops. **Client-side counters do not guarantee zero charges on an account with billing enabled**; provider-side free-plan/billing protection is still necessary.
- Ollama may be used as a local inference endpoint where browser connectivity is supported.
- Offline/mock test mode should remain available.

## Streaming chat interface

- Render model-driven web searches and sources **above** the answer, in a single default-collapsed details panel. The header reads "Searching web..." while a web tool executes, then "Searched web" when it finishes. The panel can be expanded to inspect actual queries and source links. Do not show the panel for Web-enabled answers that did not use search.
- Keep the message scroller attached to the bottom only while the user is within a small bottom threshold. Any upward scrolling should suspend automatic following during streaming, while returning near the bottom or tapping **Latest** resumes it. Sending a new message may intentionally jump to the newest content.
- Replace the text-heavy Web toggle above the composer with a compact, labelled, accessible globe icon immediately next to the file-attachment button, with clearly distinguishable on/off states.
- Preserve free-only guards, provider rotation, model tool support, diagnostics and other product boundaries.

## Rolling input context window

- Construct a rolling **model request** on every inference round from the full locally retained conversation; do not delete or rewrite historical UI messages. Do not spend extra inference calls on summarisation. For continuation calls, preserve the **exact end** of the previous response when shortening the context and include it only once, as the immediately preceding assistant turn.
- Preserve system messages/current date, the latest two conversational turns and paired assistant tool calls/tool results. Older complete turns can fall outside the window. If recent required content exceeds the selected budget, visibly mark abbreviated excerpts; retain tool-call IDs and tool-result shape.
- Expose an estimated per-provider input budget (2,048–65,536 tokens), with Groq default **8,192** for more reliable code continuation, and other providers defaulting to 8,192. Migrate existing Groq profiles still on the old 3,072 default once while retaining other user-selected budgets. The larger Groq input may increase token-rate waits. Keep optional manual **output** tokens as a separate setting; Automatic must omit the provider `max_tokens` argument. Clearly state that text/vision token estimates are not a model-specific tokenizer or a guarantee against provider 429 or context-length errors.
- Reapply rolling selection for each new provider request, including model-requested web tools, recovery and manual answer continuation. Always preserve original source evidence in browser storage, with relevant URLs still accessible in the UI.
- Log window estimates, dropped complete turns, shortened excerpts and any unavoidable over-budget cases. Show a concise context disclosure beside the relevant assistant response.
- Retain free-only caps, model selection including rotating OpenRouter free routing and static browser-only architecture.

## Provider-specific continuation and caching

- Route continuation requests through a provider-specific strategy, not one universal prompt shape. Groq GPT-OSS 20B/120B may use an ending `assistant` message as a prefill; do not append a second user continuation request in that mode. Include the previous assistant text **exactly once**.
- Default Groq GPT-OSS prefilling to enabled, with a provider-profile checkbox to disable it if a specific model rejects the request. Other Groq models, OpenRouter, Gemini and Ollama continue using the normal assistant-plus-user message protocol until their capabilities are explicitly verified.
- Groq automatic prompt caching is server-side, opportunistic, and requires no separate caching call or session identifier. Preserve the static prompt prefix and a stable date-context message within a single answer for cache friendliness, while respecting context-window truncation and freshness on new turns.
- Retain the buffered overlap joiner, incomplete-code detection, Groq free-only account safeguards, bounded 429 waits and diagnostic logging of the protocol. Do not imply that prefill or cached tokens guarantee correctness or avoidance of rate limits.

## Seamless completion and bounded rate waits

- Default to **automatic continuation** on output-token-limit and unfinished code-fence responses. Buffer the beginning of each continuation until a trustworthy exact suffix/prefix boundary is established (up to a 12,000-character lookback); remove repeated opening fences and overlapped text **before** revealing new content. Reject obvious HTML restarts or redeclarations rather than silently corrupting code. Keep the final assistant answer in one message and provide a Settings opt-out.
- Maximum **six additional model requests per completed answer**; do not automatically use Tavily, a paid model or a different provider. Every request must reserve a free-use counter entry. If cap, provider, cancellation or continuation quality prevents completion, retain partial text, mark the reply incomplete, and preserve manual continuation as a fallback. Stop aborts pending work.
- For Groq **429**, parse `Retry-After` (numeric seconds or HTTP date), `Please try again in <duration>` and `x-ratelimit-reset-tokens`; support fractional seconds and hours/minutes/seconds. Retry only after the stated wait with a small guard, and only when the rejected attempt has emitted **no user-visible tokens**.
- Bound to three 429 retries per model request and 15 minutes per individual provider wait; do not wait indefinitely for daily limits. Respect the separate app-side Groq minute/day caps (minute waits bounded, day limits final). Do not silently retry HTTP 400/401/413 or non-Groq providers. If a Groq rate-limit error arrives after some text has streamed, preserve the partial text and use the bounded suffix-completion path instead of replaying the original request.
- The static/mobile browser cannot provide persistent background execution while the tab is closed or the OS suspends it. No background worker/server required. Log auto-continuation and rate waits in Normal/Full Trace without interrupting the ordinary chat presentation.

## In-place output continuation

- Offer a **Continue writing** action on the latest assistant reply when it ends at the output limit or with a visibly unfinished fenced-code block; allow older stored Done messages with an unclosed fence to be continued too.
- One press sends **one separately counted, free-tier-checked model request**; no automatic retry and no Tavily tools. Preserve the same existing assistant message and ID, append streaming text into its content and avoid a separate user/chat message.
- Put the original incomplete answer at the end of model context, following any stored source excerpts, and instruct the model to supply only the missing suffix. Do not reopen code fences or repeat text; trim common duplicated fence starts and short overlaps safely, without pretending arbitrary model-generated code is always syntactically valid.
- Treat a repeated output limit, unfinished fence, cancellation or provider failure as incomplete, preserve all safely aligned partial text and allow another manual continuation where suffix continuation is appropriate. Cap at six presses per answer and do not continue earlier replies after the conversation has progressed.
- Keep the existing free-tier local caps, provider selection and intentional OpenRouter free-model rotation. Record continuation starts/completions/failures in diagnostics.

## Output length and code completion

- Surface any provider `finish_reason: "length"` or equivalent as an **incomplete** response until a continuation finishes. Before marking a completed fenced HTML artefact as Done, syntax-check its inline classic JavaScript without executing it; flag malformed scripts and duplicate named functions as Incomplete, preserve the text and log the reason. A syntax pass cannot guarantee correct program behaviour.
- **Default to Automatic** for every provider profile: omit `max_tokens` entirely and let the model/provider choose its per-request output allowance. Do not retain an artificial 2,048, 4,096 or 8,192-token cap in Automatic mode.
- Permit an explicitly selected **Manual** positive integer output-token override with no arbitrary 8,192-token client ceiling; the provider still validates its own supported range. Migrate previously persisted profiles to Automatic by default while preserving their old numeric value for later Manual use.
- Keep automatic in-place continuation, Groq's bounded 429 retry, per-provider conservative browser quotas and no paid fallback. Provider-chosen output lengths can still hit independent tokens-per-minute, output, context or free-tier limits.

## Model-controlled page extraction

- Offer `web_open(index)` in addition to `web_search(query, time_range?)`. Only accept a validated 1-based index of a Tavily search result already returned **within the current user turn**. Never allow the model to choose an arbitrary URL or make browser requests to random websites.
- Implement using Tavily's **Basic Extract** endpoint only, no Advanced depth, paid proxy or fallback. The model can open at most three unique result pages per turn; the existing search budget remains separately limited to one to three searches. Reuse successfully opened pages without paying for duplicate extraction.
- Check and reserve the free plan local Tavily allowance before every extract. Count **one conservative local credit per attempted Basic Extract call**, even though Tavily presently charges one credit per five successful URLs. Share the existing 900/month Tavily counter with Basic searches, and check that a continuation LLM request remains possible before executing a model-requested tool.
- Pass at most 18,000 characters of extracted text to the model for one open; retain up to 12,000 characters of opened evidence per source in chat history for later follow-up questions. Full Trace can capture the actual raw extraction response. Mark truncation accurately.
- Show opened pages inside the existing collapsed web-activity UI, preserving search results and links. Treat extracted page text as untrusted evidence. Extraction failures and dynamic page omissions must be visible to the model and in diagnostics; do not invent missing content.
- Preserve OpenRouter rotating free models, Groq free-tool error handling, browser-only deployment and existing security safeguards.

## Groq follow-up tool validation

- A Groq model may emit invalid local tool arguments after a valid search; the API may then send a `tool_use_failed` error as a streaming event with initial HTTP 200. Preserve the error code in diagnostics.
- If already-retrieved sources exist and a Groq inference round fails specifically due to tool validation, allow **one** extra free-capped model request with tools explicitly disabled (`tool_choice: "none"`) and a final-answer instruction. Do not search again.
- Check the local LLM cap **before** the recovery request. On quota failure, malformed or tool-only output, or repeated provider failure, leave sources accessible and mark the answer incomplete.
- Do not alter OpenRouter Any Free Model rotation or introduce paid hosted tools and external services.

## Rotating-model web-tool interoperability

- Preserve OpenRouter's Any Free Model rotation even between function-calling rounds.
- Recognise the known Dots-style textual `web_search` envelope as an interoperability fallback, with strict parsing and validation. Do not execute arbitrary textual commands or tool names.
- Do not render textual tool markup as a successful assistant answer. Report the result as incomplete and preserve source links when the tool budget is exhausted or markup is malformed.
- Keep previously retrieved titles, URLs and bounded excerpts available to future follow-up requests as untrusted evidence; prefer reusing them before charging another Tavily search.
- Ensure a free inference continuation remains possible before making any additional Tavily request. If not, halt without spending the search credit.
- Preserve the existing free-tier counters, three-search maximum, diagnostics and static-browser-only architecture.

## Date-aware, evidence-grounded research

- Inject the current browser date/time, timezone and UTC date into every LLM request, including after tool calls. Do not rely on any model training cutoff as the current date.
- Preserve **OpenRouter Any Free Model rotation**. Different free models may legitimately handle successive inference rounds; keep full tool-call messages and tool results in the conversation each time. Never pin models without a user selection.
- Keep Tavily **Basic** search and the existing no-billing assumptions, request caps and maximum search-iteration budget. Use optional publication `time_range` filters for current-news intent; do not accidentally filter evergreen live listings, or overrule explicitly requested historical periods.
- Give tool results stable source indices across searches, deduplicate URLs and pass publication metadata only if the provider supplies it. Favour breadth and primary, dated research evidence rather than repeated searches for already-supported findings.
- Show a provenance warning for answer URLs not in retrieved results, out-of-range numeric references or absent traceable citations. Log the audit for inspection. Do not present this as independent fact-checking or full-page retrieval.
- Preserve the single-file static deployment, browser-only API calls, full diagnostics and no paid fallback.

## Local diagnostics

- Normal, Detailed and opt-in Full Trace modes are available in the browser UI. Diagnostics are for troubleshooting provider/model selection, streaming, search and tool orchestration, and do not change inference or search behaviour.
- Full Trace preserves actual model/search request and response payloads, including message history, extracted documents, image data and raw streaming events, **locally in IndexedDB**. Compact metadata is stored in localStorage. No remote log collector is used.
- Redact authentication headers, provider keys and other recognisable credentials. Never write keys into exported logs. Exports may contain other personal information, so explicitly warn the user to inspect before sharing.
- Allow JSON exports for the current chat or all logs. Use bounded rolling retention; local browser storage is not unlimited. Clearing logs removes both localStorage index events and IndexedDB full records.
- An exported log must reveal when Full Trace data could not be stored/retrieved; do not silently pretend that a partial trace is complete.

## GitHub repository resource policy

Use this repository for **source code, documentation, commits and version history**. The owner has disabled GitHub Actions and does not want usage or charges from CI, Actions minutes, build artefacts, caches, package hosting, Codespaces or Git LFS. Do not introduce those services, their workflows or generated assets without an explicit change of policy.

This policy does not prohibit normal Git source storage or ordinary browser localStorage.

## Acceptance checks for future changes

1. Can the application run as a static webpage, without any required custom backend or installed companion?
2. Are local files read **only after explicit user selection** and **never modified in place**?
3. Are saved outputs delivered through browser-managed downloads?
4. Is any transfer of selected content to a remote AI/search provider clear?
5. Does the change remain independent of the separate PC-hosted AI ecosystem?
6. Does it avoid GitHub Actions, artefact/cache consumption and paid-provider fallbacks?

If a proposed feature conflicts with these boundaries, seek an explicit requirements change before implementing it.
