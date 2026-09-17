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

---

<a id="japanese"></a>

## 日本語

NEQ-Disaster Prevention は、日本の防災情報を一つの地図で確認するための実験的な Windows アプリです。気象庁の気象警報・注意報、地震・津波情報、緊急地震速報（EEW）と、防災科研（NIED）の強震モニタ観測点を表示します。画面表示は日本語と英語に対応しています。

> **アルファ版です。** このアプリは公的な警報の配信手段ではありません。情報の遅延、欠落、表示の誤りが起こる可能性があります。身の安全に関わる判断には、[気象庁](https://www.jma.go.jp/)や自治体の発表を確認してください。

### 主な機能

| タブ | 表示内容 |
| --- | --- |
| 気象警報 | 受信した気象庁の警報・注意報、都道府県ごとの色分け、詳細、更新情報、台風情報。 |
| 地震情報 | 最近の地震の震源、マグニチュード、深さ、発表された観測震度。 |
| 揺れ検知 | 防災科研の強震モニタ画像から読み取った観測点の色と、推定した揺れの区分。**気象庁が発表する震度ではありません。** |
| EEW | 報数、最終報・取消、推定最大震度、震源情報、予測地域、提供されている場合は長周期地震動の予測階級。通常の報では推定震央と参考用の P 波・S 波の円を表示し、PLUM 法の報は区別して表示します。 |
| 津波 | 受信した津波予報区、提供されている場合は到達予想時刻・予想される高さ、津波の観測情報。津波警報などの発表中は津波表示を優先します。 |

通知音の設定、棒読みちゃんによる日本語読み上げ、EEW シミュレーション、津波予報テストもあります。テスト表示は明示され、実際の警報は発表しません。

### Windows へのインストール

リリースを公開した場合は、このリポジトリの GitHub Releases から **NSIS Installer.exe** をダウンロードしてください。セットアップ画面でインストール先を選べます。最後の画面では「アプリを起動」と「デスクトップにショートカットを作成」を選べます。削除するときは Windows の「インストールされているアプリ」またはスタートメニューを使用してください。

Windows 10/11 と [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) が必要です。地図データはインストーラーに含まれますが、最新の情報や観測点のデータを受信するにはインターネット接続が必要です。ビルドには現在デジタル署名がないため、Windows の SmartScreen に警告が表示される場合があります。

読み上げを使う場合は、棒読みちゃんを別途起動し、ローカル TCP 接続 `127.0.0.1:50001` を利用できる状態にしてください。棒読みちゃんがなくても画面表示は利用できます。通知音は **メニュー → 通知音の設定** から変更できます。

### 情報源

- **気象警報・注意報、台風、津波：** 気象庁の公開 XML。地震情報の配信元を変更しても、気象情報は引き続き気象庁から受信します。
- **地震情報：** 初期設定は気象庁です。対応する契約と API キーがある場合は、**メニュー → データ配信元** から DMDATA を選択できます。キーは起動中のメモリにのみ保持します。
- **緊急地震速報：** Wolfx をバックアップとして使用し、利用可能な DMDATA の接続がある場合はそちらを優先します。気象情報のみの契約では EEW を利用できない場合があります。
- **揺れ検知：** 防災科研の強震モニタ画像。色から求めた値は推定値で、ノイズの影響を受けることがあります。公式の地震情報として扱わないでください。
- **地図：** 同梱の geoBoundaries ADM1 タイルと気象庁の地震・津波区域データ。

接続やデータ取得のエラーは、確認できる範囲で画面に表示します。灰色やデータの欠落は「安全」を意味しません。

### ソースからのビルド

必要なもの：Windows、Go **1.25 以降**、Node.js **22.13 以降**と npm、WebView2 Runtime。インストーラーの作成には NSIS も必要です。

```powershell
npm ci
npm run build:go
go test ./internal/...
cd desktop
go build -tags "desktop,production" -ldflags "-H windowsgui" -o "build/bin/NEQ-Disaster Prevention.exe" .
```

デスクトップ版を実行する際は、実行ファイルと同じフォルダーに `web-dist` と `config` を置いてください。ローカルのブラウザーでサンプル表示を試す場合は、プロジェクトのルートで次を実行します。

```powershell
go run ./cmd/warning-map -mode demo -bouyomi=
```

`http://127.0.0.1:8080` を開いてください。デモモードはサンプルデータを使用し、**現在の情報ではありません**。NSIS の定義ファイルは [`installer/NEQ-Disaster-Prevention.nsi`](installer/NEQ-Disaster-Prevention.nsi) にあります。実行ファイル、`web-dist`、`config` を一つの作業フォルダーに配置してから `makensis` でコンパイルします。

### フォルダー構成

- `app/`：React/MapLibre の画面と地図レイヤー。
- `internal/jma/`、`internal/eew/`、`internal/server/`：情報の解析・取得、状態管理、読み上げ、ローカル API。
- `desktop/`：Wails/WebView2 による Windows アプリ。
- `public/`：フロントエンドのビルドで `web-dist` にコピーする地図データ。
- `config/`：区域・観測点の対応表とデータのライセンス表記。
- `installer/`：Windows インストーラーの定義ファイル。

### 出典とライセンスの状況

地図の境界データには [geoBoundaries gbOpen](https://www.geoboundaries.org/)（[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)）を使用しています。地震・津波の区域形状は[気象庁の GIS データ](https://www.data.jma.go.jp/developer/gis.html)に基づきます。強震モニタの観測点・色の対応に関する表記は `config/KMONI-STATIONS-LICENSE.txt` と `config/KMONI-COLOR-LICENSE.txt` にあります。強震モニタ由来の情報を再配布する前に、[防災科研の利用条件](https://www.kyoshin.bosai.go.jp/ja/about_kmoni/)を確認してください。

このリポジトリには、現時点でプロジェクト全体に適用される LICENSE ファイルはありません。一部のデータに個別のライセンスがあっても、アプリのソースコード全体の再配布が許可されたことにはなりません。
