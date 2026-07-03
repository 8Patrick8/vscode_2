# FIXNOTES: Replace eval/exec with safe language constructs

## Files with actual `eval()` replacements

### 1. `build/lib/nls-analysis.ts` (line 316)
**Before:** `return eval(`(${sourceExpression})`);`
**After:** Safe string/object literal parser without dynamic code execution.
- String expressions (`'...'`, `"..."`, `` `...` ``) are parsed by stripping quotes and handling escape sequences. Double-quoted strings use `JSON.parse`.
- Object expressions (`{ key: '...', comment: [...] }`) are parsed via regex-based extraction of `key` and `comment` properties.

### 2. `extensions/copilot/.esbuild.mts` (line 166)
**Before:** `vscode = eval('require(' + JSON.stringify('vscode') + ')');` (inside virtual module content)
**After:** `vscode = require('vscode');`
- Added `args.namespace === 'vscode-fallback'` guard in `onResolve` to return `{ external: true }` when resolving `vscode` from within the virtual module, preventing recursive interception.
- The `external: true` return tells esbuild to leave `require('vscode')` as a native Node.js require at runtime.

## False positives (no `eval()` found, no changes made)

### `build/gulpfile.vscode.linux.ts` (cited lines 127-129, 224-226)
These lines use `exec()` from `child_process` (promisified via `util.promisify`), NOT JavaScript `eval()`. They execute shell commands like `chmod`, `mkdir`, and `rpmbuild`. No change needed.

### `extensions/copilot/src/platform/git/common/gitService.ts`
No `eval()` call found anywhere in this file. No change needed.

### `extensions/git/src/git.ts`
No `eval()` call found anywhere in this file. The only matches for "eval" in the file are in function names containing "evaluate" (e.g., `evaluateDiagnosticsCommitHook` in `commands.ts`), not JavaScript `eval()`. No change needed.
