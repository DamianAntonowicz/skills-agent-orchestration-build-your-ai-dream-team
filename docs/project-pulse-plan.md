# Project Pulse — Implementation Plan

## 1. Overview

**Project Pulse** is a lightweight, static, client-side project-status dashboard for Mona's team. It is a single-page HTML/CSS/JS app that reads a local JSON file (`app/project-data.json`) and renders a visual dashboard of projects, each shown as a card with:

- Project **name**
- **Owner**
- **Status** (e.g. On Track, At Risk, Blocked, Done)
- **Recent activity** (short free-text update)
- **Priority** / risk level (e.g. High, Medium, Low)

The dashboard gives contributors a fast, at-a-glance view of what's active, who owns it, and what needs attention — no backend, build step, or external dependencies required. It must run and be debuggable from VS Code via a **Run Project Pulse Dashboard** launch configuration that serves `app/` over HTTP (not `file://`, since `fetch()` of local JSON is blocked by CORS under the `file://` protocol) and opens `index.html` directly (not a directory listing).

---

## 2. File Assignments

| File | Owner | Responsibility |
|---|---|---|
| `app/index.html` | **Coder** (primary), Designer (structural/semantic input) | Coder owns the final HTML file: document structure, semantic elements, `<title>Project Pulse</title>`/visible "Project Pulse" heading, `<link>` to `styles.css`, script that fetches `project-data.json`, JS rendering logic that generates one `.project-card` element per project, and wiring for status/priority/recentActivity display. Designer supplies the required markup/class/ARIA structure (e.g. `.dashboard` wrapper, `.project-card` card, status-badge element, semantic roles/landmarks) that Coder implements, so the DOM is stylable and accessible from the start. |
| `app/styles.css` | **Designer** (full ownership) | All visual design: color palette, typography, spacing scale, `.dashboard` grid/layout selector, `.project-card` styling (rounded corners via `border-radius`, elevation via `box-shadow`), status-badge color coding, priority treatment, responsive breakpoints, hover/focus states, and accessibility-related styling (visible focus outlines, contrast-safe colors). |
| `app/project-data.json` | **Coder** (full ownership) | Defines and populates the data schema: top-level `"projects"` array; each project object has `name`, `owner`, `status`, `recentActivity`, `priority`. Coder provides realistic sample data (minimum 4–6 projects covering different status/priority values) so Designer has real content to style against. |
| `.vscode/launch.json` | **Coder** (full ownership) | Strict JSON, no comments. One configuration named **"Run Project Pulse Dashboard"** that: sets `cwd` to `${workspaceFolder}/app`, runs `python3 -m http.server 5500`, and uses a `serverReadyAction` to open `http://localhost:%s/index.html` in the browser (so learners see the dashboard frontend, not a directory listing). |

**Ownership rule for `index.html`:** Coder writes/edits the file. Designer does not edit `index.html` directly but must deliver (in writing, before Coder finalizes markup) the exact structural contract: wrapper class (`.dashboard`), card class (`project-card`), and any sub-element classes/ARIA roles needed for status badges and priority indicators. If Designer needs a structural change after Coder's first pass, Designer requests it from Coder rather than editing the file, to avoid merge conflicts between agents.

---

## 3. Designer Responsibilities

- **Visual design system**: define a small palette (background, surface/card, text, accent, and 3 status colors e.g. green/amber/red for On Track / At Risk / Blocked) and a typographic scale (heading, card title, body, meta text).
- **Spacing & layout**: consistent spacing scale (e.g. 4/8/16/24px rhythm), card padding, gaps in the card grid.
- **Responsive layout**: `.dashboard` as a responsive grid/flex container — multi-column on desktop, single column on narrow viewports (define at least one breakpoint, e.g. ~600px).
- **Information hierarchy**: project name most prominent, owner/status/priority secondary but scannable, recent activity as supporting text; status and priority rendered as visually distinct badges/chips (not just plain text) so scanning dozens of cards is fast.
- **Accessibility**:
  - Sufficient color contrast for text and status badges (WCAG AA, 4.5:1 for body text).
  - Don't rely on color alone for status/priority — pair color with text label.
  - Visible `:focus-visible` states for any interactive elements.
  - Semantic structure guidance to Coder (landmarks, heading levels, list semantics for the card grid if appropriate).
  - Keyboard navigability: if any interactive controls are added (filters, links), they must be reachable and operable via keyboard.
