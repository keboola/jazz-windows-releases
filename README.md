# Jazz Capture for Windows

Public source and **MSI releases** for the Windows tray client (continuous capture → Keboola Data Stream + Files).

Development happens in [keboola/jazz-desktop-client](https://github.com/keboola/jazz-desktop-client). This repository is the **public publish target**: the releases here are what an installed client can detect and install. The client checks this repository for a newer release and installs the MSI automatically.

It is a Windows-only slice of [keboola/jazz-desktop-client](https://github.com/keboola/jazz-desktop-client). It does **not** include the macOS agent.

## Releases

Installers are attached to [GitHub Releases](https://github.com/keboola/jazz-windows-releases/releases).

The [latest release](https://github.com/keboola/jazz-windows-releases/releases/latest) is the current package. `dist/` holds only the v0.26.5 MSI from the first import.

### Source and tags

The source in this repository is the v0.26.5 import from 2026-09-14, and every tag from v0.26.5 to v0.26.26 points to that import. The "Source code" archives of later releases therefore contain the v0.26.5 source. The source of later MSIs is in keboola/jazz-desktop-client. For example, v0.26.26 was built from [31caca5](https://github.com/keboola/jazz-desktop-client/commit/31caca5119cc0b81a2ee27d2a0e0b899a7821bbe).

The MSI is **unsigned** (SmartScreen will warn). It does **not** embed a Storage token or stream URL. After install, drop a filled bundle at:

`%LOCALAPPDATA%\Jazz\provisioning\device-bundle.json`

Jazz consumes and deletes it after enrollment.

## What it does

- Tray: Pause / Resume, Mark session end, labels, exempt apps
- Screenshots on by default. Narration is off until the user accepts the voice prompt for a label. Click-highlight is off by default.
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
