# FIXNOTES — Sanitize innerHTML/dangerouslySetInnerHTML usage

## False-positive occurrences (no change needed)

These assignments were flagged by the pattern but are safe as-is:

1. **`build/builtin/browser-main.js:69`** — `el.innerHTML = '';`
   - Clearing the container element; no user data is injected. Functionally equivalent to
     removing all child nodes. Not an injection vector.

2. **`extensions/mermaid-markdown-features/preview-src/shared/index.ts:64`** — `mermaidContainer.innerHTML = '';`
   - Same pattern: clearing the element before calling `mermaid.render()` to populate it.
     No user-supplied HTML is set here.

3. **`extensions/mermaid-markdown-features/preview-src/notebook/index.ts:58`** — `return temp.innerHTML;`
   - This **reads** `innerHTML`, it does not set it. The anti-pattern is about
     *assigning* unsanitized content to `innerHTML`, not reading it.

## Actual fixes applied

| File | Line(s) | Fix |
|------|---------|-----|
| `extensions/copilot/.../suggestionsPanelWebview.ts` | 166 | Changed `innerHTML` to `textContent` — the value is a plain number + label string, never HTML. |
| `extensions/markdown-language-features/markdown-editor-src/editor.ts` | 103 | Added trust comment: SVG output from mermaid.render(). |
| `extensions/markdown-language-features/notebook/index.ts` | 339 | Added trust comment documenting the `isTrusted` / DOMPurify dual-path. |
| `extensions/mermaid-markdown-features/preview-src/markdown/index.ts` | 44 | Added trust comment: content is mermaid SVG or error element. |
| `extensions/mermaid-markdown-features/preview-src/notebook/index.ts` | 45, 50 | Added trust comments: markdown-it output and mermaid SVG. |
| `extensions/mermaid-markdown-features/preview-src/shared/diagramManager.ts` | 257 | Added trust comment: static HTML for zoom controls. |
| `extensions/notebook-renderers/src/index.ts` | 112, 142, 232 | Added trust comments: each is guarded by `ctx.workspace.isTrusted`. |
