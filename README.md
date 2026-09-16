# NetworkDesigner — DCIM Architect

A browser-based data center infrastructure and network topology designer. Drag equipment onto a canvas, cable it together, group it into Purdue-model security zones, and export the whole design as JSON — all client-side, no build step, no backend.

**Live demo:** https://dylan7474.github.io/NetworkDesigner/

## Features

- **27-item component catalog** across six categories:
  - **Compute & Storage** — server racks, individual rack servers, SAN storage, backup/tape libraries, KVM consoles
  - **Network & Connectivity** — switches, routers, firewalls, load balancers, inter-DC core routers, VPN/SD-WAN gateways, DNS/NTP, patch panels, wireless APs
  - **Facilities & Power** — CRAH/chillers, UPS systems, PDUs (floor and rack), generators, fire suppression
  - **Security & Access** — CCTV, biometric doors, environmental sensors, SIEM/SOC monitoring
  - **Sites & Connectivity** — remote data center sites and public cloud endpoints, for modeling multi-site and hybrid-cloud architectures
  - **Users & Personas** — a generic User object, tagged with an access-role persona, for showing who reaches into the design and where
- **Drag-and-drop canvas** with pan, zoom, and snap-to-grid
- **Typed cabling** between devices — fiber, copper, power, BMS/Modbus, and IP security links, each rendered with distinct styling
- **Purdue security zones** — draw labeled zone regions and tag them with a Purdue Enterprise Reference Architecture level (Level 5 down to Level 0, including the 3.5/IDMZ tier) or a custom grouping, to show which tier each device sits in. Zones can be drawn nested inside one another.
- **Hypervisor cluster zones** — a second zone type representing a virtualization cluster (up to 3 hosts). Drop it inside a security zone, tag up to three devices with the Hypervisor Host role to be auto-detected and counted (with a warning past 3), and maintain a guest VM roster on the cluster itself — each guest gets its own name, server role, VLAN, IP, and status, since VMs in an HA/DRS cluster belong to the cluster's resource pool rather than one fixed host.
- **User/Persona objects with live access-compliance checking** — drop a User object into a zone and tag it with a persona; if that persona isn't in the zone's Authorized Access list, the user is flagged on the canvas (red ring + warning badge) and in the inspector, immediately, as you move things around — a first step toward a future access-path / virtual pen-test feature.
- **Live inspector panel** for editing device labels, IPs, status, power draw, and notes, or zone labels/levels
- **Server role tagging** — tag compute hardware with a common environment role (Bastion/Jump Host, Active Directory, DNS, DHCP, IPAM, NTP, Database, Web/App, Proxy, RADIUS, Building Management/BMS, Environmental Monitoring/EMS, CCTV/VMS, Access Control/PACS, Hypervisor Host, Virtualization Management, Kubernetes, DCIM Platform, SCADA/HMI, and more), shown as a colored badge on the device card
- **Zone access control** — tag each zone with the personas authorized to access it (DC User, DC Manager, Network Ops, Security Officer, CCTV Support, Door/Badge Access Support, Facilities Technician, Storage Admin, Auditor, Corp User, DC Ops User, DC Support User, Third-Party/Vendor), shown as chips on the zone's label tab
- **VLAN tagging & highlight mode** — define a VLAN directory (ID, name, subnet), tag cables (and cluster guest VMs) as access or 802.1Q trunk links, and isolate any single VLAN's path across the whole diagram — devices, cables, and guest rosters alike — with one dropdown
- **Design summary stats** — node count, active links, zone count, and estimated total power load
- **Autosave** — the current design is continuously saved to this browser's local storage, so a reload or accidental tab close won't lose work
- **Shareable links** — copy a compressed, self-contained link that reproduces your exact design for a colleague to open (a one-time snapshot, not live co-editing — no backend involved)
- **Save / Load as JSON** — export the full topology (devices, cabling, zones, VLANs) to a file and reload it later

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
3. Select **Zone**, then click-drag on empty canvas space to draw a zone (you can draw one inside another). Click its colored label tab to rename it or change its Purdue/security level; drag the tab to move it, or the bottom-right handle to resize it. In its inspector, switch **Zone Type** to "Hypervisor Cluster" to turn it into a virtualization cluster with a guest VM roster instead of a Purdue level.
4. Click any device, cable, or zone to inspect and edit its properties in the right-hand panel — including a Server Role for compute hardware, a User Type for a dropped User object, or Authorized Access roles for a zone.
5. Use **Manage VLANs** in the toolbar to define VLANs, then tag them onto a selected cable from its inspector. Use the **Highlight** dropdown next to it to isolate one VLAN's path across the whole diagram.
6. Use **Save JSON** / **Load** in the header to export or reimport a complete design, or **Copy Share Link** to hand someone a one-time snapshot of the current design without a file.

Your work is autosaved to this browser automatically — no login and no server involved, so it won't follow you to another browser or device unless you use Save JSON / Share Link.

## Tech stack

Plain HTML/CSS/JavaScript with [Tailwind CSS](https://tailwindcss.com/) loaded via CDN — no framework, no bundler, no package manager. Everything lives in `index.html`.

## License

Released under the [MIT License](LICENSE).
