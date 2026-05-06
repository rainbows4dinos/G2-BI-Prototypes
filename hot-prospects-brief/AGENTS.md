# Hot Prospects Brief (v1)

## Overview

Hot Prospects Brief is a single-file React-via-CDN prototype built for SDRs at G2. It surfaces
high-intent companies with ICP-scored fit, answering the core question: "Tell me who to call today
and why." The app ranks ~50 mock companies by a composite score (60% buyer intent + 40% ICP fit)
and presents them across 5 persona-specific views tailored to different sales roles and workflows.

Each persona sees the same underlying data through a different lens: Francisco gets a ranked feed,
Stacy gets industry-by-size segmentation, Iesha gets score explainability, Alex gets a
high-conviction shortlist of 3, and Maya gets time-sensitive SMB outreach sorted by recency. The
prototype runs entirely in the browser with no build step, no server, and no API integration.
All data is mocked; 4 fixture companies with predictable scores exist specifically for QA.

Dark mode is supported via a localStorage-persisted theme toggle. The Elevate Design System token
set governs all colors, spacing, and typography, keeping the prototype visually consistent with
G2's production design language.

## Tech Stack

Single-file React 18 via CDN (React + ReactDOM + Babel Standalone). No build step, no bundler,
no npm. Styling uses Elevate Design System tokens (25 base CSS custom properties + 4
dark-mode-specific additions) and Material Icons via Google Fonts CDN. Runs as static HTML in any
modern browser; no server, no API integration, no persistence beyond localStorage for theme
preference (`'hpb-theme'`).

## File Structure

`index.html` (~1003 lines) — single-file prototype organized top-to-bottom:

- **Dark mode bootstrap** — reads `localStorage['hpb-theme']`, sets `data-theme` attribute on
  `<html>` before first paint to prevent flash (lines 12–30)
- **CSS tokens** — Elevate light/dark theme definitions; 25 base tokens in `:root`, 4
  dark-mode-specific tokens in `[data-theme="dark"]` (lines 33–95)
- **Dark mode overrides** — contrast fixes for components that need explicit dark-mode rules,
  scoped under `[data-theme="dark"]` (lines 98–112)
- **Component styles** — shell layout, top bar, sidebar, page header, persona switcher, filter
  bar, fit badges, score badge, view visibility rules, persona-specific styles (Francisco, Stacy,
  Iesha, Alex, Maya), toast, theme transitions, responsive breakpoints (lines 114–354)
- **Mock data** — 50 companies + 4 test fixtures with predictable scores (line 372)
- **Scoring functions** — `dimensionMatch`, `computeFit`, `computeComposite`, `computeWhyCall`
  (lines 375–449)
- **Helper functions** — `recencyLabel`, `latestSignal`, `weekSignalCount` (lines 463–481)
- **React components** — `FitBadge`, `TopBar`, `Sidebar`, `FilterBar`, `FranciscoView`,
  `StacyView`, `IeshaView`, `AlexView`, `MayaRow`, `MayaView` (lines 484–897)
- **App root** — state management, enrichment pipeline, filtering, sorting, persona switching,
  theme toggle (lines 900–996)
- **Root render** — `ReactDOM.createRoot(document.getElementById('app')).render(<App />)`
  (line 998)

## Personas

| Name | View | Sort/Group | Key Feature |
|---|---|---|---|
| Francisco | List view (`.fr-row`) | Composite desc, then intent desc | Top-line hot prospect feed; shows company, score, fit badge, signal type, "why call" rationale |
| Stacy | Segment grid (`.st-segment`) | Industry × size buckets, sorted by avg composite | Cross-cut segmentation for ABM; heat-coded (hot ≥70, warm 40–69, cool <40); expandable company lists |
| Iesha | Bar charts (`.ie-section`) | Per-company score breakdown + aggregate overview | Score explainability; shows intent contribution (×0.6), fit contribution (×0.4), dimension match/no-match/unknown |
| Alex | Top-3 hero cards (`.ax-hero`) | Composite desc, capped at 3 | High-conviction call list; large score display, action buttons (Add to CRM, Email, Slack, Mark contacted, Skip), toast feedback |
| Maya | Recency-sorted rows (`.my-row`) | SMB-only (employees < 200 or null), split "This Week" + "Other SMB Activity" | Time-sensitive outreach; shows recency label (e.g., "2 days ago"), week signal count |

Persona state lives in a single `useState` in App (line 971). The persona switcher select and the
chip display both read from this state; there's no separate chip state. Switching persona updates
`data-persona` on the shell element, which CSS uses to show/hide the correct view.

## Scoring

Function signatures (declarations only; see `index.html` for bodies):

- **`dimensionMatch(value, rule)`** (line 375) — evaluates one ICP dimension against a company
  field value; returns `'match' | 'no-match' | 'unknown'`

- **`computeFit(company, icp)`** (line 394) — aggregates ICP fit across all dimensions;
  returns `{ tier, score, matches, missingDims }`
  - Tiers: Strong (≥3 of 4 dims match, score=100), Partial (1–2 match, score=50–75),
    Weak (0 match, score=25), Unknown (all dims null, score=null)

- **`computeComposite(company, icp)`** (line 420) — combines intent and fit into a single
  0–100 score; returns `{ composite, intent, fit, formula }`
  - Formula: `composite = intent × 0.6 + fit × 0.4` when fit is known
  - Fallback: `composite = intent` when fit is null (no penalty for missing firmographic data)

