# Tenexity Workflow Capture Desktop Build

This checkout is the desktop capture base for Tenexity Workflow Capture.

## Tenexity Configuration

- Tauri config: `apps/screenpipe-app-tauri/src-tauri/tauri.tenexity.conf.json`
- Env template: `apps/screenpipe-app-tauri/src-tauri/tenexity.env.example`
- Managed config template: `apps/screenpipe-app-tauri/src-tauri/tenexity.enterprise.example.json`
- App icons: `apps/screenpipe-app-tauri/src-tauri/icons/tenexity/*`
- Tray icons: `apps/screenpipe-app-tauri/src-tauri/assets/tenexity-logo-tray-*.png`
- Enterprise sync module: `ee/desktop-rust/enterprise_sync.rs`
- Policy hook: `apps/screenpipe-app-tauri/lib/hooks/use-enterprise-policy.ts`

## Environment

```text
TENEXITY_CAPTURE_DEVICE_KEY=replace-with-per-device-secret
TENEXITY_CAPTURE_PROJECT_ID=replace-with-tenexity-project-id
TENEXITY_CAPTURE_API_URL=https://api.tenexity.ai
TENEXITY_CAPTURE_RETENTION_DAYS=30
```

Legacy `SCREENPIPE_ENTERPRISE_LICENSE_KEY` and `SCREENPIPE_ENTERPRISE_INGEST_URL` remain supported as fallbacks.

For Intune or signed-EXE pilots, deploy an `enterprise.json` beside the executable
or into the app data directory:

```json
{
  "license_key": "replace-with-per-device-secret",
  "project_id": "replace-with-tenexity-project-id",
  "api_url": "https://api.tenexity.ai"
}
```

`license_key` is intentionally accepted as the device key so the existing
enterprise UI policy prompt and the Rust sync task read the same managed file.

## Windows Build

```bash
cd apps/screenpipe-app-tauri
bunx tauri build --config src-tauri/tauri.tenexity.conf.json --target x86_64-pc-windows-msvc --features enterprise-build
```

The existing enterprise release workflow already has the primitives needed for Windows signing, NSIS packaging, and Intune packaging. The Tenexity release path should reuse that flow with the Tenexity config and final Tenexity assets.

## Windows Release Workflow

Run the manual GitHub Actions workflow:

```text
Release Tenexity Workflow Capture
```

The workflow copies `tauri.tenexity.conf.json` to `tauri.conf.json`, builds the
Windows x64 enterprise target, creates an Intune package, and uploads the
installer plus `.intunewin` as artifacts. When the SSL.com and Tauri signing
secrets are configured, the same workflow signs the NSIS installer before
upload; unsigned artifacts should stay limited to pilot validation.

## Current Evidence Stream

The enterprise sync stream emits JSONL records for:

- `frame`: OCR/accessibility text with app, window, and browser URL context.
- `audio`: transcription text and speaker/device metadata when available.
- `ui`: click, text, clipboard, focus, app switch, and other UI events.
- `snapshot`: downsized screenshot thumbnails.

Full video/audio chunk upload should be implemented through a separate media upload flow and recorded through the Tenexity media manifest endpoint.
