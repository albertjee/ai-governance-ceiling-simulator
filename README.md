# AI Security Governance Ceiling Simulator

A dependency-free, browser-based what-if simulator for exploring how AI security governance maturity can cap practical capability scores.

Version: v0.1.2

Open `index.html` in a browser to use the app. It runs entirely in the page with embedded HTML, CSS, and JavaScript.

## Included

- Step 0 maturity calibration cards for Stage 1 through Stage 5
- Starting maturity estimate slider from 1.0 to 5.0, defaulting to 1.75
- Domain maturity sliders for governance, technical capability, evidence confidence, NHI lifecycle, and logging / traceability
- Improvement toggles for common governance and evidence controls
- Reset to baseline control for restoring default sliders and clearing improvements
- Results for raw score, evidence-adjusted score, final capped score, blocker, next unlock, and highest-impact improvements

## Constraints

This project intentionally uses no npm, React, Vite, external dependencies, backend services, telemetry, local storage, or network calls.

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
- Confirm no network calls, telemetry, local storage, or external dependencies are used.
- Resize the page to desktop and narrow/mobile width and confirm it remains usable.

## Known Limitations

- This is a heuristic simulator.
- It is not an official SANS score.
- It is not Microsoft Secure Score.
- It does not inspect a real tenant.
- Evidence-based assessment requires a private collector or consulting review.

## Disclaimer

This is a heuristic what-if simulator, not an evidence-based tenant assessment. It is not affiliated with, endorsed by, or representative of SANS, Microsoft, or Microsoft Secure Score.

## Author

Created by Albert Jee  
IAM Consultant & Former Microsoft FastTrack Architect

© 2026 Albert Jee. All rights reserved.

This public simulator is a heuristic educational model. It is not affiliated with, endorsed by, or representative of SANS, Microsoft, or Microsoft Secure Score, and it is not an evidence-based tenant assessment.
