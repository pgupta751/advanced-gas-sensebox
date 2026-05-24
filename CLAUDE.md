# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page interactive map client for an Advanced GIS course assignment (ETH Zürich, FS 26). Compares a mobile SenseBox walking transect against the MeteoSwiss Zürich-Fluntern fixed station for the evening of 10 May 2026 in the Sihl valley.

The entire app is one file: `index.html`. There is no build step, no package manager, no test suite, no lint config.

## Running

Open `index.html` directly in a browser, or serve the directory (e.g. `python3 -m http.server 8000`) and visit it. The ArcGIS Maps SDK v5.0 loads from the Esri CDN at runtime — an internet connection is required.

The `esriConfig.apiKey` in the `<head>` is a live ArcGIS Online API key. The two `FeatureLayer` URLs (`sensebox_sihlwalk_clean/FeatureServer/0` and `fluntern_external_may10/FeatureServer/0`) point at the author's published services on `services1.arcgis.com/VEqtF8LmJCM1Rgkd/`. If those services are unpublished or the key is rotated, the map loads but the layers will fail to render.

## Architecture

Two cooperating but independent scripts inside one HTML document:

1. **Map script (`<script type="module">`)** — uses the SDK's `$arcgis.import()` CDN helper to pull `FeatureLayer` and the smart-mapping `color` renderer module. Waits for `mapEl.viewOnReady()`, adds both layers, then calls `colorRendererCreator.createContinuousRenderer({ field: "temp_c" })` so the SenseBox point colours are derived from the actual data range rather than hand-coded class breaks. Fluntern uses a hard-coded `simple-marker` so it always reads as the reference point. The panel checkbox (`#toggle-fluntern`) binds directly to `fluntern.visible`.

2. **Chart script (plain `<script>`)** — completely standalone, no SDK dependency. Renders the temperature-vs-time SVG from two hard-coded arrays (`SB_DATA`, `FL_DATA`) embedded at the bottom of the file. The walk is a finished historical event, so these values are intentionally frozen snapshots of the published CSVs; do not try to derive them from the live FeatureLayers.

The map declares its widgets as web-component children of `<arcgis-map>` (`arcgis-zoom`, `arcgis-home`, `arcgis-basemap-toggle`, `arcgis-legend`) — this is the SDK v5 component API, not the older JS module API. Both APIs are loaded by the single `https://js.arcgis.com/5.0/` script tag.

## Layout & styling

CSS lives in a single `<style>` block in the `<head>`. The layout is a two-column CSS grid (`1fr 380px`) that collapses to stacked rows below 820 px. Theme tokens (`--ink`, `--paper`, `--warm`, `--cool`, `--accent`, `--hairline`) are defined on `:root` and reused throughout; the `.ramp` gradient and the chart's stroke colours echo the same warm→cool temperature palette, so changing a token cascades to both the panel UI and the chart.
