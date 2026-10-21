# Project Pulse dashboard implementation plan

## Goal and definition of done

Build a small, static **Project Pulse** dashboard that lets contributors scan Mona's active projects. The first view must be a polished dashboard—not a directory listing—and must present multiple project cards with a project name, owner, status, recent activity, priority or risk signal, and a contributor-friendly summary where practical.

The completed work consists of these four files:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

The dashboard is done when it is accessible and responsive, visibly renders the data-driven project cards, and the **Run Project Pulse Dashboard** VS Code launch configuration serves `app/index.html` at port 5500.

## Roles and ownership

The Orchestrator owns sequencing, non-overlapping assignments, integration review, and final reporting. The Planner has completed repository discovery and supplies this plan. Neither role should implement the dashboard files during this work.

### Designer responsibilities

Assign the Designer the visual and accessibility direction and the styling implementation in **`app/styles.css`**:

- Establish a card-based dashboard composition: a clear title/header, concise supporting copy or project count, and a responsive project-card grid.
- Specify an information order that supports scanning: project name and priority first, then owner and status, followed by recent activity and optional summary.
- Create a readable visual system using semantic color tokens, strong text/background contrast, status badges that do not rely on color alone, priority emphasis, consistent spacing, typography, `border-radius`, and `box-shadow`.
- Implement responsive CSS with the required `.dashboard` and `.project-card` selectors. Cards should remain readable at narrow widths rather than depend on horizontal scrolling.
- Account for keyboard and assistive-technology use in the design: visible focus styles for any interactive elements, adequate contrast, and no information conveyed exclusively by color.
- Report the CSS hooks and data fields that the HTML implementation must use. The Designer must not edit the HTML, JSON data, or launch configuration unless the Orchestrator explicitly reassigns them.

### Coder responsibilities

Assign the Coder the functional implementation in **`app/index.html`**, **`app/project-data.json`**, and **`.vscode/launch.json`**:

- Create the data contract and realistic, deterministic sample records in `app/project-data.json`.
- Build semantic, accessible HTML that loads the JSON, renders each record as a visible `project-card`, and handles fetch, HTTP, parse, and malformed-data failures with an explicit on-page error instead of silently rendering nothing.
- Add a loading message while project data is requested, then replace it with cards. Escape or otherwise safely insert data values; use DOM node APIs rather than injecting raw JSON strings as HTML.
- Link `styles.css`, reference/load `project-data.json`, and use the Designer's agreed hooks and status/priority class names.
- Create the runnable launch configuration as strict JSON (no comments), with the exact command, working directory, name, and browser target described below.
- Run the implementation checks in this plan and report results and any remaining browser-only risks. The Coder must not alter design-only CSS without a Designer-approved adjustment.

## File-level implementation requirements

| File | Owner | Required implementation |
| --- | --- | --- |
| `app/project-data.json` | Coder | Valid JSON object with a top-level `projects` array. Include multiple objects. Every object must have non-empty `name`, `owner`, `status`, `recentActivity`, and `priority` fields; include a short summary field if it is used by the cards. Use consistent status and priority vocabulary so CSS mapping is deterministic. |
| `app/styles.css` | Designer | Styles the complete dashboard. Include `.dashboard` and `.project-card`; use a responsive grid, readable typography, badges/labels, `border-radius`, `box-shadow`, color-plus-text status/priority treatment, and responsive rules. |
| `app/index.html` | Coder | A complete HTML document whose page/title content says exactly `Project Pulse`. Link `styles.css`, fetch or otherwise visibly reference `project-data.json`, and dynamically create a `project-card` for every valid project. Cards visibly show name, owner, `status`, `recentActivity`, and `priority`; semantic landmarks, headings, labels, and live loading/error messages support accessibility. |
| `.vscode/launch.json` | Coder | Strict JSON with no comments and a configuration named exactly `Run Project Pulse Dashboard`. Serve from `cwd: "${workspaceFolder}/app"` using `python3 -m http.server 5500`. Include `serverReadyAction` that targets `http://localhost:%s/index.html`, so the frontend opens directly rather than a server directory listing. |

## Data and rendering contract

1. The Coder establishes `projects` as an array before HTML rendering begins.
2. A project card consumes `name`, `owner`, `status`, `recentActivity`, and `priority` exactly as named in the JSON; use the same field names visibly or in predictable JavaScript property access so the workflow's required-content checks can find them.
3. `status` and `priority` values are rendered as text in addition to any color class. For example, a card can use a badge whose text says `Status: On track` and a label whose text says `Priority: High`.
4. The renderer validates that `projects` is an array and required values are strings before creating a card. Invalid input should result in a concise, user-visible error state (or explicitly skip the bad record and report that state) without breaking the entire page.
5. Relative `fetch("project-data.json")` requires HTTP serving. Opening the file directly is not a valid preview method; the launch configuration is part of the functional dependency.

