# Sentinel for Windows

Official early-access installers and checksums for Sentinel. No Python or developer tools are needed to install the application.

## Download and install

1. Download **Sentinel-Setup.exe** from [Releases](https://github.com/GuillermoBarreto/sentinel-downloads/releases).
2. Run the installer for your Windows account.
3. Open **Sentinel** from the Desktop or Start menu.

Windows 10/11 x64. Microsoft WebView2 is installed if missing; that step needs internet. Existing Python-based Sentinel workspaces are copied into the new per-user data location on first launch, leaving the original intact.

The preview installer is currently unsigned. Windows may show an unknown-publisher warning. Verify the release source and SHA-256 checksum before deciding whether to install.

## Included

- Live CPU, RAM, GPU, storage, process and network metrics where supported.
- Bounded local-network discovery with clear neighbor-cache states.
- Actual Windows security health and reviewed Defender/DISM/SFC action controls.
- Native app window, per-install local credentials, and an uninstaller that preserves personal workspace data.

This is early-access software, not a finished commercial antivirus. The separate AI investigation workspace still uses a mock provider. Windows security actions can alter files and require explicit native confirmation; elevated workflows need additional disposable-system validation. Zero known threat records or a successful command exit is not proof a PC is clean or fully repaired.

System readings stay local. Defender and Windows tools retain their own cloud, update and remediation policies. See the app's About & privacy page.

This repository distributes preview installers, documentation and checksums. It does not contain the private development repository. Third-party notices are included with the application.

## Help

Use this repository's issues for feedback. Do not upload credentials, private files, threat resource paths, MAC addresses or unredacted system logs.
