# 🌊 Flood Risk Assessment Pipeline — Sri Lanka
> **SP2409 Geo-Spatial Analysis** · Department of Town and Country Planning · University of Moratuwa

---

## 📌 Background

Sri Lanka lies within one of the world's most disaster-prone regions. Asia accounts for **48% of global disaster occurrences**, **85% of people affected**, and **86% of economic damage** (World Disaster Report, 2014). Within Sri Lanka, floods are the single most impactful hazard type — affecting over **2.9 million people** between 2000 and 2008 alone.

This project builds a **fully automated, reproducible flood risk assessment pipeline** in Python, replacing a traditional QGIS / ArcGIS Model Builder workflow with open-source geospatial libraries. The pipeline was originally developed for the **Kelani River Basin** (Western Province) and later extended to cover the **entire island of Sri Lanka**.

---

## 🗺️ Study Areas

| Parameter | Kelani River Basin | Full Sri Lanka |
|---|---|---|
| Coverage | Western Province | Island-wide |
| DEM Source | ALOS 12.5 m (4 tiles merged) | SRTM / ALOS national mosaic |
| CRS | EPSG:32644 (UTM Zone 44N) | EPSG:4326 → EPSG:5235 |
| Flood Level (WSE) | 9.1 m (DMC dangerous threshold) | 12.0 m (configurable) |
| Boundary clipping | Bounding box | National boundary shapefile (GDAL) |
| GN Divisions | Colombo District | ~14,000 island-wide |

---

## ⚙️ Pipeline Architecture

