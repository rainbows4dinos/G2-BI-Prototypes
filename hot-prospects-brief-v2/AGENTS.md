# Hot Prospects Brief (v2)

## Overview

v2 unifies the 5 divergent persona views from v1 into a single `.card` component design. Where v1
gave each persona a distinct layout (Francisco's table, Stacy's segment cards, Iesha's breakdown
sections, Alex's hero cards, Maya's row list), v2 renders all personas through one consistent card
shape. The card adds rich "why call" rationale via `computeWhyCallV2` (employees + dominant signal
verb), a 4-card summary stats row always reflecting full data, 7 quick-filter preset chips with
AND logic, a hot accent bar (4px purple left edge when composite ≥ 80), color-coded signal pills
(5 types × 2 themes = 10 tokens), and Iesha's per-card score expansion plus a static "Scoring
Math" reference card. v1 remains untouched as the divergent-view reference implementation.

## v1 Baseline

v2 retains the following from v1 unchanged:

- **5 personas** (Francisco, Stacy, Iesha, Alex, Maya) — same names and definitions, but ALL
  render through the unified `.card` component in v2
- **Mock data structure** — 50 companies + 4 QA fixtures, identical schema
  (`intent_score`, `company_employees`, `industry`, `geo`, `personas_detected`, `recent_signals`)
- **ICP scoring functions** — `dimensionMatch`, `computeFit`, `computeComposite` — unchanged
  signatures and behavior
- **Dark mode framework** — `[data-theme="dark"]` CSS overrides + localStorage-driven detection
- **Elevate Design System tokens** — same 19 base tokens; v2 adds 10 signal pill tokens on top
- **Accessibility** — ARIA labels, semantic HTML, keyboard navigation, reduced-motion support

**See `../hot-prospects-brief/AGENTS.md` for the full v1 spec.** This doc covers v2 deltas only.

## Tech Stack

Single-file React 18 via CDN + Babel Standalone — same as v1. Elevate Design System tokens.
Material Icons via Google Fonts CDN. Static HTML, no build step, no bundler, no npm.

## File Structure

`index.html` (~1070 lines) — single-file prototype organized top-to-bottom:

- **CSS tokens** (lines ~33–280) — same 19 base tokens as v1, plus 10 new signal pill tokens
  (lines 65–117) and dark-mode overrides for all new tokens
- **Dark mode block** (lines ~79–130) — same `[data-theme="dark"]` structure as v1
- **Mock data** (line ~405) — identical to v1; 50 companies + 4 QA fixtures
- **Scoring** (lines ~467–535) — v1 functions unchanged; new `computeWhyCallV2` at line 486;
  `SIGNAL_META`, `SIGNAL_PRIORITY_V2`, `SIGNAL_VERB` constants
- **Components** (lines ~540–960) — `SummaryStats`, `PresetChips`, unified `.card` component,
  Iesha's `ScoringMathCard` and per-card expansion drawer
- **App root** (lines ~960–1070) — state, enrichment pipeline, filter logic, theme toggle

## Personas

Same 5 personas as v1. All render through the unified `.card` component. Persona-specific behavior
in v2 is limited to:

- **Francisco** — ranked feed, no persona-specific additions beyond the unified card
- **Stacy** — segment badges (Enterprise / Mid-market / SMB) visible on all cards; no
  Stacy-specific 70 threshold (removed; hot threshold is now uniformly ≥ 80)
- **Iesha** — static "Scoring Math" reference card pinned at top of view; per-card score
  expansion (click score badge to reveal intent bar, fit bar, formula, dimension chips)
- **Alex** — unified card; no separate hero-card layout
- **Maya** — auto-activates the "SMB" preset chip on persona switch; chip and persona share
  single source of truth (no divergent state)

All personas now show: hot accent bar (≥ 80), score + intent underline, segment badges, signal
pills, "why call" rationale.

## Scoring

All v1 scoring functions inherited unchanged: `dimensionMatch`, `computeFit`, `computeComposite`.

**New in v2:**

`computeWhyCallV2(company, mockNow)` (line 486) — generates a rich rationale string.

- Format: `"{employees} employees · {dominant verb} ({count}x in last 7 days)"`
- Priority order: pricing (1.0) > compare (0.9) > competitors (0.7) > category (0.5) > profile (0.3)
- Drops the employee prefix when `company_employees` is null
- Falls back to `"Active on G2 recently"` if no signals in the 7-day window
- Verb mapping: pricing → "researching pricing", compare → "running comparisons",
  competitors → "researching competitors", category → "browsing {industry}",
  profile → "viewing profile"
