# UI Polish Pass — Design Spec
Document Revision: REV 1.0
Project: AI Security Governance Ceiling Simulator
Last Updated: 2026-07-07
Status: Draft

## Purpose

Polish the `index.html` simulator's UI to feel more refined executive-grade — tighter typography, richer midnight-blue + champagne-gold palette, smooth motion, and better spacing rhythm. No feature changes, no scoring changes, no new elements. Pure CSS and font polish only.

---

## 1. Typography — Geist Sans

### Changes

- Replace `Inter` font stack with `Geist` / `Geist Sans` as primary system font
- Fallback chain: `Geist Sans, Geist, "Inter var", "Inter", system-ui, sans-serif`
- Load nothing from Google Fonts or CDN — use system-installed Geist where available, Inter elsewhere via `Inter var` auto-upgrade
- Tighten heading line-height: H1 from `0.98` to `0.95`
- Reduce label `letter-spacing` on `h1` from `0` to `-0.01em` for Geist
- Tighten body line-height where appropriate
- Tighten `.metric-value` sizes — reduce `clamp` upper bound slightly
- Kicker / label text: add `letter-spacing: 0.1em` (already present on kicker, confirm all labels)
- Stage cards and badges: tighten padding, slightly smaller font (already at 0.78rem, fine)

### Inter Var Note

`Inter` in modern browsers (Chromium 99+, Firefox 97+) renders as `Inter var` automatically from the system font stack without a font-load network request. This achieves near-Geist typography feel with zero external dependencies and zero latency.

---

## 2. Color Palette — Midnight Blue + Champagne Gold

### CSS Custom Properties Changes

Replace the existing `:root` color block:

```css
:root {
  color-scheme: dark;
  --bg: #060810;
  --panel: #0d1424;
  --panel-2: #131d35;
  --line: #1a2250;
  --text: #f0f2f8;
  --muted: #8892a0;
  --gold: #b8960e;
  --gold-mid: #d4a82a;
  --gold-soft: #e8c84a;
  --red: #7a1a22;
  --red-soft: #d4707a;
  --green: #4a8a6a;
  --shadow: 0 22px 70px rgba(0, 0, 0, 0.6);
}
```

Differential from current palette:
- `--bg`: `#06080d` → `#060810` (cooler, deeper black-blue)
- `--panel`: `#111a2a` → `#0d1424` (midnight navy)
- `--panel-2`: `#111a2a` → `#131d35` (lighter panel layer)
- `--line`: `#273142` → `#1a2250` (deeper blue-tinted grid lines)
- `--text`: `#f6f7fb` → `#f0f2f8` (near-white)
- `--muted`: `#b5becd` → `#8892a0` (cooler, more muted)
- `--gold`: `#c9a646` → `#b8960e` (champagne amber)
- `--gold-soft`: `#f0d37d` → `#e8c84a` (brighter champagne)
- `--red`: `#9e1f2f` → `#7a1a22` (deeper crimson)
- `--red-soft`: `#f0a7ae` → `#d4707a` (softer, more refined red)

### Shadow Upgrade

Deepen shadows throughout:

```css
--shadow: 0 22px 70px rgba(0, 0, 0, 0.6);
```

Also add subtle inner glow on panel edges via `box-shadow` on `.panel`:

```css
box-shadow: var(--shadow), inset 0 1px 0 rgba(232, 200, 74, 0.05);
```

### Background Gradient

Tighten the body gradient to accent the midnight direction:

```css
background:
  linear-gradient(120deg, rgba(184, 150, 14, 0.07), transparent 28%),
  linear-gradient(180deg, #080c18 0%, #060810 46%, #04050a 100%);
```

### Accent Glows

Add subtle glow to gold-highlighted elements (final-marker, active switches, selected states):

- `.final-marker`: already has `drop-shadow(0 0 8px rgba(240, 211, 125, 0.5))` — update to match new gold: `drop-shadow(0 0 8px rgba(232, 200, 74, 0.55))`
- Toggle switches: update `:checked` colors throughout
- Active `.panel-header`: subtle `rgba(232, 200, 74, 0.06)` top border inset

---

## 3. Spacing Rhythm

### Tightening Targets

- `.calibration` cards: reduce `gap` from `12px` to `10px`; reduce card `padding` from `16px` to `14px`
- `.stage-card` `min-height`: `160px` → `148px`
- `.stage-card` `padding`: `16px` → `14px`
- `.slider-row` `margin-bottom`: `18px` → `14px`
- Results `.metric`: reduce `padding` from `16px` to `14px`; reduce `min-height` from `126px` to `116px`
- Status band `.status-card`: reduce `min-height` from `148px` to `138px`

---

## 4. Motion

### Global Transition Defaults

Add to `:root`:

```css
:root {
  --ease-smooth: cubic-bezier(0.16, 1, 0.3, 1);
  --duration-base: 320ms;
}
```

### Specific Transitions

