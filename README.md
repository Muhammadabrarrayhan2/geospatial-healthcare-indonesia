# Healthcare Service Distribution in Indonesia — A Geospatial Analysis

> Mapping inequality in Indonesia's hospital infrastructure across 38 provinces, combining **geospatial analysis**, **composite indexing**, and **interactive cartography** to surface where healthcare access is concentrated — and where it is not.

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![GeoPandas](https://img.shields.io/badge/GeoPandas-139C5A?logo=python&logoColor=white)
![Folium](https://img.shields.io/badge/Folium-77B829?logo=leaflet&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

---

## Table of Contents

- [Motivation](#motivation)
- [Key Findings](#key-findings)
- [Dataset](#dataset)
- [Methodology](#methodology)
  - [1. Data Cleaning & Aggregation](#1-data-cleaning--aggregation)
  - [2. Healthcare Access Index (HAI)](#2-healthcare-access-index-hai)
  - [3. Geospatial Integration](#3-geospatial-integration)
  - [4. Visualization](#4-visualization)
- [Results](#results)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Installation & Usage](#installation--usage)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Author](#author)

---

## Motivation

Indonesia is the world's fourth-most-populous country, spread across more than 17,000 islands and 38 provinces. Healthcare infrastructure, however, is not spread evenly — it is concentrated along the Java corridor, where economic activity and population density are highest. Provinces in Eastern Indonesia (Papua, Maluku, Nusa Tenggara) consistently lag behind in hospital counts, bed capacity, and medical workforce.

This project quantifies that disparity using the national hospital registry. Rather than relying on a single headline metric (e.g., "hospitals per capita"), it builds a **composite Healthcare Access Index (HAI)** that integrates four dimensions — facility count, bed capacity, workforce size, and service breadth — and visualizes the result on an interactive choropleth map of Indonesia.

The goal is not just analytical, but actionable: to produce a policy-relevant artifact that can inform resource-allocation decisions and highlight underserved regions.

---

## Key Findings

- **Java dominance.** West Java, East Java, Central Java, and DKI Jakarta together account for the overwhelming majority of healthcare capacity. The top three provinces alone (HAI > 0.89) score more than **5× higher** than the 10th-ranked province.
- **Structural concentration, not just population.** Even after normalization, the concentration persists — suggesting infrastructure investment, not just demographic weight, is the driver.
- **Eastern Indonesia gap.** Papua Pegunungan, Papua Selatan, and Papua Barat Daya register the lowest absolute capacity across every dimension measured.
- **Sector composition.** Private hospitals (`SWASTA/LAINNYA`, 869 facilities) outnumber regency-government (`Pemkab`, 648) and corporate (`Perusahaan`, 514) hospitals — raising equity questions about privately-financed care delivery.

---

## Dataset

- **Source:** Indonesian national hospital registry (`Hospital_Indonesia_datasets.csv`)
- **Size:** 3,155 hospitals × 12 attributes
- **Coverage:** All 38 provinces
- **Geometry:** Simplified Indonesia provincial boundaries — [superpikar/indonesia-geojson](https://github.com/superpikar/indonesia-geojson)

### Attributes Used

| Column | Description |
|--------|-------------|
| `nama` | Hospital name |
| `propinsi` | Province |
| `kab` | Regency / city |
| `jenis` | Hospital type (General, Maternal, Mental, etc.) |
| `kelas` | Class (A, B, C, D, D-Pratama) |
| `kepemilikan` | Ownership (Private, Regency, Corporate, Ministry, Military, etc.) |
| `total_tempat_tidur` | Total beds |
| `total_layanan` | Total services offered |
| `total_tenaga_kerja` | Total healthcare workforce |

---

## Methodology

### 1. Data Cleaning & Aggregation

- Column names standardized (lowercase, underscore-separated).
- Province names uppercased and stripped to enable robust merging with the GeoJSON boundary file.
- Per-province aggregates computed via `groupby()`: facility count, total beds, total workforce, total services.

### 2. Healthcare Access Index (HAI)

To avoid the pitfalls of any single metric (hospital counts favor large provinces; bed ratios favor urban provinces), a composite index was built:

```
HAI(province) = 0.25 × RS_scaled
              + 0.25 × Bed_scaled
              + 0.25 × Workforce_scaled
              + 0.25 × Services_scaled
```

Each component is **Min-Max normalized to [0, 1]** using `sklearn.preprocessing.MinMaxScaler`, ensuring no single dimension dominates through raw scale differences. Equal weighting (0.25 each) was chosen as a neutral prior — future iterations could use domain-weighted or population-normalized schemes.

### 3. Geospatial Integration

- The Indonesia provincial GeoJSON was loaded via `geopandas`.
- Province-name harmonization was handled through uppercasing, stripping, and manual inspection of set-difference mismatches (Aceh, Banten, Papua-family provinces, etc.) to diagnose naming drift between the administrative registry and the simplified geometry.
- Merging was performed via `gdf.merge(score_df, left_on="Propinsi", right_on="propinsi", how="left")`, with missing values imputed as 0 to preserve map completeness.

### 4. Visualization

Three visual artifacts were produced:

1. **Static choropleth** — hospital counts (`Reds` colormap) — highlights raw volume.
2. **Static HAI map** — composite index (`RdYlGn` colormap) — surfaces access quality.
3. **Interactive Folium map** — Leaflet-based, zoomable, hoverable; suitable for embedding in dashboards or reports.

Supporting charts include bar plots of hospitals per province, top-20 regencies by hospital count, ownership distribution, class distribution, and bed-capacity histograms.

---

## Results

### Top 10 Provinces by Healthcare Access Index

| Rank | Province | Hospitals | Beds | Workforce | Services | **HAI** |
|-----:|----------|----------:|-----:|----------:|---------:|--------:|
| 1 | Jawa Barat | 421 | 59,802 | 113,887 | 18,649 | **0.930** |
| 2 | Jawa Timur | 432 | 52,252 | 116,483 | 18,109 | **0.910** |
| 3 | Jawa Tengah | 355 | 77,753 | 102,862 | 16,296 | **0.894** |
| 4 | DKI Jakarta | 188 | 30,357 | 82,785 | 10,859 | **0.525** |
| 5 | Sumatera Utara | 202 | 23,813 | 38,964 | 7,281 | **0.370** |
| 6 | Banten | 129 | 15,846 | 29,083 | 5,892 | **0.261** |
| 7 | Sulawesi Selatan | 123 | 16,710 | 31,709 | 4,802 | **0.251** |
| 8 | Bali | 79 | 9,203 | 22,833 | 3,998 | **0.171** |
| 9 | Sumatera Selatan | 87 | 10,366 | 23,931 | 3,151 | **0.170** |
| 10 | Aceh | 79 | 11,030 | 24,643 | 2,833 | **0.165** |

### Interpreting the Gap

The HAI drops from **0.93 → 0.17** between the 1st and 10th-ranked province — a five-fold decline. The gap is even steeper further down the ranking, with Papua-region provinces scoring near zero. This pattern confirms what the descriptive statistics suggest: Indonesia's healthcare system is effectively a **two-tier system** split between Java and the rest of the archipelago.

---

## Tech Stack

**Data & Analysis**
- `pandas`, `numpy` — tabular manipulation
- `scikit-learn` — `MinMaxScaler`, `KMeans` (exploratory)

**Geospatial**
- `geopandas` — spatial joins and provincial boundary handling
- `shapely`, `pyproj`, `pyogrio` — geometry backend
- `mapclassify` — choropleth classification schemes

**Visualization**
- `matplotlib`, `seaborn` — static plots
- `folium` — interactive Leaflet maps
- `branca` — colormap and legend utilities

---

## Project Structure

```
geospatial-healthcare-indonesia/
├── healthcare-access-analysis-indonesia.ipynb   # Main notebook (full pipeline)
├── data/
│   └── Hospital_Indonesia_datasets.csv          # National hospital registry (3,155 rows)
├── outputs/
│   ├── hospitals_per_province.png
│   ├── hai_choropleth.png
│   └── interactive_map.html
├── requirements.txt
└── README.md
```

---

## Installation & Usage

### Prerequisites
- Python 3.10+
- pip

### Setup

```bash
git clone https://github.com/Muhammadabrarrayhan2/geospatial-healthcare-indonesia.git
cd geospatial-healthcare-indonesia

python -m venv venv
source venv/bin/activate          # Linux/Mac
venv\Scripts\activate             # Windows

pip install -r requirements.txt
```

### Run the Analysis

```bash
jupyter notebook healthcare-access-analysis-indonesia.ipynb
```

Or open directly in Google Colab:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Muhammadabrarrayhan2/geospatial-healthcare-indonesia/blob/main/healthcare-access-analysis-indonesia.ipynb)

---

## Limitations

This analysis should be read with several caveats in mind:

- **Absolute, not per-capita.** The HAI measures total capacity, not capacity-per-population. A province's low score may reflect small population rather than poor access. Integrating BPS population data is the most important next step.
- **No quality dimension.** The index counts beds and staff but says nothing about medical outcomes, wait times, insurance coverage, or patient satisfaction.
- **Province-level granularity.** Intra-province disparities (e.g., Jakarta Pusat vs. Jakarta's outer islands) are invisible at this aggregation. Regency-level analysis would surface these.
- **Equal-weight HAI is a choice, not a given.** Giving workforce (the scarcest resource) a higher weight might produce a materially different ranking.
- **Static snapshot.** The registry is a single point in time and does not capture investment trends, closures, or new facility openings.
- **Boundary-file gaps.** The simplified GeoJSON uses older province naming (pre-Papua split), requiring manual harmonization that may under-represent newer provinces (Papua Pegunungan, Papua Selatan, Papua Barat Daya, Papua Tengah).

---

## Future Work

**Analytical extensions**
- Population-normalized HAI using BPS provincial demographics (per-capita beds, per-capita workforce).
- Travel-time accessibility analysis using OpenStreetMap road networks and population-weighted centroids — i.e., *how far is the average citizen from the nearest class-A/B hospital?*
- Clustering provinces into access tiers (`KMeans`, `HDBSCAN`) and comparing against GDP per capita or Human Development Index (IPM).
- Sensitivity analysis on HAI weights to test whether the Java-first ranking is robust to reweighting.

**Data integrations**
- Puskesmas (community health center) data to complement hospital-level analysis.
- Health-outcome data (maternal mortality, stunting prevalence, life expectancy) to test whether infrastructure capacity actually predicts outcomes.
- Longitudinal registry snapshots to measure capacity growth over time.

**Deployment**
- Streamlit or Dash dashboard with province-selector, adjustable HAI weights, and downloadable reports.
- Publish the interactive Folium map as a standalone GitHub Pages site.

---

## Author

**Muhammad Abrar Rayhan**
Information Systems Undergraduate · Telkom University Jakarta
Research interests: applied ML for public-health and infrastructure problems in Indonesia

- GitHub: [@Muhammadabrarrayhan2](https://github.com/Muhammadabrarrayhan2)

---

## License

Released under the MIT License. Hospital registry data is public and attributed to its original source; the Indonesia GeoJSON is used under the terms of the [superpikar/indonesia-geojson](https://github.com/superpikar/indonesia-geojson) repository.

---

<div align="center">

*Built to make inequality in Indonesian healthcare infrastructure easier to see — and harder to ignore.*

</div>
