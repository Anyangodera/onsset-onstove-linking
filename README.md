# OnSSET–OnStove Linking Pipeline

This repository provides a reproducible pipeline for converting
[OnSSET](https://github.com/onsset/onsset) electrification model outputs into
rasterised inputs for [OnStove](https://github.com/onstove/onstove) clean-cooking
analysis.

## Why this pipeline exists

OnSSET models least-cost electrification pathways at the settlement level and
outputs tabular CSV data. OnStove, which models clean-cooking technology
transitions, expects its electrification inputs as raster layers. This notebook
bridges the two by:

1. Reclassifying OnSSET electrification codes so that only grid and mini-grid
   connections — technologies that can realistically power electric cooking
   appliances — are counted as electrified.
2. Computing a weighted average LCOE and electrification rate for each
   settlement across the modelling horizon.
3. Merging the tabular results with spatial settlement-cluster geometries.
4. Rasterising the merged data into GeoTIFF layers at a configurable
   resolution for OnStove ingestion.

## Requirements

- Python 3.9+
- [OnStove](https://github.com/onstove/onstove) (provides `VectorLayer` and
  rasterisation utilities)
- pandas
- geopandas

Install dependencies with:

```
pip install -r requirements.txt
```

## Input data

The pipeline expects two input files:

| File | Format | Description |
|------|--------|-------------|
| OnSSET results | CSV | Settlement-level OnSSET output containing columns such as `FinalElecCode{year}`, `ElecStatusIn{year}`, `MinimumOverallLCOE{year}`, `EnergyPerSettlement{year}`, `ElecPopCalib`, `PopStartYear`, `Pop{year}`, and `id`. |
| Settlement clusters | Shapefile or GeoPackage | Spatial geometries for each settlement cluster, with an `id` column matching the OnSSET CSV. |

These files are not included in this repository. To obtain them, run an OnSSET
analysis for your country of interest and export the cluster geometries from
the OnSSET GIS processing step.

## Usage

1. Clone the repository:

   ```
   git clone https://github.com/YOUR_USERNAME/onsset-onstove-linking.git
   cd onsset-onstove-linking
   ```

2. Place your input files in `data/input/` (or any directory of your choice).

3. Open `onsset_onstove_linking.ipynb` and edit the configuration cell at the
   top to point to your files:

   ```python
   ONSSET_CSV  = "data/input/KEN-1-2_0.csv"
   CLUSTERS_SHP = "data/input/clusters.shp"
   OUTPUT_DIR   = "data/output"
   ```

4. Run all cells. Rasterised outputs will be saved to `{OUTPUT_DIR}/onsset/`.

## Outputs

The pipeline produces:

- **`onsset_elec.gpkg`** — a GeoPackage containing settlement geometries with
  the final electrification code, electrification rate, and settlement ID.
- **One GeoTIFF per attribute**, including:
  - `FinalElecCode{year}.tif` — binary grid/mini-grid electrification status
    for each year in the modelling horizon
  - `setLCOE.tif` — weighted average levelised cost of electricity
  - `ElecRate.tif` — base-year electrification rate
  - `Pop{year}.tif` — population for the start and end years

## Configuration parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `START_YEAR` | 2023 | First year of the OnSSET modelling horizon |
| `END_YEAR` | 2033 | Final year of the OnSSET modelling horizon |
| `CELL_SIZE` | 100 | Raster cell size in metres |
| `TARGET_CRS` | 3395 | Target coordinate reference system (EPSG code) |

## Repository structure

```
onsset-onstove-linking/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── onsset_onstove_linking.ipynb   # Main pipeline notebook
└── data/
    ├── input/                     # Place input files here (not tracked)
    └── output/                    # Rasterised outputs (not tracked)
```

## Citation

If you use this pipeline in your research, please cite:

```
@misc{onsset_onstove_linking,
  author  = {YOUR NAME},
  title   = {OnSSET–OnStove Linking Pipeline},
  year    = {2026},
  url     = {https://github.com/YOUR_USERNAME/onsset-onstove-linking}
}
```

## Licence

This project is licensed under the MIT Licence. See [LICENSE](LICENSE) for
details.

## Acknowledgements

This pipeline was developed as part of doctoral research on sub-national energy
planning at the University of Cape Town, in collaboration with the KTH Royal
Institute of Technology OnSSET and OnStove teams.
