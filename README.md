# 🛰️ Skyledger

**Turning free satellite imagery into decision-ready answers about physical site change — starting with construction & industrial site monitoring.**

---

## Table of Contents

- [What We're Building](#what-were-building)
- [Problem, User, Use Case](#problem-user-use-case)
- [Why Satellite Data](#why-satellite-data)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Team & Roles](#team--roles)
- [12-Week Roadmap](#12-week-roadmap)
- [Week 1 Plan](#week-1-plan)
- [Data Strategy](#data-strategy)
- [Validation & Ground Truth](#validation--ground-truth)
- [Business Model](#business-model)
- [Risks](#risks)
- [Definition of Done (12 weeks)](#definition-of-done-12-weeks)
- [Getting Started](#getting-started)
- [Contributing / Conventions](#contributing--conventions)

---

## What We're Building

A **question-first change-detection platform**, not another satellite imagery viewer or GIS dashboard. A user defines a site and a question ("has construction increased here since March?"), and the system:

1. Pulls free Sentinel-2 imagery for the site,
2. Computes spectral indices across two time windows,
3. Quantifies the change,
4. Attaches an honest confidence score,
5. Returns a plain-language, decision-ready answer.

> **Not building:** a global GIS platform, a Google Earth clone, real-time alerting, custom deep learning models, or support for every industry at once. See [Non-Goals](#non-goals) below.

### Non-Goals (for the MVP)
- ❌ Custom satellites / constellations
- ❌ Generic multi-industry support
- ❌ Real-time/continuous monitoring
- ❌ SAR analysis, object detection, custom-trained models
- ❌ Large cloud infrastructure before we have users

---

## Problem, User, Use Case

| | |
|---|---|
| **Problem statement** | Small/mid-size construction, infrastructure, and solar-development teams need to know whether a remote site is progressing, stalled, or altered — but have no cheap way to check without a site visit or a paid analyst. |
| **Target user (v1)** | A project manager, independent consultant, or investor-side monitor tracking 1–20 remote sites for a small construction, EPC, or solar-development firm. |
| **Initial use case** | *"Has construction/earthwork activity changed at this site between Date A and Date B?"* |
| **Output** | Change % (built-up area), before/after visual, confidence label + reason, one-paragraph plain-language explanation. |

We deliberately chose construction/industrial monitoring over agriculture, water, or environmental monitoring as the starting vertical — full comparison and reasoning in [`docs/niche-comparison.md`](docs/niche-comparison.md).

---

## Why Satellite Data

Construction sites are large enough to be visible at 10m resolution and change in spectrally distinct ways (bare soil/concrete vs. vegetation). Free Sentinel-2 imagery revisits every ~5 days — far more frequent than any manual site-visit process, at zero data cost.

---

## Architecture

```text
User
 ↓
Question: "Has this site changed since [date]?"
 ↓
AOI (drawn polygon) + Time Period (before / after windows)
 ↓
Satellite Data Retrieval   (Sentinel-2 L2A via Copernicus STAC API, cloud-filtered)
 ↓
Preprocessing              (clip to AOI, cloud mask, reproject, stack bands)
 ↓
Feature Extraction         (NDVI, NDBI, bare-soil index)
 ↓
Change Detection           (index differencing + thresholding)
 ↓
Confidence Assessment      (cloud %, temporal gap, seasonal flag)
 ↓
Interpretation Layer       (rule-based → LLM-assisted phrasing)
 ↓
Decision-ready Insight
 ↓
Report (web view) + optional alert
```

### Layered Intelligence Model

| Layer | Contents |
|---|---|
| L1 — Raw Data | Sentinel-2 scenes |
| L2 — Geospatial Processing | Clipping, cloud masking, reprojection |
| L3 — Measurements | NDVI, NDBI, bare-soil index |
| L4 — Detection | Thresholded differencing → % change |
| L5 — Interpretation | Confidence scoring + plain-language phrasing |
| L6 — Decision Support | Direct answer to the user's question |

---

## Tech Stack

| Layer | Technology | Introduced |
|---|---|---|
| Frontend | React + MapLibre GL | Week 3 |
| Backend | Python + FastAPI | Week 3 |
| Geospatial | rasterio, rioxarray, GeoPandas, shapely, NumPy | Week 3–5 |
| Data source | Sentinel-2 L2A (Copernicus Data Space STAC API) | Week 3 |
| Database | PostgreSQL + PostGIS (SQLite fallback if setup blocks progress) | Week 4 |
| Storage | Local disk → Cloudflare R2 / Supabase (free tier) | Week 4+ |
| Interpretation | Claude API (numbers → plain-language paragraph only, never used for numeric analysis) | Week 7–8 |
| Hosting | Render / Railway / Fly.io (backend), Vercel / Netlify (frontend) — free tiers | Week 8 |

No Docker/Kubernetes, no custom-trained ML models, no paid imagery tasking in the MVP.

---

## Repository Structure

```text
space-intelligence/
│
├── frontend/            # React + MapLibre app
├── backend/              # FastAPI app, API routes, orchestration
├── data_pipeline/        # STAC queries, acquisition, filtering
├── geospatial/           # Preprocessing, indices, change detection (core library)
├── experiments/          # Dated, throwaway exploration notebooks/scripts
├── notebooks/            # Cleaner, reusable analysis notebooks
├── tests/                # Unit tests
├── docs/                 # Methodology, validation results, architecture notes
├── scripts/              # One-off utilities (downloads, migrations)
├── configs/              # Thresholds, .env.example, region presets
├── README.md
└── LICENSE
```

`models/` is intentionally omitted until we actually train or fine-tune something.

---

## Team & Roles

| Member | Primary | Secondary | Should NOT spend time on |
|---|---|---|---|
| **M1 — Remote Sensing / Geospatial** | Sentinel-2 acquisition, preprocessing, indices, change detection, validation | Methodology docs | Frontend, deployment, outreach |
| **M2 — Backend / AI-ML** | FastAPI services, DB, orchestration, LLM interpretation layer | Deployment/DevOps | Re-deriving remote sensing math, frontend styling |
| **M3 — Product / Frontend** | UI/UX, AOI map, report screen, pilot outreach | Documentation | Geospatial internals, DB schema design |

**Collaboration contract:** M1 builds a standalone Python function —
`analyze(aoi, date_before, date_after) -> result_dict` —
with zero web-stack dependency, so M2 and M3 can build against it (or a mock of it) in parallel without blocking on each other.

---

## 12-Week Roadmap

| Phase | Weeks | Goal |
|---|---|---|
| 0 — Problem validation & learning | 1–2 | Confirm the niche with real conversations; first NDVI/NDBI experiment |
| 1 — Data pipeline | 3–4 | Retrieve, clip, preprocess Sentinel-2 for any AOI end-to-end |
| 2 — Intelligence engine | 5–7 | Change detection, confidence scoring, first real report |
| 3 — Product MVP | 8–9 | Full frontend + DB + deploy publicly |
| 4 — Validation | 10–11 | Ground-truth testing, pilot users |
| 5 — Demo & packaging | 12 | Polish, document, record demo |

Full week-by-week breakdown (objectives, per-person tasks, deliverables, DoD, hours): [`docs/roadmap.md`](docs/roadmap.md).

---

## Week 1 Plan

**Saturday:** accounts (Copernicus Data Space, GitHub) → environment setup (rasterio/GDAL, FastAPI, React+MapLibre) → first Sentinel-2 scene downloaded and clipped to a test AOI → API contract sketched.

**Sunday:** first NDVI computed and visualized → stubbed `/analyze` endpoint wired to the frontend draw-box flow → outreach list of 8–10 pilot candidates drafted.

**By Sunday night:** a visualized NDVI raster from a real Sentinel-2 scene in `experiments/`, a map that can draw an AOI and call a (stubbed) backend, and a first outreach list.

Full hour-by-hour breakdown: [`docs/week1.md`](docs/week1.md).

---

## Data Strategy

| Dataset | Use | Resolution | Revisit | Access |
|---|---|---|---|---|
| **Sentinel-2 L2A** (primary) | NDVI, NDBI, change detection | 10m | ~5 days | Copernicus Data Space (free, STAC API) |
| OpenStreetMap | Ground truth (building footprints), AOI context | Vector | Irregular | Overpass API (free) |
| Sentinel-1 (SAR) | Deferred — cloud-penetrating radar for later phases | 10m | ~6 days | Copernicus Data Space |
| Landsat 8/9 | Deferred — cross-validation only | 30m | 16 days | USGS / AWS open data |

Full dataset comparison and preprocessing decisions: [`docs/data-strategy.md`](docs/data-strategy.md).

---

## Validation & Ground Truth

We validate against: OpenStreetMap footprint changes, manual visual annotation of true-color Sentinel-2 composites, and publicly announced project milestones (e.g., solar farm commissioning dates). We report a simple **agreement rate** between the system's change-flag and manual judgment — not formal precision/recall/IoU, until we have a labeled set large enough to make those meaningful.

Full experiment list and metrics rationale: [`docs/validation.md`](docs/validation.md).

---

## Business Model

**MVP model: pay-per-report.** No subscription billing infrastructure needed; matches how a small construction consultant thinks about spend. Subscriptions, API access, and enterprise custom monitoring are deferred until users ask for them unprompted.

---

## Risks

| Risk | Mitigation |
|---|---|
| Cloud cover blocks usable imagery | Cloud-aware scene selection; explicit "insufficient data" response |
| Seasonal vegetation change misread as construction | Seasonal-mismatch flag; prefer same-season comparisons |
| Overselling accuracy / false confidence | Every user-facing number traces to a computed value; confidence always shown with its reason |
| Team over-builds before validating demand | Pilot outreach starts Week 2, not Week 10 |

Full risk register: [`docs/risks.md`](docs/risks.md).

---

## Definition of Done (12 weeks)

- [ ] A user can draw/name an AOI, pick a before/after date range, and get a report back within minutes.
- [ ] The report states a quantified change (%), a confidence label, and a plain-language explanation.
- [ ] The pipeline has run successfully on 3+ real-world sites, including one real (non-team) pilot user's site.
- [ ] 3+ non-team people have used it and given logged feedback.
- [ ] Repo, README, and a 3-minute demo video are ready to show a customer, professor, or interviewer.

---

## Getting Started

```bash
git clone https://github.com/<org>/Skyledger.git
cd  Skyledger

# Backend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload

# Frontend
cd ../frontend
npm install
npm run dev

# Geospatial experiments
cd ../experiments
pip install rasterio rioxarray geopandas numpy matplotlib
```

You'll need a free [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/) account for Sentinel-2 access. Copy `configs/.env.example` to `.env` and fill in credentials before running the pipeline.

---

## Contributing / Conventions

- `main` is always deployable. Work happens in feature branches: `feature/ndbi-index`, `feature/report-screen`, etc.
- PRs get at least one other team member's review, even informally.
- Commit prefixes: `feat:`, `fix:`, `docs:`, `exp:`.
- One GitHub Milestone per phase; issues tagged `geospatial`, `backend`, or `frontend`.
- Update `docs/methodology.md` whenever thresholds or logic change — not just at the end of the project.

---

*This README is the entry point. The full strategic reasoning (niche comparison, competitive landscape, learning curriculum, cost plan) lives in [`docs/`](docs/) and in our internal planning doc.*
