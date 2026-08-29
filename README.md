# LS-LRSI: Location-Specific Landslide Risk Scoring Index — Passara DSD, Badulla District, Sri Lanka

## Overview
This project computes the LS-LRSI for the Passara Divisional Secretariat Division
using the ACCIMT Ditwah-Cyclone (2025) landslide inventory. The repository includes
both the analysis notebooks and a shapefile conversion utility for preparing the input
landslide dataset.

## Requirements
Install the geospatial Python dependencies used by the notebooks:

```bash
pip install geopandas pyogrio shapely ipywidgets matplotlib pandas numpy nbformat --break-system-packages
```

Use any standard Python 3.10+ geospatial environment with geopandas >= 0.14 and shapely >= 2.0.

## Recommended run order
Run the notebooks in this exact order:

1. Open and run `Shapes File Convert to Geojson.ipynb`
2. Open and run `LS-LRSI.ipynb`
3. Open and run `LS-LRSI_analysis.ipynb`

## Step-by-step workflow

### 1) Convert the uploaded shapefile to GeoJSON and CSV
Open `Shapes File Convert to Geojson.ipynb` and run all cells from top to bottom.

This notebook:
- installs the required packages,
- opens a file uploader,
- reads the uploaded `.shp` file from `uploaded_data/`,
- exports `shp_landslides.geojson`,
- exports `shp_landslides.csv` with centroid and WKT columns.

Important: some shapefiles are missing or have a damaged `.shx` index file. The notebook sets
`SHAPE_RESTORE_SHX=YES` before reading the shapefile so GDAL can restore the missing index when possible.

The expected output files are:
- `shp_landslides.geojson`
- `shp_landslides.csv`

### 2) Run the main LS-LRSI notebook
Open `LS-LRSI.ipynb` and run all cells sequentially.

This notebook performs the main geospatial analysis:
- loads the landslide inventory,
- reprojects the dataset,
- computes polygon geometry metrics,
- defines the study area,
- builds the 1 km grid and risk calculations.

### 3) Run the analysis notebook
Open `LS-LRSI_analysis.ipynb` and run all cells sequentially.

This notebook produces the final outputs and figures in `outputs/`.

## Expected final outputs
After the full workflow completes, the project generates:

- `outputs/passara_grid_1km_index.geojson`
- `outputs/passara_grid_1km_index.csv`
- `outputs/passara_landslides.geojson`

The conversion notebook also creates:
- `shp_landslides.geojson`
- `shp_landslides.csv`

## Data layout
- `data_raw/` contains the raw inventory data
- `uploaded_data/` stores uploaded shapefile inputs
- `outputs/` stores the final grid outputs and study-area subset

## Pipeline summary
1. Load national inventory (4,225 polygons) -> reproject to UTM 44N -> compute
   area, perimeter, and compactness per polygon.
2. Diagnostic 0.05-degree hotspot grid identifies Passara DSD as the highest-density locality.
3. Subset the study area (81.05-81.20 E, 6.95-7.10 N) -> 698 landslides.
4. Build a 1 km x 1 km fishnet grid (324 cells) -> spatial join -> aggregate
   landslide count, total area, and mean compactness.
5. Normalize three indicators: Historical Landslide Density (HLD), Magnitude (MAG), and
   Channelization (CHAN).
6. Weighted sum -> LS-LRSI score (0-100): 0.50*HLD + 0.30*MAG + 0.20*CHAN.
7. Quantile-based classification -> Low / Medium / High / Very High.
8. Export scored grid and figures.

## Extending to the full 6-indicator index
Add normalized slope (SL), rainfall trigger (RF), and land-cover disturbance (LC) columns to the grid
GeoDataFrame, then update the weighted-sum formula:

LS-LRSI(6) = 100 * (0.20 HLD + 0.10 MAG + 0.10 CHAN + 0.25 SL + 0.25 RF + 0.10 LC)

Re-running for a different location only requires changing `STUDY_BBOX`.

## Data sources
See the project report for full dataset documentation, including source, link, resolution,
temporal coverage, and CRS for both the directly used inventory and the auxiliary
causative-factor layers (DEM, Sentinel-2, CHIRPS, geology, OSM, NBRO maps).

## Troubleshooting
If the shapefile upload fails with a GDAL `DataSourceError` about `.shx`, make sure all shapefile parts
are uploaded together:

- `.shp`
- `.dbf`
- `.shx`
- `.prj`
- `.cpg`

If the `.shx` file is missing or corrupted, rerun the conversion notebook after the restore setting is in place.
