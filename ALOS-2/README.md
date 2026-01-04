# ALOS-2 PALSAR-2 SNAP-BASED PREPROCESSING & PUBLISHING WORKFLOW

This folder contains a fully reproducible, SNAP-based preprocessing
and publishing workflow for ALOS-2 PALSAR-2 L-band SAR data.

The workflow is implemented as a Jupyter-executed Python script
that explicitly orchestrates ESA SNAP GPT graphs, GDAL reprojection,
and structured dataset publishing.

It was developed to prepare analysis-ready radar backscatter,
polarization metrics, incidence geometry, and texture features
for study areas across the Iberian Peninsula, as part of the
BLS–BIOMASS project.

The implementation is intentionally provided as an explicit script
executed in a notebook context, not as a packaged Python library.


Why this is not a package
------------------------

• Processing assumptions must remain visible  
• SNAP operator behavior must be explicit  
• Graph logic must be inspectable and editable  
• Parameter drift across sites is expected  
• Reproducibility matters more than abstraction  

This is a workflow, not a black-box tool.


-----------------------------------------------------------------------
SCOPE & NON-GOALS
-----------------------------------------------------------------------

IN SCOPE

• ALOS-2 PALSAR-2 Level 1.1 ingestion (HH, HV, incidence angle)
• SNAP GPT–based band renaming and math
• Local Incidence Angle (LIA) handling (degrees → radians)
• Radiometric conversions:
  – Gamma0 (input)
  – Sigma0 (terrain-corrected)
  – Beta0 (slope-normalized)
• Collocation of HH, HV, and LIA
• Multi-band merging
• Speckle filtering (Lee filter)
• dB conversion
• Polarization metrics (ratio, normalized difference)
• Texture feature extraction (GLCM)
• Per-band GeoTIFF export
• Dataset flattening and reprojection with GDAL
• Publish-mode–aware cleanup
• Deterministic naming following João’s conventions

OUT OF SCOPE

• Radiometric terrain correction (RTC beyond LIA usage)
• Machine learning or feature selection
• PCA or embeddings
• Feature normalization or scaling
• Ecological or statistical interpretation
• Any scientific claims

This workflow prepares data. Interpretation happens elsewhere.


-----------------------------------------------------------------------
SOFTWARE DEPENDENCIES
-----------------------------------------------------------------------

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
• subprocess
• glob
• shutil

Not required

• scikit-learn
• pandas
• xarray
• any ML frameworks


-----------------------------------------------------------------------
DATA ASSUMPTIONS
-----------------------------------------------------------------------

Input data

• ALOS-2 PALSAR-2 tiles
• Either:
  – extracted folders containing GeoTIFFs, or
  – zipped products (.zip), read via /vsizip/

Expected input bands

• HH backscatter
• HV backscatter
• Local incidence angle (linear incidence raster)

Coordinate reference systems

• Spain (e.g. Aragón): EPSG:25830 (ETRS89 / UTM zone 30N)
• Portugal:           EPSG:3763  (ETRS89 / PT-TM06)

Target CRS is configured per study area.

Processing is tile-based.
One tile corresponds to one year.


-----------------------------------------------------------------------
PIPELINE STRUCTURE (LOGICAL STAGES)
-----------------------------------------------------------------------

STAGE 0 — GLOBAL CONFIGURATION

Defines:

• Study area name
• Platform code (ALOS2)
• Target CRS
• SNAP GPT path
• XML graph directory
• Output and export roots
• Publish mode (dev / release)
• Cache and cleanup behavior

This is the only section that must be edited
to adapt the workflow to a new study area.


STAGE 1 — INPUT VERIFICATION

• Accepts either folders or zipped tiles
• Verifies presence of HH, HV, and incidence rasters
• Inspects shapes using rasterio
• Materializes zipped files for SNAP compatibility

No processing occurs here.


STAGE 2 — SNAP GPT PROCESSING CHAIN

Executed in a fixed, ordered sequence:

1. Band renaming:
   • Gamma0_HH
   • Gamma0_HV
   • LIA_degrees

2. LIA conversion:
   • degrees → radians

3. Collocation:
   • Gamma0_HH
   • Gamma0_HV
   • LIA_radians

4. Radiometric conversions:
   • Sigma0_HH / Sigma0_HV
   • Beta0_HH  / Beta0_HV

5. Merge of six backscatter bands

6. Speckle filtering (Lee filter)

7. Renaming of speckled bands

8. dB conversion

9. Polarization metrics:
   • ratio
   • normalized difference (ND)

10. Texture features (GLCM)

Each step is executed via an explicit SNAP XML graph.
All intermediate products are written as .dim + .data.


STAGE 3 — PER-BAND EXPORT (SNAP)

• Each .dim product is exported to GeoTIFF
• Per-band export is enforced for all groups
• NaN values are explicitly converted to -9999
• Outputs are organized as:

  Exports/<Group>/<TileID>/*.tif

Groups include:

• Raw_Backscatter
• Speckle_Filtered
• dB_Backscatter
• Polarization_metrics
• GLCM
• LIA


STAGE 4 — DATASET FLATTENING & REPROJECTION (GDAL)

• All exported single-band GeoTIFFs are:
  – reprojected to the target CRS
  – assigned a consistent NoData value
  – resampled with variable-appropriate methods

Resampling policy:

• GLCM, polarization metrics → nearest neighbour
• Continuous backscatter, LIA → bilinear

Final deliverables are written to:

  Exports/<TileID>/<Group>/

with filenames following:

  <StudyArea>_<YYYY0000>_<Platform>_<Variable>_<TileCode>.tif


STAGE 5 — OPTIONAL STACKING

• Multiband stacks can be created per group
• Disabled by default
• Intended only for inspection or legacy compatibility


STAGE 6 — CLEANUP (PUBLISH MODE AWARE)

Two publish modes are supported:

DEV
• Keep all SNAP .dim products
• Keep per-group exports
• Keep cache stamps

RELEASE
• Remove SNAP .dim and .data products
• Remove per-group intermediate exports
• Keep only final flattened datasets
• Remove cache stamps

Cleanup is deterministic and tile-scoped.


-----------------------------------------------------------------------
WHAT THIS WORKFLOW IS (AND IS NOT)
-----------------------------------------------------------------------

This workflow:

✓ Is transparent
✓ Is reproducible
✓ Is deterministic
✓ Respects SNAP operator semantics
✓ Produces analysis-ready radar datasets

This workflow:

✗ Is not a model
✗ Does not interpret SAR signals
✗ Does not optimize parameters
✗ Does not hide assumptions
✗ Does not produce scientific conclusions


-----------------------------------------------------------------------
INTENDED USE
-----------------------------------------------------------------------

• Radar data preprocessing
• Biomass and structure modeling (external)
• Fire and disturbance studies
• Feature generation for ML pipelines
• Teaching reproducible SAR workflows


-----------------------------------------------------------------------
FINAL NOTE
-----------------------------------------------------------------------

If you need a package, build one on top of this.
If you need a model, apply it downstream.
If you need interpretation, that belongs in the paper.

This workflow stops exactly where preprocessing should stop.


