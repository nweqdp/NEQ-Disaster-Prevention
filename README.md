# NEQ-Disaster Prevention

An experimental Windows app for viewing disaster information in Japan on one map. It combines JMA weather warnings, earthquake and tsunami information, earthquake early warnings (EEW), and NIED Kmoni station readings. The interface supports Japanese and English.

> **Alpha software.** This is an independent viewer, not an official warning channel. Data may be delayed, unavailable, incomplete, or displayed incorrectly. For protective decisions, check announcements from [JMA](https://www.jma.go.jp/) and your local authorities.

## Features

| View | What it shows |
| --- | --- |
| Weather | Current received JMA warnings and advisories, prefecture colors, details, updates, and typhoon information. |
| Earthquakes | Recent JMA earthquake reports, epicenters, magnitude, depth, and reported seismic intensities. |
| Shaking detection | Kmoni station colors and estimated shaking categories based on NIED's monitor images. These are **not confirmed JMA intensities**. |
| EEW | Report number, final/cancellation state, estimated maximum intensity, source details, predicted regions, and long-period ground-motion estimates when supplied. Conventional reports can display estimated epicenters and illustrative P/S rings; PLUM reports are labelled separately. |
| Tsunami | Received JMA tsunami forecast areas, expected arrival/height where supplied, and observations. The active tsunami display has priority until the warning is lifted. |

The app includes configurable sounds, Japanese announcements through Bouyomi-chan, an EEW simulation, and a tsunami forecast test. Test displays are labelled and do not create real alerts.

## Install on Windows

When a release is published, download its **NSIS Installer.exe** from this repository's GitHub Releases, run it, and choose an installation folder. The setup wizard offers **Run NEQ-Disaster Prevention** and **Create desktop shortcut** on its final page. Uninstall through Windows **Installed apps** or the Start menu.

Windows 10/11 and the [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) are required. The installer bundles the map assets; internet access is still needed for live reports and station readings. Download releases only from a source you trust. Builds are currently unsigned, so Windows may show a SmartScreen warning.

To use speech, start Bouyomi-chan separately with its local TCP interface available at `127.0.0.1:50001`. Without Bouyomi-chan, visual information still works. Sounds and speech can be configured in **Menu → Sound settings**.

## Data sources

- **Weather warnings, typhoons, and tsunami:** JMA public XML feeds. Weather continues to use JMA even if another earthquake provider is selected.
- **Earthquake information:** JMA by default; DMDATA can be selected in **Menu → Data source** with a suitable subscription and API key. The key is held for the current session only.
- **EEW:** Wolfx backup stream, with DMDATA preferred when an authorized EEW connection is available. A weather-only DMDATA subscription may not include EEW access.
- **Shaking detection:** NIED Kmoni surface-monitor images. The color-derived values are estimates, can be noisy, and must not be read as official earthquake reports.
- **Map:** bundled geoBoundaries ADM1 tiles and JMA earthquake/tsunami area geometry.

Status indicators show connection and data errors where available. Gray or missing map data should not be interpreted as an all-clear.

## Build from source

Requirements: Windows, Go **1.25+**, Node.js **22.13+** with npm, and WebView2 Runtime. NSIS is needed only to build the installer.

```powershell
npm ci
npm run build:go
go test ./internal/...
cd desktop
go build -tags "desktop,production" -ldflags "-H windowsgui" -o "build/bin/NEQ-Disaster Prevention.exe" .
```

Run the desktop executable with `web-dist` and `config` beside it; its relative paths resolve from the executable's folder. For a local browser-based development view:

```powershell
go run ./cmd/warning-map -mode demo -bouyomi=
```

Open `http://127.0.0.1:8080`. Demo mode uses sample data and is **not live**. The NSIS installer source is in [`installer/NEQ-Disaster-Prevention.nsi`](installer/NEQ-Disaster-Prevention.nsi); compile it with `makensis` after staging the built executable, `web-dist`, and `config` in one directory.

## Project layout

- `app/`: React/MapLibre interface and map overlays.
- `internal/jma/`, `internal/eew/`, `internal/server/`: parsing, feeds, state, speech, and local API.
- `desktop/`: Wails/WebView2 Windows application.
- `public/`: bundled static map data copied into `web-dist` by the frontend build.
- `config/`: area and Kmoni station mappings, including their data-license notes.
- `installer/`: Windows installer source.

## Attribution and project status

Map boundaries use [geoBoundaries gbOpen](https://www.geoboundaries.org/) under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Earthquake/tsunami area geometry comes from [JMA GIS data](https://www.data.jma.go.jp/developer/gis.html). Kmoni station and palette attribution is recorded in `config/KMONI-STATIONS-LICENSE.txt` and `config/KMONI-COLOR-LICENSE.txt`. Check [NIED's Kmoni usage terms](https://www.kyoshin.bosai.go.jp/ja/about_kmoni/) before redistributing monitor-derived data.

This repository has no project-wide license file yet. Do not assume the application source is licensed for redistribution merely because some bundled data has separate licenses.