- Exposed for QA at line 526: `window.__test.computeWhyCallV2`

**Hot threshold**: composite ≥ 80, applied consistently across all personas. v1's Stacy-internal
70 threshold is removed.

## Key Constants (Deltas from v1)

- **localStorage key** (line 15) — `'hpb-v2-theme'`, decoupled from v1's `'hpb-theme'`; v1 and
  v2 maintain independent theme state; sharing this key breaks v1's theme isolation
- **7 quick-filter preset chips** — `all`, `hot`, `pricing`, `comparison`, `enterprise`,
  `midmarket`, `smb`; multi-select with AND logic between active presets
- **4 summary stat cards** — "Companies", "Hot 80+", "Pricing signal", "Comparison signal";
  always reflects full enriched data, not the filtered subset
- **10 signal pill tokens** — 5 types (profile, pricing, compare, category, competitors) × 2
  themes (light + dark); token names follow `--sig-{type}-bg` / `--sig-{type}-text` pattern
- **Hot accent bar** — 4px left edge on `.card--hot`, uses `var(--primary)` purple; no new
  accent color tokens added
- **2 actions per card** — "Add to CRM" + "Email" only; v1's Skip, Slack, and Mark contacted
  are removed

## How to Extend (v2-specific)

### Add a new signal type

1. Add verb mapping to `SIGNAL_VERB` object (line 479–484)
2. Add priority weight to `SIGNAL_PRIORITY_V2` (line 471–477)
3. Add CSS token pair `--sig-{type}-bg` and `--sig-{type}-text` in both light and dark blocks
   (near lines 65–117)
4. Add `.pill--{type}` CSS class near the existing pill classes (line ~335–339)
5. Update `SIGNAL_META` (line ~529–535) with icon and label
6. Add to preset chips array if it should be filterable; add to summary stats if it needs counting

### Add a new preset filter chip

1. Add preset object to `presets` array in `PresetChips` component (line ~692–700)
2. Add filter logic to `applyFilters` function (line ~625–655)
3. Add preset key to `handlePreset` switch logic (line ~702–712)
4. Optionally wire up persona-driven auto-activation following the Maya → smb pattern
   (lines 587–603)

### Tweak the why-call dominant-verb mapping

1. Locate `computeWhyCallV2` (line 486)
2. Edit `SIGNAL_VERB` object (line 479–484) for verb text changes
3. Edit `SIGNAL_PRIORITY_V2` (line 471–477) if priority order changes
4. Test via browser console: `window.__test.computeWhyCallV2(company, mockNow)`
5. Verify format: `"{employees} employees · {verb} ({count}x in last 7 days)"`

### Add a new summary stat card

1. Add calculation logic to `SummaryStats` component (line ~781–796); filter `enriched`
   companies on the criterion with a 7-day cutoff where applicable
2. Add new `.stat-card` div with `.stat-label` + `.stat-value` (line ~790–793)
3. Update the summary stats `role` label if needed (line ~789)
4. Verify aria and responsive behavior; existing CSS grid handles layout automatically

## Invariants (v2-specific)

- **Hot accent bar uses `--primary` purple** — do not introduce new accent color tokens; the
  existing `var(--primary)` is the only hot-state color
- **2 actions per card only** (Add to CRM + Email) — Skip, Slack, and Mark contacted are not
  in v2 and must not be reintroduced
- **Iesha view: static "Scoring Math" card at top + per-card expansion** — no inline tooltips;
  the expansion drawer is the only score-detail mechanism
- **`recent_signals` may be null** — null guards required; verified locations: lines 489, 548,
  554, 639, 642, 786, 787, 820
- **Persona auto-populates chips** (Maya → SMB) — chip and persona share single source of
  truth; do not introduce divergent state between chip selection and persona selection
- **localStorage key must stay `'hpb-v2-theme'`** — independent of v1's `'hpb-theme'`; sharing
  breaks v1's theme isolation
- **Hot threshold = composite ≥ 80** — applied consistently across all personas; v2 removed
  v1's Stacy-specific 70 threshold
- **All personas render through unified `.card` component** — do not reintroduce divergent
  per-persona view shapes; persona deltas live in summary stats, chips, and Iesha's expansion
  only
