# XSS Hardening Plan

**Project:** `ai-governance-ceiling-simulator-xss-hardening`
**Branch:** `community-preview-1.5x-xss-hardening` (target merge: `community-preview-1.5x`)
**Baseline commit:** `2ebfb3b`
**Date:** 2025-07-07
**Status:** REVISED AND IMPLEMENTED — corrected after Claude review; see "Corrections from original draft" (§7) for what changed

---

## 1. Threat Model Baseline

Single static HTML app, no backend, no build step, no external dependencies, zero user input in DOM rendering.

| Surface | Status |
|---------|--------|
| `innerHTML` injection | ✅ Safe — all sources are hardcoded `domains[]` and `improvements[]` arrays |
| `eval` / `new Function` | ✅ None present |
| User-controlled DOM write | ✅ Zero — simulation state is numeric/boolean only |
| Script in Guide.html | ✅ None — pure static markup |
| JSON export | ✅ Safe — one-way Blob download, no re-parse into DOM |
| Exposed globals | ✅ Non-writable, non-configurable, `deepFreeze()` on all exported data |
| PostMessage surface | ✅ None — zero `addEventListener('message', ...)` calls in code |
| `<iframe src="Guide.html" sandbox="allow-same-origin">` | ⚠️ Benign today; future risk if Guide.html gets script |

**Aggregate XSS surface: Effectively zero.** No structural changes required to fix an actual vulnerability. Everything below is defense-in-depth.

---

## 2. Hardening Actions

### Action 1 — Add CSP Meta Tag to index.html and Guide.html (INLINE, file change) — IMPLEMENTED

**Rationale:** Adds a defense-in-depth layer against injection attacks. The app currently has zero CSP declaration.

