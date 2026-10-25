# Sentinel — Data-center operations preview

Public installers, collectors, documentation and checksums. The development repository remains private.

**[Download Sentinel for Windows](https://github.com/GuillermoBarreto/sentinel-downloads/releases/download/v0.5.0-preview/Sentinel-Setup.exe)** · **[Release notes](https://github.com/GuillermoBarreto/sentinel-downloads/releases/tag/v0.5.0-preview)** · **[Website](https://sentinel-landing-wine.vercel.app)**

## Install

1. Download Sentinel-Setup.exe for Windows 10/11 x64.
2. Verify its [SHA-256 checksum](https://github.com/GuillermoBarreto/sentinel-downloads/releases/download/v0.5.0-preview/Sentinel-Setup.exe.sha256), then run setup for your Windows account.
3. Open **Sentinel** from the Desktop or Start menu. Python is included; setup installs WebView2 if missing, requiring internet.

Version **0.5.0-preview** is unsigned early-access software. Keep endpoint protection enabled and investigate any detection with your security provider. Installation preserves the existing workspace; back it up before upgrading.

## What you can evaluate

- Site, room and rack inventory, missing/stale coverage and metric history.
- Confirmed threshold alerts and operator acknowledgement.
- An explicitly labeled synthetic sample environment.
- Read-only host and configured Redfish collectors.
- Windows performance, network visibility, security-health and reviewed support tools.

The Windows console owns a local API bound to loopback. **Remote site collectors need a separately deployed HTTPS control plane, organization credentials and registered assets.** No hosted monitoring account or connected equipment is supplied by this download.

Download [Sentinel-Collector.zip](https://github.com/GuillermoBarreto/sentinel-downloads/releases/download/v0.5.0-preview/Sentinel-Collector.zip) and read [SETUP.md](SETUP.md) before configuring equipment. The collector needs Python and its pinned dependency. Redfish uses a read-only account and verified TLS; no subnet discovery or device-changing commands are included.

## Preview boundaries

This is not a production DCIM/NMS/BMS platform, an antivirus replacement, or an autonomous AI defender. SNMPv3, hypervisor integrations, paging, production retention, signed updates and broader hardware validation remain future work. No bundled local AI runtime or autonomous repairs are shipped. Optional Muse support requires explicit consent and separate provider access; provider quota and paid inference were not verified for this release.

Desktop data stays in its configured local workspace. Site collectors send normalized measurements to your configured control plane. Optional cloud support may send the summary you explicitly approve. Windows tools keep their own cloud and remediation policies.

## Feedback

The preview is free to download and evaluate; no subscription or checkout is active.

[Share early-access feedback](https://github.com/GuillermoBarreto/sentinel-downloads/issues/new?template=early-access-feedback.yml) about installation, readings, workflows or missing capabilities. A GitHub account is required and feedback is public. Never upload credentials, private files, device addresses or unredacted logs. Third-party notices are included in the application.
