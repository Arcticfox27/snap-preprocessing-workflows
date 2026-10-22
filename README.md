# SNAP Preprocessing Workflows

This repository contains **transparent, auditable remote sensing workflows** developed and used in the Iberian Peninsula within the ESA-funded **BLS-BIOMASS** project.

The repository has two distinct parts:

1. **Sensor-specific SNAP preprocessing workflows** (`ALOS-2/`, `Sentinel-1/`, `Sentinel-2/`): generalizable Jupyter notebooks that expose processing parameters, XML graph generation, and processing decisions.
2. **BLS-BIOMASS project workflows** (`BLS-BIOMASS/`): research notebooks and SNAP XML graphs covering additional raster preprocessing, dataset preparation, biomass mapping, and uncertainty-related summaries.

The focus is on **inspectable research workflows**, not packaged software or a one-click processing system. The original sensor-specific workflows and the project-specific additions are kept separate so their purposes and processing boundaries remain clear.

---
## Repository Structure

```text
snap-preprocessing-workflows/
├── ALOS-2/                 # General-purpose ALOS-2 preprocessing workflows
├── Sentinel-1/             # General-purpose Sentinel-1 preprocessing workflows
├── Sentinel-2/             # General-purpose Sentinel-2 preprocessing workflows
└── BLS-BIOMASS/            # Project-specific workflows (see below)
    ├── DataPreprocessing/
    │   ├── ALOS-2/         # Band splitting and mosaicking
    │   ├── S1-GRD/         # Band splitting and mosaicking
    │   ├── S1-SLC/         # SNAP GPT processing, XML graphs, band splitting, mosaicking
    │   ├── S2/             # Band splitting and mosaicking
    │   └── 4_GENERIC_NoData.ipynb
    ├── DatasetPreparation/
    │   ├── 10_Generate_StudyAreas_PointFile.ipynb
    │   └── 20_Features_Interpolation.ipynb
    └── BiomassMapping/
        └── 30_StudyArea_Mapping_Uncertainty.ipynb
```

The `BLS-BIOMASS/` files retain the original notebook and graph names supplied by the project contributors. They are presented as **project workflows**, rather than replacements for the sensor-specific reference workflows at the repository root.

---
## Scope and Philosophy

This repository is designed around the following principles:

- **Workflow over package**  
  The notebooks are research workflows meant to be read, inspected, adapted, and extended, rather than a Python package or command-line application.

- **SNAP-based, inspectable preprocessing**
  The original sensor-specific workflows use ESA SNAP operators through `gpt`, with XML graphs generated programmatically where needed. The BLS-BIOMASS material additionally includes supplied SNAP XML graphs and Python/GDAL-based processing steps.

- **Reproducibility without abstraction loss**  
  Processing parameters and decisions remain visible in the notebooks and graph definitions. Reusing the workflows requires appropriate input data and environment configuration.

- **Sensor-specific separation**  
  ALOS-2, Sentinel-1, and Sentinel-2 processing steps remain distinguishable; no unified abstraction is imposed on different satellite products.

- **Explicit scope boundaries**
  The original top-level sensor workflows focus on *preprocessing*. The project-specific `BLS-BIOMASS/` material extends to dataset preparation, biomass predictions using previously trained regressors, and prediction-distribution summaries.

---
## Intended Use

This repository is intended for researchers who need transparent preprocessing and mapping code, projects where parameter traceability matters, and users comfortable adapting notebooks to their own study areas, coordinate reference systems, input products, and processing environments.

It is **not** a one-click processing tool, a general remote sensing library, or a production data service.

---
## Computational Environment and Inputs

The original sensor-specific preprocessing workflows assume ESA SNAP installed and accessible via `gpt`, a Jupyter-capable Python environment with the relevant geospatial libraries (including GDAL), and sufficient disk space for intermediate SNAP products. See the corresponding sensor folders for their processing configurations.

The BLS-BIOMASS notebooks use combinations of **SNAP GPT, GDAL, NumPy, pandas, GeoPandas, SciPy, Shapely, joblib, and XGBoost**, depending on the notebook. The Sentinel-1 SLC workflow also refers to **SNAPHU** and a locally configured processing environment.

**Important reuse limitations:**

- The supplied BLS-BIOMASS notebooks contain **local Windows filesystem paths** and expect specific directory structures. These must be adapted before execution.
- The dataset-preparation and mapping notebooks depend on **external raster/vector inputs, feature-selection results, and trained regression models**. Those inputs and model files are **not included** in the supplied code package.
- The notebooks have **not been converted into a fully portable or automatically executable pipeline**. Their presence here documents the processing implementations; it does not imply that the published paper can be reproduced from this repository alone.

---
## Provenance and Use Context

The workflows were developed in connection with **Below, Litter & Shrub BIOMASS density mapping combining optical, LiDAR and SAR Earth Observation data (BLS-BIOMASS)**, an ESA-funded research project involving the **Instituto Superior de Agronomia, Universidade de Lisboa**, and collaborating institutions.

The root-level sensor folders provide reusable preprocessing references developed for study plots in Portugal and Spain. The separate `BLS-BIOMASS/` folder contains additional code used for project-specific data preparation and biomass mapping.

### Related Publication

Pereira-Pires, J. E., Guerra-Hernández, J., Anil-Lathika, A., Saarela, S., Silva, J. M. N., Fonseca, J. M., & Mora, A. (2026). **Biomass Prediction Combining Radar, Multispectral and Field Inventory Data**. *The International Archives of the Photogrammetry, Remote Sensing and Spatial Information Sciences*, **XLVIII-M-13-2026**, 41–48. https://doi.org/10.5194/isprs-archives-XLVIII-M-13-2026-41-2026

The publication describes biomass estimation for four components: aboveground biomass density (AGBD), belowground biomass density (BGBD), litter biomass density (LBD), and shrub biomass density (SBD), using satellite and ancillary predictors. This repository provides related processing and mapping code; **it does not currently include the full model-training workflow or all required data**.

---
## License

The repository's existing code is released under the **MIT License**; see [`LICENSE`](LICENSE). Reuse or redistribution of contributed material should follow the applicable permissions and attribution requirements.

---
## Notes on Extension

The original sensor-specific preprocessing workflows are intentionally conservative: they are intended to be extended as SNAP operators, satellite products, or research requirements evolve, rather than replaced. Project-specific material is maintained separately so future additions do not obscure the scope of those reference workflows.