- `.bar-fill`: change `transition` from `width 180ms ease` to `width var(--duration-base) var(--ease-smooth)`
- `.switch::after`: `transform` and `background` transition: 160ms → `var(--duration-base)` with `var(--ease-smooth)`
- `.switch`: `background` and `border-color` transition: 160ms → `var(--duration-base)`
- `.panel`: add `transition: box-shadow var(--duration-base) var(--ease-smooth)` for subtle depth on hover (if hover states exist or are added for panels)
- Value text updates (`#cappedScore`, etc.): numbers in this app don't animate via CSS — that's fine, JS-driven rendering is instantaneous

### Toggle Switch Polish

Refine the toggle from 24px height / 16px knob / 18px translation to slightly larger:
- Width: `42px` → `44px`
- Height: `24px` → `26px`
- Knob: `16px` → `18px`
- Translation: `18px` → `20px`
- Knob position offset: `3px` → `4px`

Update CSS variables and switch styles accordingly.

### Button Hover Polish

Add subtle scale and glow on hover for `.reset-button`:

```css
.reset-button:hover {
  background: rgba(184, 150, 14, 0.15);
  border-color: rgba(232, 200, 74, 0.75);
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(184, 150, 14, 0.18);
}
.reset-button {
  transition:
    background var(--duration-base) var(--ease-smooth),
    border-color var(--duration-base) var(--ease-smooth),
    transform var(--duration-base) var(--ease-smooth),
    box-shadow var(--duration-base) var(--ease-smooth);
}
```

### Focus State Polish

Refine focus rings across all interactive elements to use new gold:

```css
input[type="range"]:focus-visible {
  outline: 2px solid #e8c84a;
  outline-offset: 5px;
  border-radius: 999px;
}

.toggle:focus-within {
  outline: 2px solid #e8c84a;
  outline-offset: 3px;
  border-color: rgba(184, 150, 14, 0.65);
}

.reset-button:focus-visible {
  outline: 2px solid #e8c84a;
  outline-offset: 3px;
}
```

---

## 5. Chart / Data Visualization

### Bar Fill Enhancement

- Add subtle gradient to `.bar-fill`:

```css
.bar-fill {
  background: linear-gradient(
    90deg,
    rgba(184, 150, 14, 0.65),
    #e8c84a 80%,
    #f0d080
  );
  box-shadow:inset 0 1px 0 rgba(255,255,255,0.12);
}
```

- `.bar-track`: deepen background from `rgba(0,0,0,0.42)` to `rgba(0,0,0,0.55)`

### Domain Chart Card

- `.domain-chart` background: `rgba(0,0,0,0.16)` → `rgba(0,0,0,0.25)` for more depth
- Add card border color: `rgba(184, 150, 14, 0.15)` (subtle gold border)
- `.chart-axis` text: update from `--line` colored ticks — ticks should use `rgba(240,242,248,0.25)` for subtlety

### Chart Axis Refinement

Add `aria-label` updates to chart ticks section. Already has `aria-hidden="true"` on the axis — fine.

---

## 6. Calibration Cards

Polish the calibration helper and calibration card grid:

- `.calibration-helper`: already at `0.92rem` — bump to `0.88rem` for tighter hierarchy; reduce `margin-top` from `24px` to `18px`
- Add subtle border-left accent on stage cards:

```css
.stage-card {
  border-left: 2px solid rgba(184, 150, 14, 0.2);
  transition: border-left-color var(--duration-base) var(--ease-smooth);
}
```

---

## 7. Responsive Refinements

Minor polish only:
- At `760px`: calibration grid from `repeat(3, minmax(0, 1fr))` is fine — single column on `480px` is fine
- At `480px`: tighten padding on `.app` from `14px` → `12px`

No structural layout changes.

---

## 8. Print Polish

Print styles already exist — update `box-shadow: none` is fine. No print color changes needed — `@media print` was already solid.

---

## 9. Accessibility

- All existing ARIA attributes remain
- Update focus ring colors from old gold to new champagne gold throughout
- Ensure `.bar-fill` contrast meets WCAG AA against its track background — already borderline, deepen track bg
- Color changes: ensure `--muted` (#8892a0) vs `--text` (#f0f2f8) contrast is sufficient — 9.5:1 on dark bg, well above AA
- Ensure `--red-soft` (#d4707a) on dark panels meets AA — ~5.2:1, pass on large text only; confirm it's used on `.metric-value` (large text) not body text — it's only on metric-value class, which is `1.45rem` and above, PASS

---

## Exclusions (Out of Scope)

- No scoring formula changes
- No new domains or improvements
- No new result cards
- No animation library additions (pure CSS transitions only)
- No JavaScript changes
- No structural HTML changes (elements, layout)

---

## Files Affected

- `index.html` — CSS `:root` variables, typography, color, spacing, motion, and interactive style updates only

---

## Review Checklist

- [ ] All old gold references replaced with new champagne palette
- [ ] Geist typography renders or degrades gracefully to Inter
- [ ] Motion is smooth and not distracting
- [ ] Print styles still work
- [ ] All three gates pass (parse, lint, git diff)
- [ ] Only `index.html` changed
- [ ] Scoring unchanged (no accidental JS changes)