**index.html** (contains the app's own inline `<script>` — must permit inline execution):

```html
<meta http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'none'; object-src 'none'; base-uri 'self';" />
```

**Guide.html** (genuinely scriptless — can be stricter):

```html
<meta http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'none'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'none'; object-src 'none'; base-uri 'self';" />
```

**Directive rationale:**

| Directive | Value | Why |
|-----------|-------|-----|
| `default-src 'self'` | Restricts all fetches to origin — none exist today, but blocks accidental future fetches |
| `script-src 'self' 'unsafe-inline'` (index.html only) | **`'unsafe-inline'` is required.** The app's entire logic lives in one inline `<script>` block with no build step or server-side nonce mechanism. `script-src 'self'` alone does NOT permit inline `<script>` execution — without `'unsafe-inline'` the app would fail to load at all (this was a bug in the original draft, caught before implementation). Same tradeoff already accepted for `style-src` below, applied consistently. This does mean the CSP does not block inline-script injection if a DOM-write vulnerability is ever introduced — acceptable given the current zero-injection-surface threat model, worth re-evaluating if the app's structure changes. Alternative considered: extract the script to an external `app.js` so `script-src 'self'` alone would suffice — deferred to preserve the single-file distribution model documented in README.md. |
| `script-src 'none'` (Guide.html only) | Guide.html has zero `<script>` tags by design — fully blockable, no tradeoff needed |
| `style-src 'self' 'unsafe-inline'` | Required because all styles are inline CSS in `<style>` blocks; no nonce mechanism without a server |
| `img-src 'self' data:` | Allows inline data URIs for any future image use |
| `font-src 'self'` | Fonts are loaded via system/shell stack; blocks accidental external font requests |
| `connect-src 'none'` | Explicitly deny fetch/XHR — no network calls exist or are desired |
| `object-src 'none'` | No plugins — standard hardening |
| `base-uri 'self'` | Prevents base tag hijacking if script were ever introduced |

**`frame-ancestors` deliberately excluded from this meta tag.** The CSP spec explicitly ignores `frame-ancestors` when delivered via `<meta>` — same restriction as `frame-src` (excluded below). Including it (as the original draft did) would have given false confidence that clickjacking protection was active when it silently does nothing. Real clickjacking protection is deferred to Action 3 (server header), where it actually works.

**What cannot be enforced via meta tag CSP:**
- `frame-src` — CSP Level 2 spec explicitly excludes `frame-src` from meta element enforcement. Browsers silently ignore it. A server HTTP header is required.
- `frame-ancestors` — same meta-tag exclusion as `frame-src`. See Action 3.
- Nonce-based inline script CSP — requires per-request nonce generation; a static nonce in HTML defeats its purpose.

**Implementation location:** Both `index.html` and `Guide.html` `<head>` sections, with different `script-src` values as shown above.

---

### Action 2 — Restrict Guide iframe sandbox to fully opaque (INLINE, file change) — IMPLEMENTED

**Rationale:** `sandbox="allow-same-origin"` keeps `Guide.html` under its true origin rather than an opaque one. Currently inert because `Guide.html` has no `<script>` tag whatsoever, but future-unsafe: if anyone later adds interactive content to `Guide.html`, the retained origin becomes exploitable surface.

**Current line in index.html (before this pass):**
```html
<iframe
  id="guideFrame"
  class="guide-frame"
  src="Guide.html"
  title="Simulator guide"
  sandbox="allow-same-origin"
  hidden
></iframe>
```

**Corrected fix — empty the sandbox value, do not delete the attribute:**
```html
<iframe
  id="guideFrame"
  class="guide-frame"
  src="Guide.html"
  title="Simulator guide"
  sandbox=""
  hidden
></iframe>
```

**Important correction from the original draft:** the first draft of this plan said to delete the `sandbox` attribute entirely. That would have done the *opposite* of what was intended — an `<iframe>` with no `sandbox` attribute at all has **no restrictions whatsoever** (full script execution if content had scripts, full same-origin access, top-navigation allowed). The restriction only exists while the attribute is present; an empty `sandbox=""` (or bare `sandbox`) is what actually triggers the maximally restrictive opaque-origin mode: no scripts, no forms, no same-origin access, no top navigation — regardless of what `Guide.html` ever contains. This was caught before implementation.

**Impact assessment:**
- `Guide.html` is already scriptless — this closes a future-risk path with zero functional change to the guide display
- No `postMessage` listener exists in `index.html` (confirmed by code audit) — nothing depends on same-origin iframe communication
- If `Guide.html` ever needs interactivity, add `sandbox="allow-scripts"` explicitly, evaluated against the actual use case at that time

**Do NOT add `allow-scripts` unless Guide.html gets script content.**

---

### Action 3 — Ensure `X-Content-Type-Options: nosniff` on Served Responses (SERVER CONFIG, not file change)

**Rationale:** The JSON export feature (`downloadScoringDiagnostic()`) creates a `Blob` with `type: "application/json"` and downloads it as a `.json` file. Without this header, a serving CDN/proxy could serve the file with `Content-Type: text/html`, and some browsers might MIME-sniff and execute it. This is relevant **only when the app is served over HTTP/S**, not for `file://` access.

**Implementation:** Add to serving infrastructure (nginx config, Netlify `_headers`, CloudFront response headers, S3 metadata, GitHub Pages custom headers, etc.):

```
X-Content-Type-Options: nosniff
```

**When a real hosting config is set up, also add `frame-ancestors` at the same time** (this is where it actually works, unlike the meta tag):
- `index.html`: `frame-ancestors 'none'` — nothing should ever embed it
- `Guide.html`: `frame-ancestors 'self'` — **not** `'none'`. `'none'` would block `Guide.html` from being framed by `index.html` itself, breaking the Guide button entirely.

**Not a file change** — no modification to `index.html` or `Guide.html`. This is a note for whenever real hosting-header configuration becomes available (e.g. GitHub Pages via a Cloudflare Worker, or similar).

---

## 3. Not Applicable (Ruled Out)

| Item | Reason |
|------|--------|
| SRI (subresource integrity) | No external scripts, stylesheets, or fonts are loaded from CDN. `href` for fonts uses the system/shell stack only. |
| HTTP security headers via HTML (HSTS, X-Frame-Options, Permissions-Policy) | These require HTTP response headers from the serving host, not `<meta>` tags. |
| Nonce-based CSP | Requires per-request nonce generation by a server. A static nonce in HTML source is trivially bypassed. |
| `frame-src` in CSP meta tag | CSP spec Level 2+ explicitly excludes `frame-src` from meta element enforcement. Browser ignores it silently. Server HTTP header required. |
| `frame-ancestors` in CSP meta tag | Same meta-tag exclusion as `frame-src`. Moved to Action 3 (server config note). |
| Server-side form validation | No server-side logic, no forms, no backend. |
| CSRF tokens | No state-changing server requests. |
| Rate limiting | No server endpoints. |
| Third-party dependency audit | Zero dependencies. |
| CORS configuration | No cross-origin API calls. |

---

## 4. Summary of Changes

| # | File | Change | Type |
|---|------|--------|------|
| 1 | `index.html` | Add CSP meta tag (`script-src 'self' 'unsafe-inline'`) at top of `<head>` | Inline |
| 2 | `Guide.html` | Add CSP meta tag (`script-src 'none'`, stricter) at top of `<head>` | Inline |
| 3 | `index.html` | Change `sandbox="allow-same-origin"` to `sandbox=""` on `<iframe id="guideFrame">` | Inline |
| 4 | Server config (future) | Add `X-Content-Type-Options: nosniff` and `frame-ancestors` (per-file values, see Action 3) | Server |

Items 1–3 are HTML file edits, implemented in this pass. Item 4 is a serving-environment configuration note for whenever this is hosted over HTTP(S) — not applicable to local `file://` usage.

---

## 5. Verification

After implementing changes:

1. **Parse check:** `python -m py_compile` not applicable to HTML; extracted the inline `<script>` from `index.html` and ran `node --check` — passed. Confirmed tag balance across both files.
2. **CSP test:** Open browser DevTools → Console/Network panel on `index.html`; confirm the app loads and functions normally (sliders, toggles, reset, Guide button all work) with zero CSP violation errors logged.
3. **Sandbox test:** `document.querySelector('#guideFrame').getAttribute('sandbox')` returns `""` (empty string, not `null`) — confirms the opaque-origin sandbox is active. Note: `element.sandbox` as a *property* is always a live `DOMTokenList`, never `undefined`, regardless of the attribute's presence — checking the attribute directly via `getAttribute` is the correct test, not the property.
4. **Linting:** `python C:/Dev2/Tools/lint.py -p "*.html" -I` (deferred per Albert — not run this pass)
5. **Git diff:** Only `index.html` and `Guide.html` modified; no other files touched

---

## 6. Alternative Considerations

### Option A — Keep `sandbox="allow-same-origin"` as-is

The Guide iframe currently poses zero risk. Tightening it is defense-in-depth, not a fix for an active vulnerability. Not chosen — the corrected fix (`sandbox=""`) costs nothing functionally, so there's no reason to defer it.

### Option B — `sandbox=""` (chosen)

The maximally restrictive sandbox: script execution blocked, opaque origin, no parent communication, no forms, no top navigation. Implemented.

### Option C — Keep `sandbox="allow-same-origin allow-scripts"` (NOT recommended)

Adding `allow-scripts` would enable script execution in Guide.html if script were added. Since Guide.html is intentionally scriptless, this is strictly worse than the status quo. Not chosen.

---

## 7. Corrections from Original Draft (Claude review, pre-implementation)

The first draft of this plan (Claude Code subagent output) had two bugs that would have caused real problems if implemented as written. Both were caught in review before any code was touched, and are corrected in the sections above:

1. **`script-src 'self'` with no `'unsafe-inline'`** would have blocked the app's own inline `<script>` from executing under CSP enforcement, breaking the entire simulator on load (sliders, calculations, everything). The draft applied the correct "no server means no nonce, so `'unsafe-inline'` is needed" reasoning to `style-src` but missed applying the same reasoning to `script-src`. Fixed by adding `'unsafe-inline'` to `script-src` for `index.html` (Guide.html gets `script-src 'none'` instead, since it's genuinely scriptless).

2. **Action 2's proposed fix (deleting the `sandbox` attribute)** would have done the opposite of its stated goal — removing the attribute entirely removes *all* sandbox restrictions, not just `allow-same-origin`. Fixed by using `sandbox=""` instead, which is what actually produces the maximally restrictive opaque-origin sandbox described in the original rationale.

3. **`frame-ancestors 'none'` in the CSP meta tag** does nothing — the spec explicitly excludes `frame-ancestors` from meta-tag enforcement, identical to the already-correctly-excluded `frame-src`. The draft caught the `frame-src` case but not `frame-ancestors`. Removed from the meta tag; moved to the Action 3 server-config note where it can actually take effect, with a specific caveat that `Guide.html` needs `frame-ancestors 'self'` (not `'none'`) to avoid breaking its own embedding by `index.html`.

4. **Verification step 3** originally checked `element.sandbox === undefined`, which is never true regardless of the attribute's state (`.sandbox` is always a live `DOMTokenList` property). Corrected to check `getAttribute('sandbox')` instead.
