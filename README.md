# E-BEWS USA — Hantavirus Digital Epidemiology Command Center

[![Pages](https://img.shields.io/badge/demo-GitHub%20Pages-0ea5e9)](https://<your-github-username>.github.io/ebews-usa-dashboard/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Data: Google Health Trends API](https://img.shields.io/badge/data-Google%20Health%20Trends-fbbf24)](https://support.google.com/trends)
[![Live overlay: ArcGIS](https://img.shields.io/badge/live-ArcGIS%20FeatureServer-a78bfa)](https://services1.arcgis.com/wb4Og4gH5mvzQAIV/arcgis/rest/services/Tracking_Hantavirus_2026/FeatureServer/1)

A single-file, animated, real-time public-health command-center dashboard for **Hantavirus** in the United States, built on DMA-resolution **Google Health Trends v1alpha** search probabilities and live-fused with the **ArcGIS Tracking_Hantavirus_2026 FeatureServer**.

> **Disclaimer.** Independent research dashboard. Counts are tentative and may change as the outbreak develops. For situational awareness only — *not* medical advice.

---

## Why this exists

Traditional surveillance reports **confirmed cases** (lagging). E-BEWS reports **behavioral probabilities** (leading) at the resolution at which media interventions are actually purchased (Nielsen DMA), and resurrects rural signal that Google's **k-anonymity** filter normally suppresses.

| Feature      | Traditional surveillance        | E-BEWS (this repo)                                      |
|--------------|---------------------------------|---------------------------------------------------------|
| Data source  | Hospital / lab reports          | Google Health Trends API v1alpha + ArcGIS overlay       |
| Metric       | Confirmed cases (lagging)       | Behavioral probability (leading)                        |
| Resolution   | County / State                  | Nielsen DMA (market-specific)                           |
| Privacy      | Subject to reporting delays     | Overcomes k-anonymity via **Grouped Expressions**       |
| Outcome      | Intervention (reactive)         | Digital Semantic Defense (predictive)                   |

See [docs/METHODOLOGY.md](docs/METHODOLOGY.md) for the full methodology.

---

## Live demo

A static `index.html` is enough to run the dashboard. GitHub Pages serves it directly:

> **https://<your-github-username>.github.io/ebews-usa-dashboard/**

Or open `index.html` locally — no build, no dependencies, no Node.

---

## What's in the dashboard

- **Animated playback engine** — virtual-time index that auto-scrubs through the data window (May 1 → May 10 in the bundled CSV). Play / pause / reset, four speeds, gradient scrubber.
- **Live ticker** — derived from the data + the ArcGIS overlay (PEAK · SURGE · SILENCE-BREAK · WHO · CDC · ARCGIS · ECDC · PAHO).
- **Executive cards** — Live National Σ (interpolated + count-up), national surge multiplier, most active DMA, silence-break count.
- **Digital EKG** — national Σ time-series with a moving "NOW" reference line that sweeps as playback advances. Marks the May 1–2 rural-silence band and the peak day.
- **Public Health Advisory** — auto LOW / ELEVATED / HIGH from the live national mean.
- **ArcGIS Live Case Overlay** — polls `Tracking_Hantavirus_2026/FeatureServer/1` every 60 s, animated count-up for CONFIRMED / DECEASED / SUSPECTED / MONITORING. Manual ⟳ refresh + CORS-safe fallback link.
- **Geospatial heatmap** — custom SVG equirectangular projection of the Western US, interpolated markers, breathing halos for high-risk DMAs, Four Corners + Pacific NW reservoir overlays.
- **Top-15 DMAs by 10-day Σ** + **Surge Velocity** (signed Δ today vs prior).
- **DMA × Date intensity matrix** — full 41 × 10 SVG heatgrid with an active-column highlight that tracks the playback cursor.
- **Silence-Break Tracker** — every DMA with baseline = 0 that surged ≥ May 3.
- **Outbreak context** — MV Hondius / Andes virus cluster (Ushuaia → Atlantic transit → European returnees → USA digital echo).
- **Methodology + 43 sources** — every source organised into 6 categories (live data · health agencies · wires · global press · regional press · science).

See [docs/FEATURES.md](docs/FEATURES.md) for the full panel inventory.

---

## Repo layout

```
ebews-usa-dashboard/
├── index.html                       # The dashboard (single self-contained HTML)
├── dashboard-mock.html              # Earlier synthetic-pulses prototype (kept for reference)
├── data/
│   ├── Hantavirus_DMA_Master_Merge.csv     # Master merged CSV (Date, SearchInterest, DMA, Term)
│   └── raw/                                # 41 per-DMA raw exports from the v1alpha API
├── scripts/
│   ├── fetch_health_trends.py             # Google Health Trends fetcher (sample1 batches)
│   └── merge_dma_csvs.py                  # Concatenates per-DMA CSVs into the master merge
├── docs/
│   ├── METHODOLOGY.md
│   ├── FEATURES.md
│   └── SOURCES.md
├── .github/workflows/pages.yml            # GitHub Pages auto-deploy
├── CITATION.cff
├── CHANGELOG.md
├── LICENSE
└── README.md
```

---

## Quick start

```bash
git clone https://github.com/<your-github-username>/ebews-usa-dashboard.git
cd ebews-usa-dashboard
# Just open it. No build step.
xdg-open index.html        # Linux
open    index.html         # macOS
start   index.html         # Windows
```

Or serve it through any static HTTP server (some browsers gate `fetch()` to the ArcGIS endpoint at `file://`):

```bash
python3 -m http.server 8000
# → http://localhost:8000/
```

---

## Updating the data

The dashboard reads the embedded CSV inside `index.html` (so it works at `file://`). To refresh:

```bash
# 1) Pull fresh per-DMA exports from the Google Health Trends v1alpha endpoint
python3 scripts/fetch_health_trends.py

# 2) Merge them into one master CSV
python3 scripts/merge_dma_csvs.py

# 3) Re-embed the CSV in index.html (replace the inline block between
#    `const CSV = \`Date,SearchInterest,DMA,Term\` and the closing backtick).
```

A small helper to re-embed is on the to-do list (see [issues](../../issues)).

---

## Architecture

- **Single HTML file** — React 18, Recharts, Tailwind, lucide, all via CDN.
- **No server** — runs at `file://` or any static host.
- **Animation engine** — `useRAF` + `usePlayback(n)` advances a virtual time index; `snapshot` and `nationalNow` are linearly interpolated between adjacent dates.
- **ArcGIS live overlay** — `useArcGIS(intervalMs)` polls the public FeatureServer every 60 s, buckets features by `STATUS`.
- **DMA name reconstruction** — the upstream merge step splits DMA labels on whitespace; the dashboard restores canonical Nielsen DMA names (e.g. `Albuquerque-Santa` → *Albuquerque-Santa Fe NM*) via a lookup table that also carries lat/lng and region.

---

## Citing

If you use this dashboard, see [CITATION.cff](CITATION.cff) or:

> Akshaya. *E-BEWS USA — Hantavirus Digital Epidemiology Command Center.* v1.0.0, 2026. https://github.com/<your-github-username>/ebews-usa-dashboard

---

## License

[MIT](LICENSE).
