### ALOS-2 PALSAR-2 SNAP-BASED PREPROCESSING WORKFLOW

This folder contains a fully reproducible, SNAP-based preprocessing workflow for ALOS-2 PALSAR-2 StripMap (F02DAR) data, implemented as a Jupyter-style executable script.

The workflow prepares analysis-ready radar backscatter and derived products for tile-based studies in Iberian peninsula, as part of the BLS–BIOMASS project.

The implementation is intentionally provided as an explicit, linear workflow rather than a packaged library.

WHY THIS IS A WORKFLOW (NOT A LIBRARY)

• Radar preprocessing choices are study-specific
• SNAP operator semantics matter and must remain visible
• Backscatter definitions (Gamma⁰, Sigma⁰, Beta⁰) are explicit, not implicit
• Assumptions must be auditable, not abstracted away
• Reproducibility matters more than code reuse

This script documents what is done and in what order, not just how to run it.

SCOPE & NON-GOALS
IN SCOPE

• ALOS-2 PALSAR-2 StripMap (F02DAR) ingestion
• Tile-wise processing (directory or zipped input)
• Gamma⁰ backscatter renaming from source TIFs
• Local Incidence Angle (LIA) handling (degrees → radians)
• Collocation of HH, HV, and LIA
• Computation of:

Sigma⁰ (terrain-corrected)

Beta⁰
• Multi-band backscatter merging
• Speckle filtering (Lee, fixed window)
• dB conversion with explicit offset
• Polarization metrics (ratio, ND)
• GLCM texture features
• Per-band GeoTIFF export
• CRS reprojection and dataset flattening
• Deterministic naming (João-conformant)
• Publish-mode aware cleanup (DEV vs RELEASE)

OUT OF SCOPE

• Radiometric terrain correction beyond LIA adjustment
• Absolute calibration validation
• Feature scaling or normalization
• PCA, embeddings, or feature learning
• Machine learning or modeling
• Ecological or structural interpretation
• Statistical inference

This workflow prepares inputs. Interpretation happens elsewhere.

SOFTWARE DEPENDENCIES
Required

• ESA SNAP (Desktop + GPT)
• GDAL (CLI tools: gdalwarp)
• Python ≥ 3.9

Python packages

• numpy
• rasterio
• zipfile
• xml.etree
• hashlib

Not required

• pandas
• scikit-learn
• xarray
• any ML frameworks

DATA ASSUMPTIONS

• Input data:

Folder with GeoTIFFs, or

Single .zip tile archive
• Required bands per tile:

HH backscatter

HV backscatter

Incidence angle raster
• Tile naming contains:

Tile code (e.g. N40W001)

Year (2- or 4-digit, parsed automatically)
• Target CRS:

Spain: EPSG:25830
• Processing is tile-isolated:

No cross-tile mosaicking

No temporal aggregation

WORKFLOW STRUCTURE (LOGICAL STAGES)
STAGE 0 — GLOBAL CONFIGURATION

Defines:

• Study area name
• Platform code
• CRS
• SNAP GPT path
• XML graph directory
• Output and export roots
• Publish mode (DEV / RELEASE)

This is the only section intended for routine edits.

STAGE 1 — INPUT VERIFICATION & INGESTION

• Accepts directory or .zip tiles
• Verifies presence of required HH, HV, and LIA files
• Uses /vsizip/ only for inspection
• Materializes TIFs to disk for SNAP compatibility

No processing occurs here — only validation.

STAGE 2 — CORE BACKSCATTER PROCESSING (SNAP)

Using SNAP GPT graphs:

Rename raw Gamma⁰ bands

Rename LIA (degrees)

Convert LIA degrees → radians

Collocate HH, HV, and LIA

Compute Sigma⁰ and Beta⁰

Merge Gamma⁰, Sigma⁰, Beta⁰ (HH/HV)

Apply speckle filtering

Rename speckle bands

Convert to dB

Compute polarization metrics

Compute GLCM textures

All intermediate products are written as SNAP .dim + .data.

STAGE 3 — PER-BAND EXPORT (SNAP)

• All groups exported band-by-band
• No large multiband GeoTIFFs produced prematurely
• NaNs explicitly replaced with NoData (-9999)
• Outputs written to:

Exports/<Group>/<TileID>/


This ensures transparency and avoids hidden band order assumptions.

STAGE 4 — DATASET FLATTENING & REPROJECTION (GDAL)

• Each per-band GeoTIFF is:

Reprojected to target CRS

Assigned variable-appropriate resampling
• Final deliverables written to:

Exports/<TileID>/<Group>/
  <STUDY>_<YYYY0000>_<PLATFORM>_<VARIABLE>_<TILE>.tif


Group-specific folders prevent variable collisions
(e.g. raw vs speckle-filtered backscatter).

STAGE 5 — PUBLISH-MODE AWARE CLEANUP

Modes:

• DEV

Keep all .dim products

Keep per-group exports

Keep cache stamps

• RELEASE

Keep only final GeoTIFF deliverables

Remove .dim and .data

Remove cache stamps

This allows full traceability during development and clean delivery for sharing.

WHAT TO BE MINDFUL OF

• This workflow assumes correct upstream calibration of ALOS-2 inputs
• Sigma⁰ and Beta⁰ formulations depend on LIA quality
• Speckle filtering parameters are fixed and not adaptive
• GLCM features are sensitive to NoData handling
• Tile naming must contain year information
• The script is designed for sequential tile processing, not parallel execution

INTENDED USE

• Biomass and structure modeling (downstream)
• Radar-optical feature fusion
• Fire and disturbance studies
• Dataset preparation for ML pipelines
• Teaching transparent SAR preprocessing

FINAL NOTE

This workflow is explicit by design.

If you need abstraction, build on top of it.
If you need models, apply them downstream.
If you need interpretation, that belongs in the paper.

This pipeline stops exactly where preprocessing should stop.
