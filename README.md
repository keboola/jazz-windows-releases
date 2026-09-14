# Jazz Capture for Windows (continuous streaming)

Windows tray client for **continuous desktop capture** with live OTLP logs and screenshot upload to Keboola Files. Prepared for **Zásilkovna** technical due diligence.

This tree is a slice of [keboola/jazz-desktop-client](https://github.com/keboola/jazz-desktop-client) (`feat/windows-keep-fleet-tray`): Windows host, portable core, enrollment/contract schemas. It does **not** include the macOS agent.

## What it does

- Tray app: Pause / Resume, Mark session end, labels, exempt apps
- Screenshots and microphone narration on; click-highlight off by default
- Starts with `--capture-at-launch` (MSI Run key / Intune)
- Streams activity to a Keboola **Data Stream** (`/v1/logs`)
- Uploads click screenshots to **Storage Files**
- Per-user identity on every event: `enduser.id` (Windows login), `host.name` (machine)

Secrets (Storage token, stream URL) are **not** in this repository. Use `windows/provisioning/device-bundle.example.json` and keep the real `device-bundle.json` out of git.

## Layout

```
windows/Sources/JazzCapture          tray host (net8.0-windows)
windows/Sources/JazzCaptureCore      portable engine, OTLP, journal
windows/Sources/JazzEnrollmentSecurity
windows/Tests/
windows/installer/                   WiX MSI + Deploy-JazzCapture.ps1
windows/provisioning/                bundle example + validator
contract/                            capture and enrollment schemas
dist/                                unsigned per-user MSI (no secrets inside)
```

Pre-built installer: [`dist/JazzCapture-0.26.5-win-x64-unsigned.msi`](dist/JazzCapture-0.26.5-win-x64-unsigned.msi)  
SmartScreen will warn; that is expected (no code-signing certificate). This MSI does **not** embed a Storage token. After install, drop a filled `device-bundle.json` as below.

## Build

.NET 8 SDK, Windows x64:

```powershell
dotnet test windows/JazzCapture.sln
dotnet build -c Release windows/Sources/JazzCapture/JazzCapture.csproj
```

MSI (embeds `windows/provisioning/device-bundle.json` **if that file exists locally**):

```powershell
powershell -File windows/installer/build-msi.ps1
# -> windows/installer/artifacts/JazzCapture-0.26.5-win-x64-unsigned.msi
```

The MSI is **unsigned**. SmartScreen will warn.

## Run

```powershell
& .\windows\Sources\JazzCapture\bin\Release\net8.0-windows\JazzCapture.exe --capture-at-launch
```

Or install the MSI (starts capture at logon). Drop a filled bundle at:

`%LOCALAPPDATA%\Jazz\provisioning\device-bundle.json`

Jazz consumes and deletes it after enrollment.

## Enrollment: “MVP unsigned bundle” vs TPM device-bound

Two different ways a PC can get a Keboola credential:

**What this build uses (MVP unsigned bundle)**  
An operator (or Intune) drops a JSON file that already contains a Storage token and Data Stream URL. Jazz checks it against Keboola (`tokens/verify`), stores it with Windows DPAPI, and deletes the JSON. Any PC with that file can use the **same company token**. Who is capturing is still on each event (`enduser.id`, `host.name`). There is **no** hardware proof that this laptop is a unique enrolled device.

**What production Jazz Capture on macOS can do (TPM / Secure Enclave device-bound)**  
The machine creates a key that never leaves the chip (Secure Enclave on Mac, TPM on Windows when that path is finished). The server issues a credential bound to **that device**. Copying the credential to another PC does not work. Windows TPM binding is **not** in this due-diligence slice.

For a fleet PoC, the MVP bundle is enough. It is not the same security bar as device-bound enrollment.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
