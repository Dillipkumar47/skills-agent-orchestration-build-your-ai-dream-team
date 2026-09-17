# Project Pulse implementation plan

## Goal

Build **Project Pulse**, a lightweight, polished static dashboard for contributors. It will present active projects, their owners and statuses, recent activity, priorities or risks, and concise contributor-friendly summaries.

The dashboard must run from `app/` through a VS Code launch configuration named exactly **Run Project Pulse Dashboard**. The launch flow must open `app/index.html` so contributors see the rendered dashboard rather than an HTTP directory listing.

## Repository constraints

This repository provides agent definitions and documentation, not an existing application framework. The implementation must use browser-native HTML, CSS, JavaScript, and `fetch`; there is no package manager, build system, or test system to introduce or depend on.

Required deliverables:

| File | Purpose |
| --- | --- |
| `app/index.html` | Semantic dashboard page and browser-native data loading/rendering behavior. |
| `app/styles.css` | Responsive, accessible visual design. |
| `app/project-data.json` | Representative project data consumed by the page. |
| `.vscode/launch.json` | VS Code configuration that serves `app/` and opens the dashboard document. |

No agent should stage, commit, or push changes.

## File assignments and responsibilities

The Orchestrator coordinates phases, enforces non-overlapping scopes, manages dependencies, verifies the integrated result, and reports the handoff. It does not implement dashboard files.

| Owner | Assigned file(s) | Responsibility |
| --- | --- | --- |
| Orchestrator | Coordination only; no implementation files | Translate this plan into scoped phases, freeze the shared contract, coordinate parallel work, integrate outputs, verify the complete dashboard, and hand off results. |
| Designer | `app/styles.css` | Create the accessible responsive visual system: information hierarchy, layout, project-card presentation, readable contrast, status and priority/risk treatment, non-color cues, focus treatment, and mobile behavior. |
| Coder | `app/index.html` | Create semantic HTML, load the JSON with `fetch`, safely render project cards through DOM APIs and `textContent`, and provide loading, empty, and error states. |
| Coder | `app/project-data.json` | Supply valid, representative data that conforms to the agreed root schema. |
| Coder | `.vscode/launch.json` | Supply strict, comment-free JSON that starts the deterministic local server and opens `index.html`. |

## Shared UI and data contract

Before implementation, the team must agree on and freeze the following contract:

| Contract area | Required decision |
| --- | --- |
| Primary layout hook | The page includes a `.dashboard` dashboard container. |
| Repeated item hook | Every rendered project uses a `.project-card` element. |
| State hooks | Status and priority/risk receive stable hooks suitable for styling, such as semantic classes or `data-*` attributes derived from normalized values. |
| Data root | JSON has a root object with a `projects` array: `{ "projects": [] }`. |
| Required project fields | Each project has `name`, `owner`, `status`, `recentActivity`, and `priority`. |
| Contributor context | Each project includes a concise `summary`. |
| Safe rendering | Raw data is rendered with DOM creation and `textContent`, never interpolated as HTML. |

The contract lets the Designer style deterministic hooks without modifying Coder-owned files, while allowing Coder to render data predictably.

## Ordered implementation work

1. **Agree the UI and data contract.** The Orchestrator records the `.dashboard`, `.project-card`, status/priority hooks, and the root `{ "projects": [] }` schema with required `name`, `owner`, `status`, `recentActivity`, and `priority` fields plus `summary`. This shared contract is frozen before implementation starts.
2. **Create representative data — Coder, `app/project-data.json`.** Add valid JSON with multiple active projects, realistic concise summaries, owners, status values, recent-activity text, and priority/risk values. Use consistent taxonomy values chosen in step 1.
3. **Create the dashboard document and rendering — Coder, `app/index.html`.** Add semantic document structure with the visible, exact title **Project Pulse**; reference `styles.css`; load `project-data.json` with `fetch`; and render cards with DOM APIs and `textContent`. Each card exposes the required project fields and contract hooks. Include explicit loading, empty, and error states.
4. **Create responsive styling — Designer, `app/styles.css`.** Style `.dashboard` and `.project-card` as a polished contributor dashboard. Include responsive layout behavior, `border-radius`, `box-shadow`, readable hierarchy and contrast, visible status and priority/risk treatments that do not depend on color alone, and keyboard-visible focus styling.
5. **Create the launch configuration — Coder, `.vscode/launch.json`.** Use strict JSON with no comments. Define the configuration named exactly `Run Project Pulse Dashboard`, run `python3 -m http.server 5500`, set `cwd` to `${workspaceFolder}/app`, and configure `serverReadyAction` to open `http://localhost:%s/index.html`.
6. **Integrate and hand off — Orchestrator.** Confirm all outputs honor the frozen contract, run the validation checklist, resolve only affected-file issues through the assigned owner, and report the completed dashboard and any residual risk.

