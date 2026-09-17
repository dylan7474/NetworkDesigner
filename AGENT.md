# Agent Instructions

This file guides AI coding agents (and future contributors) working in this repository.

## What this project is

A single-page, client-side data center/network topology designer ("DCIM Architect"). There is no backend, no build step, no package manager, and no automated test suite. The entire application — markup, styles, and logic — lives in **`index.html`**.

- Styling: [Tailwind CSS](https://tailwindcss.com/) loaded via the Play CDN (`<script src="https://cdn.tailwindcss.com">`), plus a small `<style>` block for custom scrollbars, selection glow, and animated cable dashes.
- Logic: vanilla JavaScript in one inline `<script>` at the bottom of `<body>`. No frameworks, no modules, no transpilation.
- State: a single global `state` object (`nodes`, `connections`, `zones`, `vlans`, counters, selection, mode, zoom, `highlightVlanId`) plus lookup tables defined near the top of the script: `COMPONENT_METADATA` (device catalog), `CABLE_CONFIGS`, `ZONE_LEVELS` (Purdue levels), `ROLE_TAGS` (server software roles, e.g. Bastion/AD/DNS), and `ACCESS_ROLES` (zone access personas, e.g. DC Manager/CCTV Support — also reused as the persona list for `user`-category nodes' "User Type").
- Zones have a `zoneType`: `'purdue'` (default; styled by `ZONE_LEVELS`) or `'cluster'` (a hypervisor cluster; solid border, styled by `CLUSTER_ZONE_STYLE`). Zones can be drawn nested inside one another — `renderZoneElement()` gives smaller-area zones a higher `z-index` so nesting always renders correctly regardless of creation order. Zone membership (which nodes/zones are "in" which zone) is purely geometric — a node or zone's center point falling inside another zone's rectangle — there is no explicit parent/child link stored anywhere; see `getContainingZones()`.
- Dragging a zone (`handleZoneDragStart`) moves its contents with it: at drag start it snapshots every node and every strictly-smaller nested zone whose center currently falls inside the dragged zone's rectangle, then applies the same mouse delta to all of them each tick. This is a one-time snapshot for the interaction, not a stored relationship — it doesn't contradict the "purely geometric, no parent/child link" rule above, since membership is still re-derived from scratch (via `isNodeContained`/center-point checks) the next time anything asks. Resizing a zone (`handleZoneResizeStart`) intentionally does *not* move contents — only dragging does.
- A hypervisor cluster's hosts and guest VMs are **not** separate stored entities — they're ordinary compute-category nodes (Rack Server, Blade, etc.) dragged inside the cluster zone's rectangle, exactly like any other zone-membership relationship in this app. `getClusterHostNodes()`/`countClusterHosts()` find contained nodes tagged with the `hypervisor` role; `getClusterGuestNodes()` finds every other contained compute node (any role, including none) and is what the zone inspector's "Guest Servers" list and the zone chip's guest count are built from. Don't reintroduce a separate roster field on the zone for this — it was tried (`zone.guestVMs`) and replaced because it duplicated data that already lives on real node objects and couldn't be dragged onto the canvas.
- Persistence: `buildExportPayload()` is the single source of truth for what a saved/shared design contains. `loadTopologyState(incomingState)` is the single source of truth for rebuilding the live canvas from one — it's shared by file import (`importTopologyJSON`), the share-link loader (`loadFromShareLinkIfPresent`), and the autosave-restore path on boot (`bootApp`). Never reimplement the reset/rebuild sequence inline elsewhere.
- The Firewall Rule Matrix (`buildFirewallRuleMatrix()`) is a purely derived view — nothing about it is stored in `state` or persisted. It's recomputed from `state.connections` and `state.zones` every time the modal opens, via `findContainingSecurityZone()` (the same smallest-area-wins nesting rule used elsewhere) and the `PURDUE_OT_LEVELS`/`PURDUE_IT_LEVELS` boundary check. Keep it that way — don't cache or persist rule results, since they'd silently go stale the moment a node moves or a zone's level changes.

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

5. **Keep `buildExportPayload()` and `loadTopologyState()` in sync** with whatever top-level state fields exist (`nodes`, `connections`, `zones`, `vlans`, `counter`, `zoneCounter`, `cableCounter`, …) — if you add a new piece of persistent state, include it in both functions, and bump the `version` string in `buildExportPayload()` if the schema changes in an incompatible way. These two functions back JSON export/import, the shareable-link feature, and autosave alike — fixing one without the other silently breaks the other two.

6. **Call `scheduleAutosave()` at the end of every function that mutates `state`** (creating/deleting/editing a node, connection, zone, or VLAN; drag-end for nodes and zones). It's cheap and debounced (350ms), so prefer calling it defensively over trying to reason about whether a given mutation "really" needs persisting. It writes through `buildExportPayload()`, so a new state field only needs to be added there, not to every autosave call site.

7. **IDs must not collide within a session.** Nodes, zones, cables, and guest VMs each use their own counter (`state.counter`, `state.zoneCounter`, `state.cableCounter`, `state.vmCounter`) rather than `Date.now()` — several can be created in the same synchronous tick (e.g. the seed demo, or a bulk import), and millisecond timestamps are not guaranteed unique across rapid calls. Follow the same counter pattern for any new entity type.

8. **Call `refreshZoneDerivedState()` after anything that could change zone membership or a zone's access rules.** It re-renders every cluster zone's host/VM-count badge and every `user`-category node's access-violation styling, since both are computed live at render time from current node positions and `zone.accessRoles`/`role==='hypervisor'` — they are not stored, so a stale render is the only way they'd ever be wrong. Existing call sites: node create/delete, node drag-end, `updateNodeRole`, `updateNodeUserType`, zone drag/resize-end, `toggleZoneAccessRole`, `updateZoneType`, `executeDeleteZone`, and the end of `loadTopologyState`. Add it to any new mutation in the same family (e.g. a future bulk-move or duplicate feature).

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