## Execution phases, dependencies, and parallel decisions

### Phase 1 — establish the design and data contract (sequential)

1. The Orchestrator gives the Designer the dashboard brief and ownership of `app/styles.css`.
2. The Designer returns/implements the component hierarchy, responsive behavior, field order, status/priority vocabulary, CSS hook names, and accessibility treatment.
3. The Orchestrator passes those decisions to the Coder before the Coder finalizes HTML.

This is **sequential**, not parallel, because the HTML needs stable card markup and class names to correctly consume the visual system. It prevents the Coder from creating markup that conflicts with the intended hierarchy or requires CSS rework.

### Phase 2 — implement independent artifacts (parallel where safe)

After the data contract and design hooks are agreed, the following work may run **in parallel** because the file scopes do not overlap:

- **Designer:** implement `app/styles.css`.
- **Coder:** create `app/project-data.json` and `.vscode/launch.json`.

The CSS depends on the agreed markup hooks, while the JSON and launch configuration do not depend on CSS. The Coder should share final field values and status/priority class mapping with the Designer before integration.

### Phase 3 — implement and integrate the page (sequential)

The Coder implements `app/index.html` **after** the JSON schema and Designer's hooks are available. This is **sequential** because the renderer must use the final field names, and the DOM must carry the selectors/classes the stylesheet expects.

The Coder then verifies that:

- every data record creates one visible `.project-card`;
- the page contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, and `priority`;
- status and priority text remain understandable without CSS.

### Phase 4 — integration and acceptance (sequential)

The Orchestrator reviews all four files together after both agents report completion. This phase cannot be parallelized: JSON validity, the renderer, CSS, and the launch server must work as one path. Route fixes back to the owner of the affected file; if a markup/CSS contract must change, have Designer and Coder agree on it before either edits.

## Launch configuration shape

The Coder should use a VS Code launch entry compatible with Python debugging, for example a `python` configuration whose `request` is `launch`, with:

- `name`: `Run Project Pulse Dashboard`
- `module`: `http.server`
- `args`: `["5500"]`
- `cwd`: `${workspaceFolder}/app`
- `serverReadyAction.pattern`: a pattern that captures port `5500` from the Python server output
- `serverReadyAction.uriFormat`: `http://localhost:%s/index.html`

The resulting invocation must be exactly equivalent to `python3 -m http.server 5500` from the `app` directory. Do not use JSON-with-comments syntax; the repository workflow parses this file with Python's strict JSON parser.

## Validation and acceptance checklist

Run these checks from the repository root after implementation:

1. Validate both JSON files:

   ```bash
   python3 -m json.tool app/project-data.json >/dev/null
   python3 -m json.tool .vscode/launch.json >/dev/null
   ```

2. Confirm the workflow-required content is present:

   ```bash
   grep -qi 'Project Pulse' app/index.html
   grep -qi 'styles.css' app/index.html
   grep -qi 'project-data.json' app/index.html
   grep -qi 'project-card' app/index.html
   grep -qi 'status' app/index.html
   grep -qi 'recentActivity' app/index.html
   grep -qi 'priority' app/index.html
   grep -q '\.dashboard' app/styles.css
   grep -q '\.project-card' app/styles.css
   grep -q 'border-radius' app/styles.css
   grep -q 'box-shadow' app/styles.css
   grep -q '"projects"' app/project-data.json
   grep -q 'Run Project Pulse Dashboard' .vscode/launch.json
   grep -q 'index.html' .vscode/launch.json
   ```

3. Compare the result to `.github/workflows/3-step.yml`, whose required-content checks require the three app files and launch file; the named HTML references and card fields; the `.dashboard`, `.project-card`, rounded, and shadow styling; JSON parseability; and the launch name/target.
4. In VS Code, select **Run Project Pulse Dashboard** in **Run and Debug** and start it. Confirm the server runs from `app/`, the browser opens `http://localhost:5500/index.html` (through the configured `http://localhost:%s/index.html` target), and no directory listing appears.
5. Browser acceptance: verify several cards are visible, every required field is readable, status and priority are textually identifiable, the layout works at a narrow viewport, and loading/error behavior is understandable. Stop the preview server after checking.

## Handoff

The Orchestrator reports each agent's files, validation results, and any unresolved risk (such as browser-only compatibility) to the learner. No agent stages, commits, or pushes; those Git actions remain under learner control.
