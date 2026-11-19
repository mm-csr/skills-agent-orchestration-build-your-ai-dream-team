# Project Pulse dashboard: implementation plan

Plan produced by the Planner agent.

## Summary

Mona's **Project Pulse** dashboard is a small static frontend. It reads project records from `app/project-data.json` and shows each one as a card with a status badge, a priority marker, the owner and recent activity. A VS Code launch configuration named **Run Project Pulse Dashboard** serves the `app/` folder and opens `index.html`, so the learner sees the dashboard rather than a directory listing.

Constraints from the repo:

- `app/` and `.vscode/launch.json` don't exist yet. `.vscode/tasks.json` exists and must not be changed.
- The Orchestrator and Planner have no file-writing tools, so a Coder (or the learner via Copilot CLI) saves files.
- The Codespace image includes `python3` but not the Python VS Code extension, so the launch config uses the built-in `node-terminal` debug type.
- The Step 3 grader looks for exact strings:
  - `app/index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
  - `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
  - `app/project-data.json`: `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`; must parse as JSON
  - `.vscode/launch.json`: must parse as JSON and contain `Run Project Pulse Dashboard` and `index.html`

## Shared markup and CSS contract

Coder writes the HTML while Designer writes the CSS at the same time, so both must use the same class names and data values.

| Hook | Element | Markup owner | Styling owner |
| --- | --- | --- | --- |
| `.dashboard` | `<main class="dashboard">` wrapper | Coder | Designer |
| `.dashboard-header` | `<header>` with `<h1>Project Pulse</h1>` and subtitle | Coder | Designer |
| `.project-grid` | Container for the cards | Coder | Designer |
| `.project-card` | `<article class="project-card">`, one per project | Coder | Designer |
| `.project-name` | `<h2>` inside the card | Coder | Designer |
| `.project-owner` | Owner line | Coder | Designer |
| `.project-activity` | Recent activity text | Coder | Designer |
| `.status-badge` + `.status-on-track` / `.status-at-risk` / `.status-blocked` / `.status-done` | Status pill | Coder | Designer |
| `.priority` + `.priority-high` / `.priority-medium` / `.priority-low` | Priority marker | Coder | Designer |
| `.dashboard-message` | Loading, empty and error messages (`role="status"`) | Coder | Designer |

Data values:

- `status` is one of `"On track"`, `"At risk"`, `"Blocked"`, `"Done"`.
- `priority` is one of `"High"`, `"Medium"`, `"Low"`.
- Modifier classes are the lowercased value with spaces replaced by hyphens (`"At risk"` becomes `status-at-risk`).
- Badges always show the value as text, not only as color.

## Ordered implementation steps

### Phase 0: Save the plan (sequential)

0. **Coder / learner** saves this plan to `docs/project-pulse-plan.md`.

### Phase 1: Data (sequential)

1. **Coder** creates `app/project-data.json`:
   - Strict JSON with a top-level `"projects"` array of 4 to 6 projects.
   - Each project has `name`, `owner`, `status`, `recentActivity` and `priority` (strings).
   - Use every status and priority value at least once.
   - Include one project with long text to test wrapping.

### Phase 2: Markup and logic, styling, launch config (parallel)

2a. **Coder** creates `app/index.html`:
   - `<!DOCTYPE html>`, `lang="en"`, charset and viewport meta, `<title>Project Pulse</title>`, `<link rel="stylesheet" href="styles.css">`.
   - Static shell: `.dashboard`, header, `.project-grid`, `.dashboard-message`.
   - Inline script that fetches `project-data.json`, checks `response.ok` and that `projects` is an array, and renders one `article.project-card` per project (name, owner, status badge, `recentActivity`, `priority`) using `textContent` rather than `innerHTML`.
   - Handles error, empty and `file://` cases (see Edge cases).

2b. **Designer** creates `app/styles.css`:
   - CSS custom properties for color, spacing and radius; readable typography.
   - `.dashboard` container, `.project-grid` as a responsive CSS grid (for example `repeat(auto-fill, minmax(280px, 1fr))`).
   - `.project-card` with `border-radius`, `box-shadow`, padding, subtle hover/focus lift (respect `prefers-reduced-motion`).
   - Status badge colors with WCAG AA contrast, priority treatment, `.dashboard-message` styling.
   - An `@media` breakpoint and visible `:focus-visible` styles.