## Dependencies and sequencing

| Work item | Depends on | Why |
| --- | --- | --- |
| Step 1: shared contract | None | It establishes stable data fields and styling hooks. |
| Step 2: project data | Step 1 | Data values and root schema must match the agreed contract. |
| Step 3: HTML/rendering | Step 1; use Step 2 before final data-binding validation | HTML can establish rendering behavior after the contract, but final validation requires actual conforming data. |
| Step 4: CSS | Step 1 | It can proceed independently once the hooks and hierarchy are fixed. |
| Step 5: launch configuration | Decision to serve the `app/` directory | The selected application directory determines `cwd` and the opened document path. |
| Step 6: integrated validation | Steps 2–5 | Validation requires all implementation outputs. |

### Parallel work

After the shared contract is frozen:

- Coder can create `app/project-data.json` and `.vscode/launch.json` in parallel because they have separate files and no dependency on one another beyond the contract and `app/` decision.
- Designer can implement `app/styles.css` in parallel with Coder’s data, HTML, and launch-config work, provided both parties honor the frozen hooks.

### Sequential constraints

- The shared contract is first; no implementation begins before it is agreed.
- Integration and handoff are last, after the four deliverables exist.
- Any remediation is sequential for the affected file: identify the defect, assign it to that file’s owner, apply the correction, then revalidate the affected behavior and integration. This avoids overlapping edits and regression.

## Edge cases and handling expectations

| Edge case | Expected behavior |
| --- | --- |
| Opening through `file://` or another fetch failure | Show a clear error state explaining that the dashboard must be served through the local launch configuration. |
| Missing JSON response or non-OK HTTP response | Catch the condition and display a useful error state instead of leaving a blank dashboard. |
| Invalid JSON or invalid root schema | Catch parsing/validation errors and show an informative error state. Require an object with a `projects` array. |
| Empty `projects` array | Render a contributor-friendly empty state. |
| Incomplete fields or unexpected status/priority values | Use safe fallbacks and neutral state treatment; do not crash rendering. |
| Long summaries, owner names, or activity text on mobile | Permit wrapping and use responsive card/layout rules without clipping important content. |
| Color accessibility | Pair color with readable text, labels, and/or icons or other non-color indicators; maintain sufficient contrast. |
| Untrusted text in project data | Render with `textContent` to prevent HTML injection. |
| Port `5500` conflict | Surface the server startup failure and stop or resolve the conflicting local process before retrying; retain the required configured port. |
| Directory-listing regression | Verify `serverReadyAction` opens `/index.html`, not the server root. |

## Validation expectations

The Orchestrator should validate the completed result as follows:

1. Verify all four required files exist: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.
2. Parse both JSON files with `python3 -m json.tool`: the project data and launch configuration must be valid JSON.
3. Validate the data schema: the root is an object containing a `projects` array, and representative entries include `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary`.
4. Inspect `app/index.html` for the exact visible title `Project Pulse`, a stylesheet reference, JSON fetch of `project-data.json`, DOM/`textContent` card rendering, `.dashboard` and `.project-card` usage, and loading, empty, and error state coverage.
5. Inspect `app/styles.css` for `.dashboard` and `.project-card` hooks, `border-radius`, `box-shadow`, responsive rules, contrast-aware status/priority treatment, non-color semantic cues, and visible focus styling.
6. Inspect `.vscode/launch.json` for the exact configuration name, `cwd` of `${workspaceFolder}/app`, `python3 -m http.server 5500`, and a `serverReadyAction` URL of `http://localhost:%s/index.html`.
7. Run the VS Code configuration and confirm that it opens the rendered `app/index.html` page with multiple project cards rather than a directory listing.
8. Perform desktop and mobile-width visual checks for hierarchy, card readability, wrapping, status/priority visibility, and layout.
9. Exercise an error state and keyboard focus; confirm error messaging is understandable and status/priority meaning is not communicated by color alone.
10. Stop the local server after validation.

## Open questions

The repository brief does not specify the following. The Orchestrator should resolve them before or during the shared contract step, using sensible representative defaults if no answer is available:

- Exact sample project data and the number of projects to show.
- The approved status, priority, and risk taxonomy and its labels.
- Preferred recent-activity format, including whether it should be human-readable relative time or a fixed date.
- Whether cards need interactions beyond static display, such as filtering, links, or detail views.
- Required browser support baseline.
