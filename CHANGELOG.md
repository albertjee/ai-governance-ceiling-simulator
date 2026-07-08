# Changelog

## v1.50-xss-hardening - 2026-07-08 (branch: community-preview-1.5x-xss-hardening)

- Added a Content-Security-Policy meta tag to both `index.html` and `Guide.html`. `index.html` uses `script-src 'self' 'unsafe-inline'` (required since the app's logic is one inline `<script>` block with no build step); `Guide.html` uses the stricter `script-src 'none'` since it has zero scripts by design.
- Tightened the Guide iframe sandbox from `sandbox="allow-same-origin"` to `sandbox=""` (fully opaque origin, no script execution, no forms, no top navigation) — defense-in-depth against any future script added to `Guide.html`.
- `frame-ancestors` deliberately excluded from the CSP meta tag (spec-ignored via `<meta>`, same as `frame-src`); documented as a server-config item for whenever real HTTP headers become available, with a note that `Guide.html` needs `frame-ancestors 'self'` (not `'none'`) to avoid breaking its own embedding by `index.html`.
- Full corrected plan and rationale in `docs/XSS-Hardening-Plan.md`, including a record of two bugs caught in Claude's review of Claude Code's original draft before implementation: (1) the original `script-src 'self'` with no `'unsafe-inline'` would have blocked the app's own inline script from running at all; (2) the original fix for the iframe sandbox (deleting the attribute) would have removed all sandbox restrictions instead of tightening them.
- No scoring formulas changed, no build step added, no dependencies added. Verified via extracted-script `node --check`, tag-balance check, and manual confirmation of CSP/sandbox values in both files.
- This branch merges into `community-preview-1.5x` only, not into `main`.

## v1.50 - Community Preview - 2026-07-08 (branch: community-preview-1.5x)

- Removed the Scoring Trace panel and `Export Scoring Trace JSON` button from the visible GUI. The underlying diagnostic computation, `console.table`/`console.log` output, and `window.getScoringDiagnostic()` / `window.downloadScoringDiagnostic()` / `window.AI_GOVERNANCE_SIMULATOR_SCORE_TRACE` globals all remain fully functional from the browser console.
- Added a new `Open Guide` button in the results panel (same slot the Scoring Trace panel occupied) that toggles an in-page iframe loading the new `Guide.html`.
- Added `Guide.html` — a plain-language, publicly-oriented operating guide, styled to match the app, separate from the developer-facing `README.md`.
- Relabeled the simulator version from `Rev2` to `v1.50 — Community Preview` in the footer and in all scoring-trace JSON diagnostic fields (`simulatorVersion`, exported filename).
- Fixed a stale `CLAUDE.md` line claiming Chart.js CDN as a dependency; confirmed no Chart.js reference exists anywhere in `index.html` — the domain chart is hand-built.
- Updated `README.md` to reflect the removed panel/button and the new Guide button.
- No scoring formulas changed. This branch intentionally diverges from `main`/`master`, which remains the active development line.

## Rev2 - 2026-07-07

- Revamped color palette: midnight blue-black base with champagne gold accents.
- Added Geist Sans typography (system-optimized, no CDN dependency).
- Added CSS motion system: `--ease-smooth` and `--duration-base` variables for 320ms transitions.
- Polished toggle switches: larger, smoother, champagne gold accents.
- Added button hover glow and lift effect.
- Refined domain chart: deeper background, gold border accent, enhanced bar gradient.
- Tightened spacing across calibration cards, stage cards, metrics, and status cards.
- Added `border-left` accent on stage calibration cards with smooth transition.
- Updated focus rings across all interactive elements to use champagne gold.
- All scoring formulas and features preserved. All three lint gates pass.

## v1.5 - 2026-06-07

- Added scoring trace diagnostic panel.
- Added JSON scoring export.
- Added browser console scoring diagnostic.
- Added global diagnostic interfaces.
- Added Domain Maturity Profile graph polish.
- Improved mobile graph layout.
- Improved scoring trace readability.
- Preserved the static, no-backend, no-telemetry, no-storage, and no-network design.
- Preserved scoring formulas.
- No scoring formulas were changed in the Pass #2 UI/graph polish.

## 0.1.2 - 2026-06-03

- Clarified the result label from Governance-capped score to Final capped score.
- Expanded the AUP improvement label to Ratified AI Acceptable Use Policy.
- Added a Reset to baseline button for restoring default sliders and clearing all improvements.

## 0.1.1 - 2026-06-03

- Aligned the Governance Ceiling cap to the governance maturity +1.0 rule.
- Polished maturity stage labels for Stage 3 and Stage 5.
- Added keyboard focus and accessibility improvements for interactive controls and results updates.
- Hardened the footer disclaimer and updated the simulator version reference.
- Added print/PDF polish to preserve the executive dark visual style and reduce awkward page breaks.

## 0.1.0 - 2026-06-03

- Created the single-file AI Security Governance Ceiling Simulator.
- Added maturity calibration cards, sliders, improvement toggles, and executive-style results.
- Added README documentation and project constraints.
