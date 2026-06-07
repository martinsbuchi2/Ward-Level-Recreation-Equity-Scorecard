# Neighbourhood-Level Recreation Equity Scorecard — Edmonton
**Author:** Martins  
**Date:** May 12, 2026  
**City:** Edmonton, Alberta, Canada  
**Software:** QGIS (EPSG:3776 — NAD83 / Alberta 10-TM Forest)  
**Project File:** `Neighbourhood_Equity_Scorecard.qgz`

---

## 1. Project Overview

This project delivers a **neighbourhood-level recreation equity scorecard** for
Edmonton's 400 OSM neighbourhoods by spatially joining 340 in-city community arts
and recreation resources to real polygon boundaries, then computing service density,
category diversity, and a composite equity score for each neighbourhood. It replaces
the earlier convex hull ward proxy approach with actual OSM administrative boundaries
fetched via the QuickOSM plugin.

---

## 2. Why Neighbourhoods Instead of Wards

Edmonton's 12 civic wards were renamed with Indigenous language names in 2021
(O-day'min, Métis, papastew, etc.). These boundaries **do not exist in OpenStreetMap**
at any admin_level. Extensive querying via QuickOSM confirmed this — admin_level=10
returns 426 Edmonton neighbourhoods, but no ward-level polygons are available. The
neighbourhood layer is the finest real boundary available from OSM and provides
a more granular and geographically accurate analysis than the convex hull ward proxy.

---

## 3. Input Data

| Layer | Source | Features | Geometry | CRS |
|-------|--------|----------|----------|-----|
| Community Resources | 211 Alberta | 502 total / 340 in-city | Point | EPSG:3857 |
| Neighbourhoods | OSM via QuickOSM (`admin_level=10`) | 426 raw / 400 clipped | Polygon | EPSG:4326 → 3776 |
| City Boundary | Project layer | 1 | Polygon | EPSG:3776 |

---

## 4. Methodology

### 4.1 Data Preparation
All layers reprojected to EPSG:3776. Neighbourhoods clipped to city boundary
(426 → 400). Resources filtered to exclude null or "Outside Edmonton" `civic_ward`
values (502 → 340 in-city records).

### 4.2 Spatial Resource Counting
A spatial index (`QgsSpatialIndex`) was built on the resource points. For each
neighbourhood polygon, candidate points were retrieved from the index via bounding
box intersection, then confirmed with `contains()` geometry test. This correctly
assigns each point to exactly one neighbourhood — no double-counting.

- 337 of 340 resources mapped to a neighbourhood
- 3 resources fall in boundary gaps between neighbourhoods and were unmapped
- 77 of 400 neighbourhoods received at least one resource
- 323 neighbourhoods received zero resources

### 4.3 Scoring Metrics

**Service Density:** `total_res / area_km2`

**Category Diversity (Shannon Entropy):**
```
H = -Σ (p_i × ln(p_i)) / ln(6)     where p_i = count_cat_i / total_res
```
Returns 0–1; wards with resources in all six categories evenly score near 1.0.

**Composite Equity Score:**
```
equity_score = (norm_density × 0.60 + norm_diversity × 0.40) × 10
```
Min-max normalisation applied across all 400 neighbourhoods independently for
density and diversity.

### 4.4 Tier Classification
Percentile thresholds computed **from the 77 active neighbourhoods only** —
neighbourhoods with zero resources are classified as "No Resources" and excluded
from threshold calculation to prevent tier collapse.

| Tier | Threshold | Count |
|------|-----------|-------|
| 🟢 Well-Served | score ≥ 2.62 (top third of active) | 27 |
| 🟡 Moderately Served | score ≥ 1.66 | 25 |
| 🔴 Underserved | score < 1.66 | 25 |
| ⬜ No Resources | total_res = 0 | 323 |

---

## 5. Key Findings

### Top 5 Neighbourhoods
| Rank | Neighbourhood | Resources | Density | Score | Tier |
|------|--------------|-----------|---------|-------|------|
| 1 | Cromdale | 33 | 91.39/km² | 9.83 | Well-Served |
| 2 | Downtown | 81 | 34.80/km² | 6.28 | Well-Served |
| 3 | University of Alberta | 9 | 7.49/km² | 4.39 | Well-Served |
| 4 | McCauley | 10 | 6.70/km² | 4.38 | Well-Served |
| 5 | Abbottsfield | 10 | 23.91/km² | 4.02 | Well-Served |

### Key Observations
- **Cromdale** is the most resource-dense neighbourhood in Edmonton — 33 resources
  in a compact area giving a density of 91.39/km², nearly 3× the runner-up.
- **Downtown** has the most absolute resources (81) but lower density due to its
  larger footprint; still ranks second overall.
- **323 neighbourhoods (80.75%) have zero registered 211 resources** — these are
  not necessarily unserved, but no services are registered with 211 Alberta
  within their boundaries.
- The 3 unmapped resources fall in boundary gaps between OSM neighbourhood
  polygons — a known limitation of the OSM boundary layer completeness.

---

## 6. Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `nbh_name` | String | OSM neighbourhood name |
| `osm_id` | String | OSM object ID |
| `area_km2` | Double | Neighbourhood area (km², EPSG:3776) |
| `total_res` | Integer | Total resources spatially within boundary |
| `pct_total` | Double | Share of all in-city resources (%) |
| `density` | Double | Resources per km² |
| `rec_programs` | Integer | Sports & Recreation Programs |
| `social_clubs` | Integer | Social Activities & Clubs |
| `special_events` | Integer | Special Events |
| `arts_programs` | Integer | Arts & Culture Programs |
| `rec_facilities` | Integer | Sports & Recreation Facilities |
| `arts_facilities` | Integer | Arts & Culture Facilities |
| `diversity` | Double | Shannon entropy diversity index (0–1) |
| `equity_score` | Double | Composite equity score (0–10) |
| `equity_rank` | Integer | City-wide rank (1 = highest score) |
| `equity_tier` | String | Well-Served / Moderately Served / Underserved / No Resources |

---

## 7. Output Files

```
Ward-Level Recreation Equity Scorecard/
├── Neighbourhood_Equity_Scorecard.qgz      ← QGIS project
├── neighbourhood_equity_scorecard.gpkg
│   ├── neighbourhood_scorecard             ← Primary output
│   ├── resources                           ← Community resources
│   ├── city_boundary                       ← Edmonton boundary
│   └── neighbourhoods_osm                  ← Raw OSM neighbourhoods
└── README.md
```

---

## 8. Recommended Next Steps

- **Source official ward boundaries** from Edmonton Open Data portal to enable
  ward-level aggregation alongside neighbourhood granularity.
- **Investigate the 323 zero-resource neighbourhoods** — cross with population
  data to identify which are residential and genuinely underserved vs industrial
  or park areas where zero resources is expected.
- **Add population normalisation** — resources per 1,000 residents is a stronger
  equity metric than density per km² for residential areas.
- **Expand to all 502 resources** — investigate and reassign the 162 "Outside
  Edmonton" records; many may be misclassified and actually serve Edmonton residents.

---

*Generated with QGIS MCP Plugin + Claude (Anthropic) — May 12, 2026*
