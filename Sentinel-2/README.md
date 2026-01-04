### SENTINEL-2 SNAP-BASED PREPROCESSING WORKFLOW

===========================================

This folder contains a fully reproducible, SNAP-based preprocessing
workflow for Sentinel-2 Level-2A data, implemented as a Jupyter notebook.

The workflow was developed to prepare analysis-ready raster datasets
(reflectance, biophysical variables, textures) for study plots across
the Iberian Peninsula, as part of the BLS–BIOMASS project.

The implementation is intentionally provided as an explicit notebook,
not as a Python package.


Why a notebook?
---------------

• Processing parameters differ across studies and sites
• SNAP operators evolve (e.g. BiophysicalOp, GLCM, Collocate)
• Assumptions must remain visible and editable
• Reproducibility matters more than abstraction

This is a workflow, not a productized library.


-----------------------------------------------------------------------
SCOPE & NON-GOALS
-----------------------------------------------------------------------

IN SCOPE

• Sentinel-2 L2A ingestion via SNAP GPT
• Surface reflectance extraction and resampling
• Scene Classification Layer (SCL) handling
• Biophysical variables (LAI, FAPAR, FCOVER, Cab, Cw)
• Cloud-masked temporal compositing
• Texture feature extraction (GLCM)
• Export to GeoTIFF
• Dataset flattening (band-wise products)
• Quality accounting (SCL class tallies)
• Reproducible cleanup policies

OUT OF SCOPE

• Machine learning
• PCA, embeddings, or feature engineering
• Feature scaling or normalization
• Model training or inference
• Statistical interpretation
• Scientific claims about vegetation or disturbance

This workflow prepares data. Interpretation happens elsewhere.


-----------------------------------------------------------------------
SOFTWARE DEPENDENCIES
-----------------------------------------------------------------------

Required

• ESA SNAP (Desktop + GPT)
• GDAL (CLI tools: gdal_translate, gdalwarp)
• Python ≥ 3.9

Python packages

• numpy
• rasterio
• geopandas (only if AOI clipping is enabled)

Not required

• scikit-learn
• pandas
• xarray
• any machine learning frameworks


-----------------------------------------------------------------------
DATA ASSUMPTIONS
-----------------------------------------------------------------------

• Input data: Sentinel-2 L2A SAFE folders or SAFE.zip
• CRS:
  – Portugal: EPSG:3763
  – Spain:    EPSG:25830
• Target resolution: 25 m
• Tile-based processing (one tile per run)
• Scene-wise processing followed by temporal aggregation


-----------------------------------------------------------------------
NOTEBOOK STRUCTURE (BLOCK-BY-BLOCK)
-----------------------------------------------------------------------

CELL 0 — GLOBAL CONFIG
---------------------

Defines:

• Study area (PLOT)
• Tile ID
• Year
• Input roots (SAFE / zipped SAFE)
• CRS and resolution
• Orbit filtering (optional)
• Naming conventions
• Publishing mode (dev / release)

This is the only cell that must be edited to change study area.


CELL 1 — SENTINEL-2 INGESTION & SPLITTING
----------------------------------------

Uses SNAP GPT to:

• Read SAFE / SAFE.zip
• Extract:
  – Surface reflectance bands
  – Biophysical variables
  – Scene Classification Layer (SCL)
• Resample all outputs to the target resolution
• Write one SNAP .dim product per scene per category

Outputs (under work/<PLOT>/<TILE>/):

• REF25/*.dim
• BIO25/*.dim
• SCL25/*.dim

No GeoTIFF export occurs at this stage.


CELL 2 — BIOPHYSICAL TEMPORAL COMPOSITE
--------------------------------------

• Reads all per-scene BIO .dim products
• Standardizes band naming
• Averages valid observations across time
• Produces a 5-band temporal composite:
  LAI, FAPAR, FCOVER, Cab, Cw

Output:

• BIO temporal composite (.dim)
• Optional GeoTIFF export


CELL 3 — COLLOCATION (REF + SCL STACK)
--------------------------------------

• Merges reflectance and SCL per date
• Collocates all dates into a single stack
• Ensures SCL masks travel with reflectance

This step guarantees explicit temporal masking
and controlled resampling behavior.


CELL 4 — TEMPORAL COMPOSITE (REFLECTANCE)
-----------------------------------------

• Applies SCL-based masking
• Computes per-band temporal means
• Produces a 12-band reflectance composite

Output:

• COMPOSITE25/*_REF25_TC.dim


CELL 5 — GLCM TEXTURE FEATURES
------------------------------

• Computes texture metrics on the reflectance composite
• One GLCM run per band
• Metrics include:
  Contrast, Dissimilarity, Homogeneity, ASM,
  Energy, MAX, Entropy, Mean, Variance, Correlation

Output:

• GLCM25/*.dim


CELL 6 — EXPORT & DOCUMENTATION
-------------------------------

• Converts selected .dim products to GeoTIFF
• Generates README_bandmap.txt
• Uses a dim-first policy (no TIFF metadata reads)

Deliverables created:

• S2TC/*.tif
• BIO/*.tif
• GLCM/*.tif
• DATASET/README_bandmap.txt


CELL 7 — QUALITY ASSURANCE (SCL TALLY)
-------------------------------------

• Reads SCL rasters directly (.img)
• Computes per-scene and aggregated class fractions
• Reports:
  – cloud fraction
  – masked fraction
  – dark pixel fraction

Deliverable:

• QA/*_SCL_TALLY_*.csv

This block provides transparency, not filtering.


CELL 8 — DATASET BUILDER (FLATTENING)
------------------------------------

• Splits multiband GeoTIFFs into band-wise files
• Reprojects with GDAL
• Applies variable-appropriate resampling
• Produces one file per variable

Final structure:

Deliverables/<REGION>/<TILE>/DATASET/
  ├─ <Study>_<Year>_S2_<VAR>_<TILE>.tif
  ├─ inventory CSV
  └─ README_bandmap.txt


CELL 9 — CLEANUP (PUBLISH-MODE AWARE)
------------------------------------

Cleans intermediates and optionally prunes deliverables.

Modes:

• dev     → keep all intermediates
• release → keep only DATASET + QA + README

This ensures reproducibility during development
and minimal artifacts for publication or sharing.


-----------------------------------------------------------------------
WHAT THIS WORKFLOW IS (AND IS NOT)
-----------------------------------------------------------------------

This workflow:

✓ Is transparent
✓ Is reproducible
✓ Is editable
✓ Respects SNAP operator semantics
✓ Produces analysis-ready datasets

This workflow:

✗ Is not a model
✗ Does not infer ecology
✗ Does not rank features
✗ Does not hide assumptions


-----------------------------------------------------------------------
INTENDED USE
-----------------------------------------------------------------------

• Remote sensing preprocessing
• Biomass or structure modeling (external)
• Fire, drought, or land-cover studies
• Dataset preparation for ML pipelines
• Teaching reproducible RS workflows


-----------------------------------------------------------------------
FINAL NOTE
-----------------------------------------------------------------------

If you need a package, build one on top of this.
If you need a model, apply it downstream.
If you need interpretation, that belongs in the paper.

This repository stops exactly where preprocessing should stop.