```
Raw Input Data
      │
      ├── ALOS / SRTM DEM (GeoTIFF)
      ├── National / Basin boundary shapefile
      ├── GN Division boundaries
      ├── District population (PTOTAL — census)
      └── Building footprints (HOT OSM)
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 1 — Data Loading & Clipping                          │
│  Clip DEM to study boundary via GDAL (no Shapely dissolve)  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 2 — DEM Conditioning                                 │
│  Fill pits → Fill depressions → Resolve flats (pysheds)     │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 3 — Watershed Delineation                            │
│  D8 flow direction → Flow accumulation → Stream network     │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 4 — Flood Hazard Mapping                             │
│  Bathtub inundation model (WSE − DEM) → 4-class hazard map  │
│  Very Low / Low / Moderate / High  (Sri Lanka DMC thresholds)│
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 5 — Vulnerability Analysis                           │
│  Zonal stats per GN Division → Areal population             │
│  interpolation (district PTOTAL → GN estimate) →           │
│  Building exposure count → 4-class vulnerability map        │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 6 — Flood Risk Mapping (Weighted Overlay)            │
│  Risk = (0.60 × Hazard) + (0.40 × Vulnerability)           │
│  5-class output: Very Low → Very High                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  STAGE 7 — Visualisation & Export                           │
│  DEM map · Hazard map · Vulnerability map · Risk map        │
│  Summary statistics (4-panel) · All outputs @ 200 DPI       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Key Results

### Kelani River Basin (WSE = 9.1 m)

| Metric | Value |
|---|---|
| Total study area cells | 32,330,244 |
| Flooded cells | 1,900,264 (5.9%) |
| High hazard cells | 1,206,108 (3.73%) |
| Buildings in study area | 696,046 |
| Buildings in flood zone | 103,543 |
| Dominant risk class | Moderate (3.44%) |

### Full Sri Lanka (WSE = 12.0 m)

| Metric | Value |
|---|---|
| Hazard pattern | Coastal plains dominate; central highlands = zero risk |
| Highest hazard zones | Jaffna Peninsula, Mannar, Puttalam, Colombo coast, southern tip |
| Population exposure | Highest in Low–Moderate vulnerability GN Divisions |
| Risk distribution | Very Low dominates by cell count; High corridors align with populated coasts |

---

## 🗂️ Output Files

| File | Type | Description |
|---|---|---|
| `dem_clipped.tif` | GeoTIFF (float32) | DEM clipped to study boundary |
| `flow_accumulation.tif` | GeoTIFF (float32) | Upstream contributing area |
| `flood_depth.tif` | GeoTIFF (float32) | Inundation depth in metres |
| `flood_hazard.tif` | GeoTIFF (uint8) | 4-class hazard raster (1–4) |
| `vulnerability.gpkg` | GeoPackage | GN Divisions with exposure stats |
| `flood_risk.tif` | GeoTIFF (uint8) | 5-class risk raster (1–5) |
| `map_dem.png` | PNG 200 DPI | Terrain overview |
| `map_flood_hazard.png` | PNG 200 DPI | Flood hazard map |
| `map_vulnerability.png` | PNG 200 DPI | GN Division vulnerability map |
| `map_flood_risk.png` | PNG 200 DPI | Weighted overlay risk map |
| `summary_statistics.png` | PNG 200 DPI | 4-panel statistical summary |

---

## 🔢 Classification Schemes

### Flood Hazard — Sri Lanka DMC Thresholds

| Class | Depth | Colour |
|---|---|---|
| 1 — Very Low | 0 – 0.5 m | ![#FFFF99](https://placehold.co/12x12/FFFF99/FFFF99.png) `#FFFF99` |
| 2 — Low | 0.5 – 1.5 m | ![#FFC14D](https://placehold.co/12x12/FFC14D/FFC14D.png) `#FFC14D` |
| 3 — Moderate | 1.5 – 3.0 m | ![#66BB66](https://placehold.co/12x12/66BB66/66BB66.png) `#66BB66` |
| 4 — High | > 3.0 m | ![#CC0000](https://placehold.co/12x12/CC0000/CC0000.png) `#CC0000` |

### Vulnerability — % GN Division Area Flooded

| Class | Flooded Area | Colour |
|---|---|---|
| Very Low | < 10% | ![#006600](https://placehold.co/12x12/006600/006600.png) `#006600` |
| Low | 10 – 25% | ![#33AA33](https://placehold.co/12x12/33AA33/33AA33.png) `#33AA33` |
| Moderate | 25 – 50% | ![#FFA500](https://placehold.co/12x12/FFA500/FFA500.png) `#FFA500` |
| High | ≥ 50% | ![#CC0000](https://placehold.co/12x12/CC0000/CC0000.png) `#CC0000` |

### Flood Risk — Weighted Overlay Score

| Class | Score Range | Colour |
|---|---|---|
| 1 — Very Low | 0 – 1.8 | ![#4DB8FF](https://placehold.co/12x12/4DB8FF/4DB8FF.png) `#4DB8FF` |
| 2 — Low | 1.8 – 2.6 | ![#66CC66](https://placehold.co/12x12/66CC66/66CC66.png) `#66CC66` |
| 3 — Moderate | 2.6 – 3.4 | ![#FFFF00](https://placehold.co/12x12/FFFF00/FFFF00.png) `#FFFF00` |
| 4 — High | 3.4 – 4.2 | ![#FFA500](https://placehold.co/12x12/FFA500/FFA500.png) `#FFA500` |
| 5 — Very High | > 4.2 | ![#CC0000](https://placehold.co/12x12/CC0000/CC0000.png) `#CC0000` |

> **Formula:** `Risk Score = (0.60 × Hazard Class) + (0.40 × Vulnerability Class)`

---

## 🛠️ Tech Stack

| Library | Version | Purpose | Replaces |
|---|---|---|---|
| `rasterio` | ≥ 1.3 | Read/write GeoTIFFs, raster clipping (GDAL) | ArcGIS Raster tools |
| `pysheds` | ≥ 0.3 | Watershed delineation, D8 flow routing | QGIS GRASS r.watershed |
| `geopandas` | ≥ 0.14 | Vector data, spatial joins, reprojection | ArcGIS Vector tools |
| `rasterstats` | ≥ 0.19 | Zonal statistics per polygon | ArcGIS Zonal Statistics |
| `numpy` | ≥ 1.24 | Raster algebra, weighted overlay | ArcGIS Spatial Analyst |
| `matplotlib` | ≥ 3.7 | Map rendering and export | ArcGIS Layout |
| `shapely` | ≥ 2.0 | Geometry operations | ArcGIS geometry tools |
| `scipy` | ≥ 1.11 | DEM smoothing, morphological ops | ArcGIS Focal Statistics |
| `fiona` | ≥ 1.9 | Low-level shapefile I/O | — |

---

## 🚀 Installation

```bash
# Option A — Conda (recommended, handles GDAL automatically)
conda create -n floodenv -c conda-forge python=3.11 \
    pysheds rasterstats geopandas rasterio \
    matplotlib scipy fiona -y

conda activate floodenv
```

```bash
# Option B — pip (ensure GDAL is pre-installed on your system)
pip install rasterio numpy pysheds geopandas rasterstats matplotlib scipy fiona
```

---

## ⚡ Quick Start

```python
# 1. Edit CFG in the script to point to your data paths
CFG = {
    "dem_path":         "path/to/sl_dem.tif",
    "sl_boundary_path": "path/to/SL_Boundary.shp",
    "gn_division_path": "path/to/GND.shp",
    "population_path":  "path/to/Population.shp",
    "buildings_path":   "path/to/buildings.shp",
    "out_dir":          "path/to/outputs/",
    "flood_level_m":    12.0,   # ← change to model different return periods
}

# 2. Run the pipeline
python flood_risk_srilanka.py
```

> **Tip:** Change `"flood_level_m"` to model different return-period events.  
> Every map and statistic regenerates automatically — no manual steps required.

---

## 📁 Project Structure

```
Flood Hazard Analysis/
│
├── flood_risk_srilanka.py          # Full island pipeline (main script)
├── 01.py                           # Kelani River Basin pipeline
│
├── DATA_1/
│   ├── DEM/GEO_ANA/sl_dem.tif     # Sri Lanka national DEM
│   ├── SL_Boundary/SL_Boundary.shp # National boundary shapefile
│   ├── GND/GND.shp                 # GN Division boundaries
│   ├── Roads.shp                   # Road network
│   └── hotosm_lka_buildings_*/     # HOT OSM building footprints
│
├── Population/
│   └── Sri_Lanka_Population.shp    # District-level PTOTAL
│
└── outputs/
    ├── dem_clipped.tif
    ├── flow_accumulation.tif
    ├── flood_depth.tif
    ├── flood_hazard.tif
    ├── vulnerability.gpkg
    ├── flood_risk.tif
    ├── map_dem.png
    ├── map_flood_hazard.png
    ├── map_vulnerability.png
    ├── map_flood_risk.png
    └── summary_statistics.png
```

---

## 💡 Key Technical Notes

**Why GDAL clipping instead of Shapely dissolve?**  
The Sri Lanka national boundary shapefile contains minor self-intersections at coastline vertices (notably near Mannar, ~79.89°E, 8.95°N). `boundary.dissolve()` triggers a Shapely `TopologyException`. The fix passes individual feature geometries directly to `rio_mask()` — GDAL handles the raster union internally without topology validation.

**Areal interpolation for population**  
Census population (`PTOTAL`) is available at district level, but flood exposure is assessed at GN Division level (~14,000 units). The pipeline uses areal interpolation:  
`GN estimated population = District PTOTAL × (GN area / District total area)`  
This assumes uniform density within each district — a standard approximation when sub-district census data is unavailable.

**Bathtub inundation model**  
The flood model uses a simple DEM-difference approach: all land cells below the Water Surface Elevation (WSE) are classified as inundated, with depth = WSE − elevation. This is appropriate for scenario screening but does not account for hydraulic connectivity or flow barriers. For infrastructure-level decisions, coupling with a 1D/2D hydraulic model (HEC-RAS, LISFLOOD-FP) is recommended.

---

## 📚 Data Sources

| Dataset | Source | Access |
|---|---|---|
| ALOS 12.5 m DEM | Alaska Satellite Facility | [vertex.daac.asf.alaska.edu](https://vertex.daac.asf.alaska.edu) |
| Sri Lanka national DEM | SRTM / ALOS national mosaic | Provided |
| Building footprints | HOT OpenStreetMap | [data.humdata.org](https://data.humdata.org) |
| GN Division boundaries | Survey Department Sri Lanka | Provided |
| Population (PTOTAL) | Dept. of Census & Statistics, Sri Lanka | Provided |
| National boundary | GADM Level 0 | [gadm.org](https://gadm.org) |

---

## 📖 References

- IFRC (2014). *World Disaster Report*. International Federation of Red Cross and Red Crescent Societies.
- Department of Disaster Management Sri Lanka — [dmc.gov.lk](http://www.dmc.gov.lk)
- Department of Meteorology Sri Lanka — [meteo.gov.lk](http://www.meteo.gov.lk)
- Department of Irrigation Sri Lanka — flood gauge thresholds
- SP2409 Geo-Spatial Analysis — University of Moratuwa

---

## 👤 Author

**Student Index: 182327A**  
Department of Town and Country Planning  
University of Moratuwa, Sri Lanka  
Course: SP2409 Geo-Spatial Analysis

---

<p align="center">
  <i>Built with open-source Python · No proprietary GIS software required</i>
</p>
