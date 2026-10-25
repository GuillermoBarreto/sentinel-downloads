# Sentinel 0.5.0 — Data-center preview

Sentinel now opens a data-center operations workspace for mixed on-premises environments. This is an evaluation release, not a production DCIM platform or autonomous AI defender.

## Download and install

Download **Sentinel-Setup.exe** and open it on Windows 10/11 x64. Python is included. Setup installs Microsoft WebView2 if missing, which requires internet. Open **Sentinel** from the Desktop or Start menu. The installer is unsigned; verify the source and checksum. Do not disable your antivirus or restore quarantined files to use this preview.

## Included

- Site, room and rack inventory with clear unknown, stale and missing-measurement states.
- Confirmed threshold alerts, acknowledgement and bounded telemetry history.
- An explicit sample environment with synthetic readings, separate from live inventory.
- Read-only host and configured Redfish collectors, distributed in **Sentinel-Collector.zip**.
- Existing Windows performance, network visibility, security-health and reviewed support tools.

The Windows console uses a local workspace. Remote collectors require a separately deployed HTTPS control plane, organization credentials and registered assets; this website does not provide a hosted monitoring account. See **SETUP.md** for coverage, authentication, limits and configuration.

## Verification and limits

166 API tests, 41 desktop tests, 13 data-center/collector tests and the frontend production build passed. The packaged Windows executable opened in an isolated workspace, exercised its native bridge and pages, and advanced its automatic workstation-watch heartbeat. Real localhost HTTP workflows, migrations and one host collector report passed. The installer itself was built but not installed in this release check; PostgreSQL migration SQL was compiled offline only. No physical data-center vendor equipment was connected or certified.

SNMPv3, hypervisor APIs, paging, facility BMS integrations, fleet-scale validation, signed updates and production service guarantees remain future work. No bundled local AI runtime, autonomous equipment repairs or data-center AI diagnosis is included. Optional Muse support requires explicit consent and provider access; no paid inference or provider quota was verified for this release.

Existing workspaces are preserved by setup; back up important workspaces before evaluation upgrades. Never post credentials or unredacted system/device logs in issue reports.
