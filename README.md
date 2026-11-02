# Coolkama AI Local Web Client

A standalone, mobile-first LLM workbench and chat client built as a **single client-side HTML file**. It can be opened locally or served as a **static webpage**. It is **not part of** the separate PC-hosted Coolkama AI ecosystem.

> **Project requirements:** [Scope and guardrails](docs/PROJECT_SCOPE.md). Read these before making changes.

## What it does

- Multiple saved chats, streamed model responses, Markdown/code highlighting and code snippet downloads.
- Free-only remote model selection, conservative browser-side usage limits, and optional local Ollama or offline test mode.
- Opt-in free-tier web search.
- User-selected PDF, image, text and code-file attachments.
- Browser-local persistence for chats, settings and API keys.

## File access rules

**Users choose files to upload/process. Users can download generated outputs. The app must not edit, overwrite, delete, move or otherwise modify files already on their device.** File input is through browser-controlled selection, and saving is through normal browser downloads—not direct filesystem writes.

Selected content may be sent to the configured external AI provider when the user submits a request. Files and chats are not automatically uploaded to GitHub. Browser localStorage is origin-specific, not a synchronised account, and API keys stored there are not encrypted.

## Running

Open `index.html` in a modern browser, or publish it with a static website host. CDN dependencies require internet access unless separately cached. Configure a supported free-tier provider key (or use the offline test mode); direct remote API calls and LAN Ollama access depend on the browser's CORS/network permissions.

Client-side rate counters are **not** a billing guarantee: use genuine free provider plans and provider-side billing protections. No paid model/search fallback is intended.

## Repository policy

Source code and documentation only. GitHub Actions have been disabled by the owner; do not enable Actions or add workflows, CI, cache/artefact storage, Packages, Codespaces, Git LFS or other chargeable infrastructure without explicit approval.

## GitHub Pages

For static hosting, select **Settings → Pages → Deploy from a branch → main → /(root)**. The root-level `.nojekyll` file disables Jekyll processing; this repository needs no application build step. If the public site is not published yet, review the Pages publishing status and GitHub's deployment messages. The expected URL is https://coolkama.github.io/Coolkama-AI-local-webclient/ (only after successful publication).
