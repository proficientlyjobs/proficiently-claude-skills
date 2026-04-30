# Browser Automation Setup

Standard sequence for skills that use browser automation tools to fetch web pages. Use the browser tools available in the current agent environment. In Claude Code, this is typically Claude in Chrome MCP. In Codex, use the available browser automation plugin or browser/search tools provided by the session.

## Tab Setup

Use the equivalent operations for the active agent:

```
1. Get browser state or identify the active tab
2. Create a new tab when needed
3. Navigate to the target URL
4. Extract page content
```

Claude in Chrome MCP tool names: `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `get_page_text`.

## Context Window Safety

**Avoid `get_page_text` on large or dynamic pages** (job boards, search results, listing pages, dashboards). It returns the entire page and can blow out the context window, making the conversation unrecoverable.

Instead, use targeted extraction:
- JavaScript evaluation with a selector to extract only the content you need
- Structured page reads to get element refs
- Full text extraction only for simple pages with a single article/posting

Claude in Chrome MCP equivalents: `javascript_tool`, `read_page`, and `get_page_text`.

## Error Handling

- If the browser-state tool returns no tabs or an error, ask the user to confirm that the browser and required browser automation extension/plugin are active.
- If `navigate` fails or the page doesn't load, ask the user to paste the content directly.
- If full text extraction returns empty or unusable content, try a structured page read as a fallback, then ask the user to paste if that also fails.
- Do not retry a failing page more than once. Move on and ask the user for the content.