- **`computeWhyCall(company, mockNow)`** (line 434) — generates a short rationale string
  templated from `recent_signals` within a 7-day window relative to `MOCK_NOW`

Hot threshold: composite ≥ 80 (global). Stacy view uses 70 as its own internal heat threshold,
separate from the global 80. Both values are intentional and should be changed independently.

## Key Constants

- **`MOCK_NOW = '2026-05-04T12:00:00Z'`** (line 361) — fixed reference timestamp for
  deterministic "this week" calculations; DO NOT change (fixture-driven scores depend on it)
- **`MOCK_DATA`** (line 372) — 50 companies + 4 test fixtures:
  - `Fixture-StrongFit-4of4` — all 4 ICP dimensions match, score=100
  - `Fixture-PartialFit-2of4` — 2 of 4 dimensions match, score=50–75
  - `Fixture-UnknownFit-nulls` — all dimensions null, score=null
  - `Fixture-WeakFit-0of4` — 0 dimensions match, score=25
- **`SIGNAL_META`** (line 455) — signal type metadata (icon + label) for: profile, pricing,
  compare, category, competitors
- **localStorage key** (line 15) — `'hpb-theme'` (stores `'light'` | `'dark'`)
- **Hot threshold** — 80 (global composite cutoff); Stacy view internal threshold — 70
- **25 Elevate CSS tokens** (lines 34–65):
  `--primary`, `--primary-hover`, `--primary-10`, `--primary-20`, `--neutral-5`, `--card`,
  `--border-light`, `--border-med`, `--border-brand`, `--text`, `--text-sec`, `--text-ter`,
  `--text-pri`, `--r-sm`, `--r-md`, `--r-pill`, `--sh-card`, `--sh-dd`, `--font`, `--surface`,
  `--error-bg`, `--error-text`, `--error-border`, `--success-bg`, `--success-text`
- **4 dark-mode-specific tokens** (lines 61–64, 91–94):
  `--toast-bg`, `--toast-text`, `--st-overlay-hot`, `--st-overlay-warm`

## How to Extend

### Add a new persona view

1. Add `<option value="new-persona">New Persona Name</option>` to the persona-switcher select
   in App (line 974)
2. Create `NewPersonaView({ companies })` component following the pattern of `FranciscoView`
   (line 578) or `StacyView` (line 613)
3. Add CSS visibility rules near line 221:
   `.view-new-persona { display: none; }` and
   `[data-persona="new-persona"] .view-new-persona { display: block; }`
4. Render in App (line 985): `<NewPersonaView companies={sorted} />`
   Use `companies={filtered}` if the persona needs unfiltered data (like Maya does)
5. Update page header subtitle (line 969) if the persona changes display semantics

### Tweak the hot threshold

1. Locate Stacy's internal threshold (line 645): `seg.avgComposite >= 70 ? 'hot' : ...`
2. Update all Stacy references: heat indicator (line 645), heat label (line 646), overlay
   class (line 647)
3. For the global threshold (80), update Alex's `companies.slice(0, 3)` logic and any
   heat-coding that references it
4. The two thresholds (Stacy's 70, global 80) are intentionally separate; change them
   independently

### Add a new ICP dimension

1. Extend `DEFAULT_ICP` (line 363): add new dimension key with rule shape (numeric range,
   string array, etc.)
2. Update `dimensionMatch` (line 375): add a case for the new rule shape
3. Update `computeFit` (line 394): add dimension to `dims` array (line 395), update
   match-count logic (line 410)
4. Update Iesha's `dimNames` array (line 698) to include the new dimension name
5. Update Iesha's "Current ICP" display (line 773) to render the new dimension

### Add a new toast notification

1. Locate toast state in `AlexView` (line 785): `const [toast, setToast] = React.useState(null)`
2. Add to actions array (line 794):
   `{ label: 'Action Name', icon: 'material_icon_name', msg: 'Toast message text' }`
3. Toast is already wired via `showToast(a.msg)` in the onClick handler; no further changes
   needed

### Override dark-mode contrast for a new component

1. Find the dark-mode overrides block (lines 98–112): `[data-theme="dark"] .selector { ... }`
2. Add a new rule using dedicated tokens — NOT `--text` (load-bearing in 8+ places)
3. Prefer `--primary-hover`, `--surface`, `--text-sec`, `--text-ter` for dark-mode contrast
4. Test both themes via the toggle button (line 503) and verify WCAG AA contrast passes

## Invariants

- **`MOCK_NOW` is fixed** — fixture-driven scores depend on it; changing this breaks all
  "this week" calculations and test fixture expected values
- **`--text` CSS token is load-bearing in 8+ places** — don't override for theme-specific
  overlays; use dedicated tokens (`--toast-bg`, `--toast-text`, `--st-overlay-hot`,
  `--st-overlay-warm`) instead
- **`view-{persona}` CSS rules MUST be scoped under `[data-persona="X"]`** (line 222) —
  specificity bug fix from initial build; ensures only one persona view renders at a time
- **Persona switcher and chip state share a single source of truth** (line 971) — chip
  auto-populates from persona state; no separate chip state needed
- **Composite formula is hardcoded 60/40** (line 428) — no UI to change weights; update
  `computeComposite` and document in Iesha's view (line 770) if changed
- **Hot threshold (80) is used in 3 places** — Stacy heat indicator (line 645 uses 70
  separately), Alex top-3 selection, and visual heat-coding; change all together
