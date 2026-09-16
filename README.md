# NetworkDesigner — DCIM Architect

A browser-based data center infrastructure and network topology designer. Drag equipment onto a canvas, cable it together, group it into Purdue-model security zones, and export the whole design as JSON — all client-side, no build step, no backend.

**Live demo:** https://dylan7474.github.io/NetworkDesigner/

## Features

- **26-item component catalog** across five categories:
  - **Compute & Storage** — server racks, individual rack servers, SAN storage, backup/tape libraries, KVM consoles
  - **Network & Connectivity** — switches, routers, firewalls, load balancers, inter-DC core routers, VPN/SD-WAN gateways, DNS/NTP, patch panels, wireless APs
  - **Facilities & Power** — CRAH/chillers, UPS systems, PDUs (floor and rack), generators, fire suppression
  - **Security & Access** — CCTV, biometric doors, environmental sensors, SIEM/SOC monitoring
  - **Sites & Connectivity** — remote data center sites and public cloud endpoints, for modeling multi-site and hybrid-cloud architectures
- **Drag-and-drop canvas** with pan, zoom, and snap-to-grid
- **Typed cabling** between devices — fiber, copper, power, BMS/Modbus, and IP security links, each rendered with distinct styling
- **Purdue security zones** — draw labeled zone regions and tag them with a Purdue Enterprise Reference Architecture level (Level 5 down to Level 0, including the 3.5/IDMZ tier) or a custom grouping, to show which tier each device sits in
- **Live inspector panel** for editing device labels, IPs, status, power draw, and notes, or zone labels/levels
- **Design summary stats** — node count, active links, zone count, and estimated total power load
- **Save / Load as JSON** — export the full topology (devices, cabling, zones) to a file and reload it later

## Getting started

This is a single self-contained HTML file with no dependencies to install and no build step.

- **Online:** just open the [live demo](https://dylan7474.github.io/NetworkDesigner/)
- **Locally:** clone the repo and open `index.html` directly in a browser, or serve it with any static file server, e.g.:
  ```bash
  python3 -m http.server 8000
  ```
  then visit `http://localhost:8000/index.html`

## Usage

1. Drag a component from the left-hand catalog onto the canvas (or click a card to drop it at the canvas center).
2. Select **Cable Link**, choose a cable type from the dropdown, then click a source device followed by a destination device to connect them.
3. Select **Zone**, then click-drag on empty canvas space to draw a zone. Click its colored label tab to rename it or change its Purdue/security level; drag the tab to move it, or the bottom-right handle to resize it.
4. Click any device, cable, or zone to inspect and edit its properties in the right-hand panel.
5. Use **Save JSON** / **Load** in the header to export or reimport a complete design.

## Tech stack

Plain HTML/CSS/JavaScript with [Tailwind CSS](https://tailwindcss.com/) loaded via CDN — no framework, no bundler, no package manager. Everything lives in `index.html`.

## License

Released under the [MIT License](LICENSE).