- **Interaction/hover states**: subtle hover elevation/transform on `.project-card`, hover/focus treatment on any future interactive affordances.
- **Deterministic CSS hooks** (required by the exercise): must include a `.dashboard` selector and a `.project-card` selector in `app/styles.css`, and the card styling must include `border-radius` and `box-shadow` to look "polished," not a bare HTML page.
- Deliver the structural/class contract to Coder early (see Dependencies) so both can work in parallel.

---

## 4. Coder Responsibilities

- **HTML structure** (`app/index.html`):
  - Visible page title/heading with the exact text **"Project Pulse"**.
  - `<link rel="stylesheet" href="styles.css">`.
  - A container element with class `dashboard` (per Designer's contract) that JS will populate.
  - A `<script>` (inline or `app/app.js`-equivalent, kept inline unless Orchestrator assigns a separate JS file) that:
    - `fetch('project-data.json')`, parses JSON, and reads the top-level `projects` array.
    - Renders one card per project with class `project-card`.
    - Displays `name`, `owner`, `status`, `recentActivity`, and `priority` for each project, using the classes/markup Designer specified for badges.
    - Includes error handling: if fetch fails (e.g. wrong protocol, missing file, malformed JSON), render a visible, non-cryptic error message in the DOM and `console.error` the underlying error — do not fail silently or leave a blank page.
- **Data schema design** (`app/project-data.json`):
  - Top-level shape: `{ "projects": [ { "name": string, "owner": string, "status": string, "recentActivity": string, "priority": string }, ... ] }`.
  - Must parse as valid JSON (`python3 -m json.tool` clean).
  - Provide varied sample data (different statuses/priorities) so Designer's badge styling and Coder's rendering logic are both exercised.
- **`.vscode/launch.json`**:
  - Strict JSON, no comments.
  - Configuration name exactly: `"Run Project Pulse Dashboard"`.
  - `cwd`: `${workspaceFolder}/app`.
  - Command: `python3 -m http.server 5500`.
  - `serverReadyAction` pattern that opens `http://localhost:%s/index.html` once the server is ready, pointing at port 5500 so it opens the dashboard file directly instead of a directory index.
- **Error handling & testability**: keep rendering logic as small, pure, testable functions where practical (e.g. a function that takes a project object and returns a DOM node/HTML string); avoid global mutable state beyond what's needed to hold fetched data; validate the JSON shape defensively (e.g. guard against a missing `projects` key) and surface a clear error instead of throwing an uncaught exception.
- Validate own output before reporting completion: confirm `index.html` loads `styles.css` and `project-data.json`, confirm JSON parses, confirm launch config JSON is valid and opens the right URL.

---

## 5. Dependencies

1. **Data schema before data-dependent styling**: Coder's `project-data.json` schema (field names: `name`, `owner`, `status`, `recentActivity`, `priority`) must be finalized before Designer finalizes status-badge styling and card information hierarchy in `styles.css`, since the badge/chip design depends on knowing the exact status/priority vocabulary and field set.
2. **Structural contract before final CSS polish**: Designer must hand off the required class/structure contract (`.dashboard`, `.project-card`, badge sub-classes, ARIA notes) to Coder *before* Coder finalizes the HTML/JS rendering function, OR Designer can specify this contract upfront from the brief (it is already implied by the brief/validation requirements) so both can start in parallel. Final CSS polish (exact spacing, responsive tuning) must wait until Coder's HTML skeleton/class names are stable in the repo, since Designer is styling real rendered markup, not a mockup.
3. **`index.html` + server behavior before `launch.json`**: `.vscode/launch.json` depends on knowing how the app is served — specifically that `index.html` lives in `app/` and must be reachable at `http://localhost:5500/index.html` when served from `cwd = ${workspaceFolder}/app`. This file should be created only after Coder confirms `index.html` exists at `app/index.html` (even a minimal skeleton is enough to unblock this).
4. **Rendering logic depends on the data schema**: Coder's JS fetch/render code depends on the final `project-data.json` shape being decided (even if sample data content is still placeholder, the *shape* must be fixed first).
5. **No dependency between `styles.css` content and `launch.json`**: these two files are fully independent of each other and can be built in either order or in parallel.

---

## 6. Parallel Work Decisions

**Can run in parallel (no shared files, no blocking information gaps):**
- Designer drafting the visual design system (palette, type scale, spacing tokens, responsive breakpoints) **while** Coder drafts the `project-data.json` schema and sample data.
- Designer specifying the `.dashboard` / `.project-card` / badge class contract **while** Coder scaffolds the basic `index.html` skeleton and fetch/render JS logic (as long as Designer's contract is shared early, e.g. in the plan or a short handoff note, so Coder doesn't need to guess class names twice).
- Coder building `.vscode/launch.json` **while** Designer writes `styles.css`, since these touch unrelated files and have no data dependency on each other.

**Must run sequentially:**
- Final `styles.css` visual polish (exact selectors matching real markup, responsive fine-tuning) must wait until Coder's `index.html` class names/structure are stable and committed — Designer should not style against a moving target.
- `.vscode/launch.json` finalization should wait until `app/index.html` physically exists at the expected path, so the launch config's served URL (`http://localhost:%s/index.html`) is verified against a real file rather than assumed.
- Any change to the `project-data.json` field names after Designer has built badge-specific CSS must be re-synced with Designer before being merged, to avoid silent mismatches (e.g. renaming `priority` would break a Designer-authored `.priority-high` selector).

---

## 7. Validation Expectations

**Automated / structural checks:**
- `app/project-data.json` parses as valid JSON (e.g. `python3 -m json.tool app/project-data.json`) and contains a top-level `"projects"` array where every entry has `name`, `owner`, `status`, `recentActivity`, and `priority`.
- `.vscode/launch.json` parses as valid JSON with no comments, contains a configuration literally named `"Run Project Pulse Dashboard"`, uses `cwd: ${workspaceFolder}/app`, runs `python3 -m http.server 5500`, and has a `serverReadyAction` opening `http://localhost:%s/index.html`.
- `app/index.html` contains the visible text "Project Pulse", a `<link>` to `styles.css`, a reference to `project-data.json`, and renders elements with class `project-card`.
- `app/styles.css` contains a `.dashboard` selector and a `.project-card` selector that includes `border-radius` and `box-shadow`.

**Functional / manual QA checklist:**
- [ ] In VS Code, open **Run and Debug**, select **Run Project Pulse Dashboard**, press play.
- [ ] Browser opens `http://localhost:5500/index.html` and shows the Project Pulse dashboard — **not** a directory listing.
- [ ] No `file://` CORS/fetch errors; `project-data.json` loads successfully over HTTP.
- [ ] All projects from `project-data.json` render as cards, each showing name, owner, status, recent activity, and priority.
- [ ] Open the browser DevTools console — zero errors/warnings during load and render.
- [ ] Visual check: cards have rounded corners and shadow (not a bare unstyled page); status/priority are visually distinguished (not plain text only).
- [ ] Resize the browser to a narrow width (~375–600px) and confirm the layout reflows to a single column without horizontal scrolling or overlapping text.
- [ ] Tab through the page with keyboard only; confirm focus is visible and logically ordered (relevant if any interactive elements exist).
- [ ] Temporarily rename/break `project-data.json` (or point fetch at a missing file) and confirm the app shows a visible, readable error state instead of a blank page or silent failure, then restore the file.
- [ ] Stop the preview server after testing (per exercise step instructions).

---

## Open Questions / Risks

- The brief doesn't specify whether JS lives inline in `index.html` or in a separate `app/script.js`. This plan assumes inline `<script>` in `index.html` for simplicity (fewer files, matches the brief's 3-file app listing) unless the Orchestrator explicitly assigns a separate JS file.
- Exact status/priority vocabulary (e.g. "On Track" vs "Active", "High" vs "Critical") is not specified in the brief — Coder should pick a small, consistent, finite set and document it so Designer can map it to badge colors without guessing.
- `python3` must be available in the dev container for the launch command to work; this matches the repo's existing devcontainer/validation patterns (already assumed elsewhere in the repo's scripts), so no action needed but worth a sanity check during validation.
