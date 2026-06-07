# AI Security Governance Ceiling Simulator

A dependency-free, browser-based what-if simulator for exploring how AI security governance maturity can cap practical capability scores.

Version: v1.5

Open `index.html` in a browser to use the app. It runs entirely in the page with embedded HTML, CSS, and JavaScript.

## Included

- Step 0 maturity calibration cards for Stage 1 through Stage 5
- Starting maturity estimate slider from 1.0 to 5.0, defaulting to 1.75
- Domain maturity sliders for governance, technical capability, evidence confidence, NHI lifecycle, and logging / traceability
- Improvement toggles for common governance and evidence controls
- Reset to baseline control for restoring default sliders and clearing improvements
- Results for raw score, evidence-adjusted score, final capped score, blocker, next unlock, and highest-impact improvements
- Domain Maturity Profile graph with five adjusted domain bars, a final capped score vertical reference marker, and weakest/strongest domain summary
- Scoring Trace panel with an `Export Scoring Trace JSON` button
- Browser console scoring diagnostic
- Global diagnostic interfaces:
  - `window.AI_GOVERNANCE_SIMULATOR_SCORE_TRACE`
  - `window.getScoringDiagnostic()`
  - `window.downloadScoringDiagnostic()`

## Constraints

This project remains a static, single-file HTML/CSS/JavaScript application. It intentionally uses no React, npm, backend, telemetry, local storage, network calls, or external dependencies.

## Local Usage

1. Open `index.html` directly in a browser.
2. Adjust the maturity sliders and improvement toggles.
3. Use `Export Scoring Trace JSON` to save the current scoring diagnostic.
4. Open the browser DevTools console to inspect scoring trace output.

## Scoring Transparency

- The simulator is a public heuristic what-if model.
- It is not an official SANS score.
- It is not Microsoft Secure Score.
- It is not a compliance score.
- It is not an evidence-based tenant assessment.
- The exported JSON documents the current public heuristic scoring state only.

See `SCORING-METHODOLOGY.md` for the canonical scoring reference, including formulas, ordering, defaults, rounding, deterministic tie behavior, and the baseline worked example.

## Model Rules

- Raw capability score is heuristic.
- Evidence-adjusted score discounts weak evidence and traceability.
- Governance-capped score cannot exceed governance maturity + `1`.
- Final capped score may be constrained by governance, NHI lifecycle, logging / traceability, or evidence confidence.

## Manual QA Checklist

- Open `index.html` by double-clicking it locally and confirm the app loads without a server.
- Confirm the Step 0 maturity calibration cards display Stage 1 through Stage 5.
- Confirm the starting maturity estimate defaults to `1.75`.
- Move the starting maturity slider and confirm the displayed starting estimate updates.
- Move each domain maturity slider and confirm the results update.
- Toggle each improvement and confirm the score or recommendation output changes.
- Use `Reset to baseline` and confirm all sliders return to default values, all improvements turn off, and results refresh.
- Set governance maturity low, then high, and confirm Governance Ceiling status changes appropriately.
- Confirm the final score never exceeds `5.0`.
- Confirm the final score never exceeds governance maturity + `1`.
- Confirm the disclaimer is visible.
- Confirm the Domain Maturity Profile shows five adjusted domain bars and the final capped score as a vertical marker.
- Confirm the weakest/strongest domain summary updates.
- Confirm the Scoring Trace panel updates and remains scrollable.
- Confirm `Export Scoring Trace JSON` downloads the current scoring diagnostic.
- Confirm the browser console scoring diagnostic and global diagnostic interfaces remain available.
- Confirm no network calls, telemetry, local storage, or external dependencies are used.
- Resize the page to desktop and narrow/mobile width and confirm it remains usable.

## Known Limitations

- This is a heuristic simulator.
- It is not an official SANS score.
- It is not Microsoft Secure Score.
- It is not a compliance score.
- It does not inspect a real tenant.
- Evidence-based assessment requires a private collector or consulting review.

## Disclaimer

This is a heuristic what-if simulator, not an evidence-based tenant assessment. It is not affiliated with, endorsed by, or representative of SANS, Microsoft, or Microsoft Secure Score.

## Author

Created by Albert Jee  
IAM Consultant & Former Microsoft FastTrack Architect

© 2026 Albert Jee. All rights reserved.

This public simulator is a heuristic educational model. It is not affiliated with, endorsed by, or representative of SANS, Microsoft, or Microsoft Secure Score, and it is not an evidence-based tenant assessment.
