### ALOS-2 PALSAR-2 SNAP+GDAL PREPROCESSING WORKFLOW

===========================================

This folder contains a fully reproducible preprocessing workflow for ALOS-2 PALSAR-2 backscatter products, implemented as a Jupyter notebook–style script (explicit cells / explicit configuration), using ESA SNAP GPT for radar processing and GDAL for final reprojection + publication naming.

The workflow was developed to generate analysis-ready raster deliverables (raw backscatter, speckle-filtered backscatter, dB backscatter, polarization metrics, GLCM textures, and LIA) for study areas across the Iberian Peninsula, as part of the BLS–BIOMASS project.

The implementation is intentionally kept explicit and editable, not wrapped as a package.

## Why a notebook-style workflow?

• Processing parameters differ across sites and years
• SNAP operator behavior evolves (BandMaths, Collocate, Speckle-Filter, GLCM)
• Assumptions must remain visible (band naming, nodata, resampling)
• Reproducibility matters more than abstraction

This is a workflow, not a productized library.

## SCOPE & NON-GOALS
### IN SCOPE

• Input ingestion from tile folders or .zip tile archives
• SNAP GPT processing to produce:
– Gamma0 HH/HV (renamed from band_1)
– Local Incidence Angle (LIA) in degrees + radians
– Collocation (HH + HV + LIA)
– Sigma0 + Beta0 derived via BandMaths from collocated inputs
– Band merge of 6 backscatter layers (Beta/Gamma/Sigma × HH/HV)
– Speckle filtering (Lee) on merged stack
– Renaming to _Spk convention
– dB conversion from speckle-filtered backscatter
– Polarization metrics (ratio + normalized difference) for Gamma0/Sigma0/Beta0
– GLCM texture features from speckle-filtered stack
• GeoTIFF export (band-wise or grouped, depending on config)
• Final reprojection + nodata enforcement with GDAL
• Publication-ready file naming (João convention)
• Publish-mode aware cleanup (dev vs release)

### OUT OF SCOPE

• Machine learning / embeddings / PCA
• Feature scaling or normalization
• Model training or inference
• Statistical interpretation
• Scientific claims about biomass, structure, disturbance, or ecology

This workflow prepares data. Interpretation happens elsewhere.

### SOFTWARE DEPENDENCIES

Required
• ESA SNAP (Desktop + GPT)
• GDAL CLI tools (at least gdalwarp)
• Python ≥ 3.9

Python packages
• numpy
• rasterio
• lxml or xml.etree.ElementTree (used for .dim parsing)

Optional / situational
• geopandas (only if you add AOI clipping later)

## DATA ASSUMPTIONS

Input data
• Tile-based ALOS-2 products containing at least:
– *_sl_HH_F02DAR.tif
– *_sl_HV_F02DAR.tif
– *_linci_F02DAR.tif

Tile entry formats supported
• Folder containing the above TIFs
• .zip archive containing the above TIFs

Year and tile parsing
• The script derives:
– tile_code from N\d{2}[EW]\d{3} (e.g., N40W001)
– year_token from _YY_ (e.g., _21_) or _YYYY patterns
– tile_id = <tile_code>_<YYYY>

CRS targets (Iberian Peninsula)
• Spain / Aragon: EPSG:25830 (ETRS89 / UTM zone 30N)
• Portugal: EPSG:3763 (ETRS89 / Portugal TM06)

You must set TARGET_CRS appropriately per study area/run.

## OUTPUT STRUCTURE

The workflow produces three “layers” of outputs:

1) SNAP working products

