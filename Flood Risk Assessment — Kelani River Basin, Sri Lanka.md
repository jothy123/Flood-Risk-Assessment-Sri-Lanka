# Flood Risk Assessment — Kelani River Basin, Sri Lanka
> Geo-Information Technology in Disaster Management | SP2409 Geo-Spatial Analysis  
> Department of Town and Country Planning, University of Moratuwa

---

## Overview

This project replicates a full GIS-based **flood risk assessment pipeline** for the Kelani River Basin in Sri Lanka using Python — replacing the original QGIS/ArcGIS workflow with open-source geospatial libraries.

Sri Lanka sits in the disaster-prone belt of the Asian region. According to the World Disaster Report 2014, Asia accounts for **48% of global disaster occurrences**, **85% of people affected**, and **86% of damage**. Within Sri Lanka, floods are the most impactful disaster type, affecting over **2.9 million people** between 2000 and 2008. The Kelani River Basin — draining through the densely populated Western Province — is identified as the highest-risk zone in the country.

---

## Study Area

| Parameter | Value |
|---|---|
| Basin | Kelani River Basin |
| Province | Western Province, Sri Lanka |
| DEM Source | ALOS 12.5 m resolution (4 tiles merged) |
| CRS | EPSG:32644 (UTM Zone 44N) |
| Flood Level (WSE) | 9.1 m (Kelani dangerous flood threshold) |
| Study Area Extent | 372,932 – 461,380 E, 757,181 – 823,660 N |

---

## Pipeline

```
4 ALOS DEM Tiles
      │
      ▼
  Merge Tiles  ──► dem_merged.tif
      │
      ▼
  Clip to Kelani Basin
      │
      ├──► DEM Pre-processing (fill pits, resolve flats)
      │          │
      │          ▼
      │    Watershed Delineation (D8 flow direction, accumulation)
      │
      ├──► Flood Hazard Mapping (bathtub inundation @ WSE 9.1 m)
      │          │
      │          ▼
      │    flood_hazard.tif  (4 classes: Very Low / Low / Moderate / High)
      │
      ├──► Vulnerability Analysis
      │     ├── Building footprints (HOT OSM)
      │     └── Population data (PTOTAL, area-weighted)
      │
      └──► Weighted Overlay Risk Map (60% Hazard + 40% Vulnerability)
                 │
                 ▼
           flood_risk.tif  (5 classes: Very Low → Very High)
```

---

## Results

| Metric | Value |
|---|---|
| Total study area cells | 32,330,244 |
| Flooded cells | 1,900,264 (5.9%) |
| High hazard cells | 1,206,108 (3.73%) |
| Buildings in study area | 696,046 |
| Vulnerable building cells | 103,543 |
| Dominant risk class | Moderate (3.44%) |

---

## Output Maps

| Map | Description |
|---|---|
| `map_flood_hazard.png` | 4-class flood hazard map |
| `map_vulnerability.png` | Vulnerability by GN Division |
| `map_flood_risk.png` | 5-class weighted overlay risk map |
| `flood_hazard.tif` | Classified hazard raster (uint8) |
| `flood_risk.tif` | Classified risk raster (uint8) |
| `dem_merged.tif` | Merged 4-tile ALOS DEM |

---

## Tech Stack

| Tool | Purpose | Replaces |
|---|---|---|
| `rasterio` | Read/write GeoTIFFs, raster clipping | ArcGIS Raster tools |
| `pysheds` | Watershed delineation, flow direction | QGIS GRASS r.watershed |
| `geopandas` | Vector data, spatial joins | ArcGIS Vector tools |
| `rasterstats` | Zonal statistics per polygon | ArcGIS Zonal Statistics |
| `numpy` | Raster algebra, weighted overlay | ArcGIS Spatial Analyst |
| `matplotlib` | Map export and visualization | ArcGIS Layout |
| `scipy` | DEM smoothing | ArcGIS Focal Statistics |

---

## Installation

```bash
# Create a dedicated environment (recommended)
conda create -n floodenv -c conda-forge python=3.11 pysheds rasterstats geopandas rasterio matplotlib scipy fiona -y
conda activate floodenv
```

---

## Project Structure

```
Flood Hazard Analysis/
│
├── 01.py                        # Full pipeline (pysheds watershed)
├── flood_risk_multitile.py      # Multi-tile DEM merge + risk mapping
│
├── data/
│   ├── ALOS_DEM_T08/            # DEM tile — SW quadrant
│   ├── ALOS_DEM_T09/            # DEM tile — SE quadrant
│   ├── ALOS_DEM_T13/            # DEM tile — NW quadrant
│   ├── ALOS_DEM_T14/            # DEM tile — NE quadrant
│   ├── Colombo/
│   │   ├── Colombo.shp          # District boundary + population (PTOTAL)
│   │   └── Colombo_Building.shp
│   └── hotosm_lka_buildings_polygons_shp/
│       └── hotosm_lka_buildings_polygons_shp.shp
│
└── outputs/
    ├── dem_merged.tif
    ├── flood_hazard.tif
    ├── flood_risk.tif
    ├── map_flood_hazard.png
    ├── map_vulnerability.png
    └── map_flood_risk.png
```

---

## Data Sources

| Dataset | Source |
|---|---|
| ALOS 12.5 m DEM | Alaska Satellite Facility (ASF Vertex) |
| Building footprints | HOT OpenStreetMap (hotosm_lka_buildings) |
| Population data | Department of Census and Statistics, Sri Lanka |
| Disaster statistics | Department of Disaster Management, Sri Lanka |
| Flood gauge data | Department of Irrigation, Sri Lanka |

---

## References

- World Disaster Report 2014 — International Federation of Red Cross
- Department of Disaster Management Sri Lanka — [dmc.gov.lk](http://www.dmc.gov.lk)
- Department of Meteorology Sri Lanka — [meteo.gov.lk](http://www.meteo.gov.lk)
- SP2409 Geo-Spatial Analysis, University of Moratuwa

---

## Author

**182327A** — Department of Town and Country Planning, University of Moratuwa  
Course: SP2409 Geo-Spatial Analysis
