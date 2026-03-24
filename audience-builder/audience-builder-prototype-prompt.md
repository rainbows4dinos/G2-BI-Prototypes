# Claude Code Prompt: Audience Builder Prototype

**Figma source:** [FY26 Buyer Intent — Define Your ICP](https://www.figma.com/design/Pg4BXkpVT1IjteUSuV9Rsr/FY26-Buyer-Intent?node-id=2362-78664&t=Oc0YzcKprV1sa5EI-11)

---

Build a working, interactive front-end prototype of the **G2 Audience Builder** wizard flow from the Buyer Intent product. This is a single-page React app with no backend required — all state is managed in-memory.

---

## Tech Stack

- **React** (functional components + hooks)
- **Tailwind CSS** for styling
- Single `index.html` file (or a single `App.jsx` if using Vite) — keep it self-contained

---

## Overall Layout

The app should render a full my.G2 shell with:

- **Top nav bar** (dark): G2 logo on the left, "Quick nav..." search in the center, icon buttons on the right (notifications, help, avatar)
- **Left sidebar** (white, ~130px wide): Logo area at top, nav items including Home, Product Profile, Reviews, Buyer Activity (expanded with sub-items: Your Signals, Notifications, AI SDR, Recent Visit Events), Leads Activity, Lead Form, Lead Emails, Lead Webhooks, CTA Settings, G2 Advertising, Marketing Content, Analytics, Market Intelligence, Integrations, ROI, Account. "View on G2.com" link at the bottom.
- **Main content area**: renders the active screen described below

The left sidebar's "Your Signals" item should always appear selected/active.

---

## Screens & State Machine

The app has **two top-level states**: `NO_ICP` and `ICP_SAVED`. A modal overlay manages the wizard steps.

### State 1: `NO_ICP` — Buyer Intent Dashboard (default)

**Breadcrumb:** Buyer Activity > Buyer Intent > Intent Signals

**Page title:** Buyer Intent (with a small ⓘ icon)

**Tabs:** Intent Signals (active, underlined in purple) | Churn Risks | Settings

**Action bar (below tabs):**
- Left side: Purple filled button "Define Your ICP" → clicking this opens the Audience Builder modal at Step 1
- Ghost/outline buttons: "Save Segment", "View Segments"
- Right side (floated right): text links "Manage Integrations", "Manage Notifications" | Purple filled button "+ Create a Notification"

**Filter bar (below action bar):**
Dropdown pill filters: Location ▾ | Firmographics ▾ | Companies ▾ | Product Intent ▾ | Category Intent ▾ | Signal Type ▾
Time range buttons on the right: 7d (active/selected) | 30d | 90d | 1yr | Custom ▾

**Metric cards row (4 cards):**
1. Purple/lavender card — "Recommended Action" badge, heading "Audience Builder", body "Set up your Ideal Customer Profile and get more actionable signals.", link "Get Started →" (clicking this also opens the wizard modal at Step 1)
2. White card — "Companies In Market ⓘ", large number **1,860**, subtext "Researched your categories", link "Download CSV ↓"
3. White card — "Looking at You Directly ⓘ", large number **230**, subtext "Intent toward your product", link "Set Up Notification"
4. White card — "Looking at Competitors ⓘ", large number **156**, subtext "Intent to alternatives in your categories", link "Select Your Top Competitors"

**Table section:**
- Toggle: "Table View" (active, underlined) | "Chart View"
- Table toolbar: "Download CSV ▾" button on left | "Modify Columns ▾" button on right
- Table columns: Company Name | Activity Level ⓘ | Visitor Locations | Last Activity | Bombora Signals | Compared You To ⓘ | Total Visitors | Total Signals
- Populate with ~8 static rows of mock data (company names like VFW Post 7686, Echo Global Logistics, Alexandria Capital, LPL Financial, PepsiCo, CIG, Kajabi, Adobe). Activity levels: mostly "Low", one "High" badge (green pill). Locations: US states. Last activity: relative times ("6 days ago", "a day ago", etc.). Total Signals values should be blue hyperlink-style numbers.

---

### The Audience Builder Modal

A centered white modal (~520px wide) with a dimmed backdrop overlay covering the page behind it. Has an X close button in the top-right corner.

**Modal header (always visible):**
- Title: **Audience Builder** (large, bold)
- Subtitle: "Define who receives your intent signals."
- Progress bar: 3 segments, filled up to the current step (purple fill, gray empty). Label below: "Step X/3"

---

#### Step 1/3 — Define Your ICP

**Section: "Define Your ICP"**
Subtext: "Signals matching these criteria will be routed to your integrations."

Form fields:
- **Industries** — full-width multi-select dropdown, placeholder "Select one or more industries.."
- **Geographic Region** — segmented toggle: `All AMER` | `All APAC` | `All EMEA` (toggle-style buttons, one can be active at a time, or none)
- **Country** — half-width multi-select dropdown, placeholder "Select one or more countries.."
- **State/Region** — half-width dropdown, placeholder "Select regions.."
- **Employee Count** — half-width dropdown, placeholder "Select range.."
- **Annual Revenue** — half-width dropdown, placeholder "$ Select range.."

**Section: "Define Exclusions" with "Optional" badge (pill)**
Subtext: "Reduce signal noise before it hits your CRM."

Form fields:
- **Excluded Industries** — full-width multi-select dropdown, placeholder "Select one or more industries.."
- **Excluded Geographic Region** — segmented toggle: `All AMER` | `All APAC` | `All EMEA`

**Footer:** "Back" (ghost/text button, disabled on step 1) | "Next →" (purple filled button, right-aligned)

---

#### Step 2/3 — Apply to Integrations

**Section: "Apply to Integrations"**
Subtext: "Signals are stopped entirely for non-matching accounts."

A list of 4 integration rows, each with:
- Checkbox on the left
- Integration logo (colored icon — use placeholder colored squares/circles with initials if needed)
- Integration name (bold) + category tags below (gray pill tags)
- "● Connected" status badge (green dot + text) on the right

Integrations:
1. **HubSpot** — CRM
2. **6sense Revenue Marketing** — ABM
3. **Snowflake** — ABM · Data Quality · Operations
4. **Slack** — Customer Success · Productivity · Prospecting

All rows start unchecked. Checking a row selects it (checkbox fills, row may get a subtle highlight).

**Footer:** "Back" (ghost button) | "Next →" (purple filled button) — Next should be enabled even if nothing is checked.

---

#### Step 3/3 — Review & Save

**Section: "Review & Save"**

Three labeled display sections (read-only chips/tags):

- **ICP Filters** — renders chips for each value selected in Step 1 (Industries, Geographic Region selections, Countries, Employee Count, Annual Revenue). If nothing was selected, show a light gray empty state placeholder.
- **Anti-ICP Exclusions** — chips for Excluded Industries and Excluded Geographic Region values
- **Applied To** — shows logo + name for each integration checked in Step 2. If none selected, show placeholder text.

**Footer:** "Back" (ghost button) | "Save Audience" (purple filled button)

Clicking "Save Audience" closes the modal and transitions the app to State 2.

---

### State 2: `ICP_SAVED` — Buyer Intent Dashboard (Post-Save)

The modal closes. The main dashboard updates:

**Action bar changes:**
- "Define Your ICP" button is **replaced** with a toggle control: `Only Show ICP` | `Show All Signals` (two-button segmented toggle, "Only Show ICP" active/selected by default with purple fill, "Show All Signals" is the inactive option)
- "Save Segment" and "View Segments" buttons remain

**Metric cards row changes:**
- The purple "Recommended Action / Audience Builder" card is **removed**
- Only the 3 white metric cards remain: Companies In Market | Looking at You Directly | Looking at Competitors

**Toggle behavior:**
- Clicking "Show All Signals" switches the active button to "Show All Signals" and switches back to "Only Show ICP" when clicked again. No actual table data change needed — just the toggle state.

**Everything else** (filter bar, table, tabs) remains the same.

---

## Visual Design Notes

- Primary color: **#7C3AED** (purple) for active states, filled buttons, progress bar fill, active toggle
- Font: Inter or system-ui
- Modal backdrop: `rgba(0,0,0,0.4)`
- Cards: white background, light gray border (`#E5E7EB`), subtle border-radius (~8px), light drop shadow
- Table rows: alternating white, hover state with light gray background
- Segmented toggle buttons: bordered, rounded, the active one gets the purple fill + white text
- Dropdown pills in the filter bar: light gray border, small chevron icon, rounded
- "High" activity badge: green pill (`bg-green-100 text-green-800`)
- "Low" activity level: plain gray text, no badge

---

## Interaction Requirements

1. Clicking "Define Your ICP" button OR "Get Started →" link opens the modal at Step 1
2. Clicking X or the backdrop closes the modal without saving (state stays `NO_ICP`)
3. "Next →" advances the step; "Back" goes to the previous step
4. Step 3 correctly reflects whatever was entered/checked in Steps 1 and 2
5. "Save Audience" transitions to `ICP_SAVED` state and closes modal
6. In `ICP_SAVED`, the "Only Show ICP" / "Show All Signals" toggle is interactive (toggles active state on click)
7. No page reloads — all transitions are in-memory React state

---

## Out of Scope

- No real API calls or authentication
- No routing (single page)
- Tabs (Churn Risks, Settings), Chart View, Download CSV, Modify Columns, and sidebar nav items do not need to be functional
- The filter bar dropdowns do not need to open — just render as static UI