Outputs/<tile_id>/*.dim and companion .data/ folders
These are the intermediate SNAP products for reproducibility and debugging.

2) Export staging products

Exports/<Group>/<tile_id>/*.tif
These are SNAP-exported GeoTIFFs (usually per-band to keep files manageable).

Groups include:
• Raw_Backscatter
• Speckle_Filtered
• dB_Backscatter
• Polarization_metrics
• GLCM
• LIA

3) Final published deliverables (naming + CRS enforced)

Exports/<tile_id>/<Group>/<StudyArea>_<YYYY0000>_<Platform>_<Variable>_<TileCode>.tif

Example:
ARAGON_20220000_ALOS2_BETAHH_N40W001.tif
stored under:
Exports/N40W001_2022/Raw_Backscatter/

WORKFLOW STRUCTURE (STAGE-BY-STAGE)
STAGE 0 — GLOBAL CONFIG

Defines:
• Study area name and platform token (João convention)
• Target CRS (EPSG:25830 for Spain; EPSG:3763 for Portugal)
• Input root, output root, export root
• Publish mode (dev/release)
• Caching behavior (.done stamps)

This is the only block that must be edited for a new study area.

## STAGE 1 — INPUT DISCOVERY & VALIDATION

For each tile entry (folder or .zip):
• Finds HH/HV/incidence files by suffix
• Validates raster readability and prints shapes
• If zipped: extracts required TIFs to a local _unzipped/ cache for SNAP compatibility

## STAGE 2 — SNAP GPT PROCESSING CHAIN (ORDERED)

Runs a fixed deterministic chain:

Rename Gamma0 HH/HV + LIA degrees

Convert LIA degrees → radians

Collocate HH/HV/LIA

Derive Sigma0 and Beta0 (BandMaths)

Merge 6 backscatter bands

Speckle filter merged stack

Rename _Spk bands

Convert to dB

Compute polarization metrics

Compute GLCM textures

All products are written into:
Outputs/<tile_id>/

## STAGE 3 — SNAP EXPORT TO GEOTIFF (STAGING)

Exports products into per-tile group folders:
Exports/<Group>/<tile_id>/

Per-band export is preferred for large products and for consistent downstream reprojection.

## STAGE 4 — FINAL GDAL PUBLISH (CRS + NAMING)

Uses gdalwarp to:
• Reproject every exported GeoTIFF into TARGET_CRS
• Enforce nodata = -9999 (srcnodata + dstnodata)
• Apply variable-appropriate resampling:
– near for GLCM and Polarization_metrics
– bilinear for continuous backscatter / LIA

Outputs are written into group-specific subfolders under the tile folder to prevent collisions:
Exports/<tile_id>/<Group>/...

## STAGE 5 — CLEANUP (PUBLISH MODE)

The workflow supports two modes:

dev
• Keep .dim products and stamps
• Keep staging exports
• Best for iteration, debugging, parameter changes

release
• Delete SNAP .dim and .data intermediates after export
• Optionally delete stamps (to avoid shipping cache metadata)
• Retains only final deliverables under Exports/<tile_id>/...

Note: in your current script, stamp deletion is scoped to the published tile subtree for safety.

## WHAT TO BE MINDFUL OF

• TARGET_CRS must match the study area.
– Spain: EPSG:25830
– Portugal: EPSG:3763

• Year parsing depends on filename conventions.
If tile naming changes, update parse_year_and_tile_code() first.

• Band naming is assumed by the BandMaths expressions.
Sigma0/Beta0 graphs reference specific band IDs created by earlier steps.
If SNAP changes band naming or collocation suffixes, those expressions may need adjustment.

• Speckle filter parameters are hard-coded.
If you change the filter type or size, it should be treated as a “new pipeline version”
(and cached outputs should be invalidated).

• Caching uses sidecar .done stamps.
They accelerate reruns but can hide changes if you manually alter files.
For a clean rerun, delete relevant outputs (or disable caching).

• Zipped tile inputs are extracted to _unzipped/.
That folder can grow large over time; safe to delete when you no longer need reruns.

• NoData handling:
Final gdalwarp sets nodata to -9999, but internal SNAP products may still contain NaNs.
The export graph for single-band includes a NaN→-9999 substitution.

• This workflow does not clip to AOIs by default.
It produces tile-wide rasters. Any masking/clipping should happen downstream.

## INTENDED USE

• Radar preprocessing for biomass/structure modeling (external)
• Feature generation for ML pipelines (external)
• Consistent multi-year radar covariates for Iberian-wide analyses
• Dataset preparation for integration with ALS/TLS/field plots
• Reproducible RS workflow templates for future platforms (S1, NISAR)

