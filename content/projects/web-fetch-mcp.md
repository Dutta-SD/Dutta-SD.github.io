---
title: "web-fetch-mcp"
date: 2026-06-01
summary: "An MCP server for LLM agents that fetches web pages reliably — 3-tier escalation through static fetch, headless Chrome, and stealth Chrome, failing loudly instead of silently returning a CAPTCHA."
tags: ["mcp", "python", "llm-tooling"]
weight: 2
---

I kept running into the same problem with web-fetch tools in LLM agents: the tool would "succeed" but return a CAPTCHA wall or a JavaScript challenge page, and the model would confidently reason from garbage. web-fetch-mcp fixes that by detecting block pages and either escalating to a stronger fetching strategy or raising a `FetchBlocked` error — never silently handing junk to the AI.

It's published on PyPI (`pip install web-fetch-mcp`) and gives any MCP-compatible client two tools: `fetch` for retrieving pages as markdown, plain text, HTML, or article (main content only), and `screenshot` for rendering a page in a real browser and returning a PNG.

The core is a **3-tier escalation ladder**:

1. `curl_cffi` — fast static fetch with browser fingerprint impersonation (~500ms)
2. Patchright — real headless Chrome for JavaScript-heavy SPAs (~1–3s)
3. nodriver — stealth Chrome that evades automation detection (~2–4s)

If all three tiers get blocked, it raises `FetchBlocked` instead of returning the block page. The AI knows to try a different path rather than hallucinating from a 403.

**Code:** [GitHub](https://github.com/Dutta-SD/web-fetch-mcp) · **PyPI:** [web-fetch-mcp](https://pypi.org/project/web-fetch-mcp/)
