# Agent Instructions

This file guides AI coding agents (and future contributors) working in this repository.

## What this project is

A single-page, client-side data center/network topology designer ("DCIM Architect"). There is no backend, no build step, no package manager, and no automated test suite. The entire application — markup, styles, and logic — lives in **`index.html`**.

- Styling: [Tailwind CSS](https://tailwindcss.com/) loaded via the Play CDN (`<script src="https://cdn.tailwindcss.com">`), plus a small `<style>` block for custom scrollbars, selection glow, and animated cable dashes.
- Logic: vanilla JavaScript in one inline `<script>` at the bottom of `<body>`. No frameworks, no modules, no transpilation.
- State: a single global `state` object (`nodes`, `connections`, `zones`, counters, selection, mode, zoom) plus a `COMPONENT_METADATA` catalog and `CABLE_CONFIGS` / `ZONE_LEVELS` lookup tables defined near the top of the script.

## Deployment

GitHub Pages serves this repo directly from the `main` branch root — pushing to `main` auto-redeploys within about a minute. There is no CI, no `.github/workflows`, and no separate `gh-pages` branch. **Do not** introduce a build step or move source into a `src/` or `dist/` folder without also updating the Pages source setting, or the live site will break.

## Making changes

Because everything is one file, most changes touch multiple spots that must stay in sync:

1. **Adding a new draggable component type** requires edits in two places:
   - A new entry in `COMPONENT_METADATA` (name, category, `icon` SVG string sized `w-5 h-5`, `color` border classes, `powerKW`, `defaultPorts`, `specs`).
   - A matching palette card `<div draggable ondragstart="handlePaletteDragStart(event, '<key>')">` in the left sidebar, under the appropriate category section, with its own `w-4 h-4` icon and a distinct Tailwind color swatch (`bg-*-950/60 border-*-500/30 text-*-400`) so it doesn't blend in with neighboring cards in the same category.
   - The palette item count badge updates itself automatically from `Object.keys(COMPONENT_METADATA).length` — no manual count to edit.
   - Give `defaultPorts` a non-zero value even for abstract nodes (e.g. cloud/remote-site) — the ports label uses `meta.defaultPorts || 24`, so a literal `0` falls back to displaying `24`.

2. **Never interpolate user-editable or imported data directly into `innerHTML`.** Node labels, IPs, notes, zone labels, and peer/cable labels are all user-controlled (either typed directly, or loaded from an imported JSON file) and must be passed through the `escapeHtml()` helper before being placed in a template string. This app takes arbitrary `.json` files via the "Load" button — treat every field from an imported file as untrusted.

3. **Re-rendering an existing DOM node or zone must remove the old element first.** Both `renderNodeElement()` and `renderZoneElement()` do `document.getElementById(id)?.remove()` before creating the replacement. If you add another code path that re-renders an existing entity by id, follow the same pattern — skipping it silently leaves a stale duplicate element (with its own event listeners) behind, which is exactly the bug that was fixed in this codebase once already.

4. **`importTopologyJSON()` must stay defensive.** It accepts arbitrary files from the "Load" button. Keep validating `archetypeKey` against `COMPONENT_METADATA` before calling `createNode`, and keep resetting `nodesContainer`, `cableGroup`, and `zonesContainer` (plus `state.nodes`/`connections`/`zones`) before rebuilding, so a partially-invalid file can't leave the canvas in a mixed old/new state.

5. **Keep `exportTopologyJSON()` and `importTopologyJSON()` in sync** with whatever top-level state fields exist (`nodes`, `connections`, `zones`, `counter`, `zoneCounter`, …) — if you add a new piece of persistent state, export it and restore it on import, and bump the `version` string in the exported payload if the schema changes in an incompatible way.

## Testing

There is no automated test suite. Verify changes by actually opening the page in a browser:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

At minimum, after any change, check the browser console for errors and exercise the golden path: drag a device onto the canvas, cable two devices together, draw a zone, select/edit each of node/cable/zone in the inspector, and run a Save → Load round trip. `node -e "new Function(...)"` on the extracted inline `<script>` contents is a fast way to catch syntax errors before opening a browser.

## Conventions worth preserving

- Tailwind utility classes are used inline everywhere; there's no separate CSS file to touch aside from the `<style>` block in `<head>`.
- SVG icons are hand-written inline (24×24 viewBox, `fill="none" stroke="currentColor"`), not pulled from an icon library — keep new icons in the same style for visual consistency.
- Toasts (`showToast`) and the custom confirm modal (`showConfirmModal`) exist specifically to avoid native `alert()`/`confirm()`, which block the extension/automation-driven browsing this app is sometimes tested with. Keep using them for any new user-facing notifications or destructive-action confirmations.