2c. **Coder** creates `.vscode/launch.json` (strict JSON, no comments):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Project Pulse Dashboard",
      "type": "node-terminal",
      "request": "launch",
      "command": "python3 -m http.server 5500",
      "cwd": "${workspaceFolder}/app",
      "serverReadyAction": {
        "action": "openExternally",
        "pattern": "Serving HTTP on .* port ([0-9]+)",
        "uriFormat": "http://localhost:%s/index.html"
      }
    }
  ]
}
```

### Phase 3: Integration review (sequential)

3. **Orchestrator** checks that classes in `index.html` match selectors in `styles.css` and that data values match modifier classes. Fixes go to the file owner (Coder for HTML/JSON, Designer for CSS), one agent per file at a time.

4. **Designer** (optional) reviews the rendered result for hierarchy, contrast and responsiveness. Any edits stay within `app/styles.css`.

## File assignments

| File | Owner | Phase | Responsibility |
| --- | --- | --- | --- |
| `docs/project-pulse-plan.md` | Coder | 0 | Saved plan |
| `app/project-data.json` | Coder | 1 | Project data |
| `app/index.html` | Coder | 2a | Markup, semantics, render script |
| `app/styles.css` | Designer | 2b, 4 | All styling, `.dashboard` and `.project-card` hooks |
| `.vscode/launch.json` | Coder | 2c | Run configuration |
| `.vscode/tasks.json`, `.devcontainer/*`, `.github/*` | nobody | n/a | Must not be modified |

### Responsibilities

- **Designer:** visual design, color and contrast, layout, responsive behavior, badge and priority treatment, focus and motion styling. Advises on accessible structure (heading order, ARIA) through the Orchestrator. Does not edit `index.html`.
- **Coder:** data, HTML structure, data loading and rendering logic, error handling, and the launch/run setup. Does not edit `styles.css`.

## Dependencies

1. Step 0 (plan saved) comes before implementation.
2. Step 1 (data) finishes before Steps 2a and 2b, because the `status` and `priority` values decide the modifier class names. Step 2c doesn't depend on Step 1.
3. Steps 2a and 2b depend on the shared contract, not on each other's output.
4. Step 3 needs 2a, 2b and 2c done.
5. Step 4 needs Step 3.

## Parallel work decisions

- **Parallel (Phase 2):** Coder on `app/index.html` and `.vscode/launch.json` alongside Designer on `app/styles.css`. File scopes don't overlap and the shared contract removes the data dependency.
- **Sequential:** Phase 0, then 1, then 2, then 3, then 4. Fixes to the same file never run at the same time. If data later gains a new status or priority value, Designer updates the CSS afterwards, not at the same time.
- **Never in parallel:** two agents editing the same file.

## Edge cases

| Case | Behavior |
| --- | --- |
| Opened via `file://` (fetch blocked) | Message in `.dashboard-message` explaining to use "Run Project Pulse Dashboard" or `python3 -m http.server`. |
| Fetch fails or non-OK response | Message naming `project-data.json` and the HTTP status; no blank page. |
| Invalid JSON, or `projects` missing/not an array | Error message; details logged with `console.error`. |
| Empty `projects` array | "No projects yet" empty state. |
| Missing field | Placeholder such as "Unknown owner"; never render "undefined". |
| Unknown status or priority | Neutral fallback class (for example `.status-unknown`) with the text label still shown. |
| Long names or activity text | Wrap with `overflow-wrap: anywhere`; layout stays intact. |
| HTML in data values | Render with `textContent` only. |
| Port 5500 in use | `http.server` fails and no browser opens; stop the earlier server or change the port. |
| Ready pattern not matched due to buffering | Fallback: `python3 -u -m http.server 5500`. |
| Narrow screens, keyboard users | Single column, visible focus, no meaning by color alone. |

## Validation expectations

Static checks:

- `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` pass.
- `grep` finds the grader strings listed in the Summary, plus an `@media` rule in `app/styles.css`, and `${workspaceFolder}/app` and `http://localhost:%s/index.html` in `.vscode/launch.json`.
- Every project has all five keys, for example:
  `python3 -c "import json;d=json.load(open('app/project-data.json'));assert all({'name','owner','status','recentActivity','priority'}<=p.keys() for p in d['projects'])"`
- Every class used in `index.html` has a selector in `styles.css`, and every status and priority value maps to a defined modifier class.
- `git status` shows only the assigned files; `.vscode/tasks.json` is unchanged.

Runtime checks:

- `cd app && python3 -m http.server 5500`, then `curl -s localhost:5500/index.html` and `curl -s localhost:5500/project-data.json` return the page and JSON.
- **Run Project Pulse Dashboard** in VS Code opens `http://localhost:<port>/index.html` and shows the cards, not a directory listing.

Visual and accessibility checks (Designer):

- Cards, badges and priority render at desktop width and at about 375px.
- Text contrast meets AA and keyboard focus is visible.
- One `<h1>`, with an `<h2>` on each card.

Nobody stages, commits or pushes; the learner does that through Copilot CLI.

## Open questions

1. Is the proposed status and priority wording acceptable, or does Mona have existing terms? Changing it after Phase 2 requires sequential updates to data and CSS.
2. Include an optional `summary` field in the data (recommended) or leave it out?
3. Keep the script inline, or is a separate `app/app.js` wanted?
4. Is `openExternally` (local browser via Codespaces port forwarding) preferred over an in-editor preview?
5. Confirm nothing else in the Codespace uses port 5500.
