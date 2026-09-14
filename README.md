# Jazz Capture for Windows

Public source and **MSI releases** for the Windows tray client (continuous capture → Keboola Data Stream + Files).

Development is tested in a private tree. This repository is the **public publish target**: tagged versions here are what a deployed client can detect and install.

It is a Windows-only slice of [keboola/jazz-desktop-client](https://github.com/keboola/jazz-desktop-client). It does **not** include the macOS agent.

## Releases

Installers are attached to [GitHub Releases](https://github.com/keboola/jazz-windows-releases/releases).

Current package (also in `dist/`):

- [JazzCapture-0.26.5-win-x64-unsigned.msi](dist/JazzCapture-0.26.5-win-x64-unsigned.msi)

The MSI is **unsigned** (SmartScreen will warn). It does **not** embed a Storage token or stream URL. After install, drop a filled bundle at:

`%LOCALAPPDATA%\Jazz\provisioning\device-bundle.json`

Jazz consumes and deletes it after enrollment.

## What it does

- Tray: Pause / Resume, Mark session end, labels, exempt apps
- Screenshots and microphone narration on; click-highlight off by default
- Starts with `--capture-at-launch` (MSI Run key)
- Streams activity to a Keboola Data Stream (`/v1/logs`)
- Uploads click screenshots to Storage Files
- Per-user identity on every event: `enduser.id`, `host.name`

Secrets are **not** in this repository. Use `windows/provisioning/device-bundle.example.json`.

## Layout

```
windows/Sources/JazzCapture          tray host (net8.0-windows)
windows/Sources/JazzCaptureCore      portable engine, OTLP, journal
windows/Sources/JazzEnrollmentSecurity
windows/Tests/
windows/installer/                   WiX MSI + Deploy-JazzCapture.ps1
windows/provisioning/                bundle example + validator
contract/                            capture and enrollment schemas
dist/                                unsigned per-user MSI (no secrets)
```

## Build

.NET 8 SDK, Windows x64:

```powershell
dotnet test windows/JazzCapture.sln
dotnet build -c Release windows/Sources/JazzCapture/JazzCapture.csproj
powershell -File windows/installer/build-msi.ps1
```

If `windows/provisioning/device-bundle.json` exists locally, the MSI embeds it. Do not commit that file.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
