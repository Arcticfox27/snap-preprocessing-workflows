Sentinel-1 SLC — SNAP-based Coherence & Interferometric Preprocessing

This folder contains a fully reproducible SNAP-based preprocessing workflow for
Sentinel-1 IW SLC (Single Look Complex) data.

The goal is to transform raw SLC acquisitions into geometrically aligned, physically interpretable interferometric products, including coherence, unwrapped phase (optional), and texture metrics, without embedding any modeling or inference assumptions.

This workflow was developed and validated on Iberian Peninsula study areas as part of the BLS-BIOMASS project.

The implementation is a plain Python script using SNAP GPT, not a package.
All processing stages are explicit, inspectable, and modular.

Philosophy

This repository performs preprocessing only.

It:

Reads Sentinel-1 SLC products

Applies SNAP interferometric operators

Produces coherence, phase, and texture products

Writes analysis-ready outputs

It does not:

Perform deformation analysis

Interpret phase

Apply statistical or ML models

Infer biomass, fire severity, or structure

Those steps belong downstream.

Inputs

Sentinel-1 IW SLC (*.SAFE or *.zip)

Dual-polarization acquisitions (VV/VH or HH/HV)

Two or more temporally separated acquisitions over the same area

Coordinate systems

The workflow supports two projections:

Region	CRS
Portugal	EPSG:3763
Spain	EPSG:25830

Set via CFG.country.

Processing pipeline

Processing is organized per subswath (IW1 / IW2 / IW3), with automatic or manual selection.

The pipeline is strictly sequential:

1) Core interferometric chain

For each SLC pair and subswath:

TOPSAR Split (subswath + burst selection)

Apply precise orbit files

Back-geocoding (master–slave alignment)

Enhanced Spectral Diversity (ESD)

Interferogram formation + coherence estimation

TOPSAR Deburst

This produces a debursted interferogram with:

Wrapped phase

Coherence (per polarization)

2) Multilook & phase filtering

Multilooking (range/azimuth looks)

Goldstein phase filtering

This stage produces a noise-reduced interferometric product suitable for:

Coherence analysis

Optional phase unwrapping

3) Optional phase unwrapping (SNAPHU)

Phase unwrapping is explicitly optional.

If enabled (do_unwrap = True):

SNAP SnaphuExport

External SNAPHU execution

SNAP SnaphuImport

If disabled (do_unwrap = False), the workflow stops cleanly after SnaphuExport, allowing use cases that require only wrapped interferometry or coherence.

4) Terrain correction

If unwrapping is enabled:

Terrain correction to 25 m resolution

Resulting products are geometrically consistent and map-projected.

5) Texture metrics (GLCM)

GLCM textures are computed only from coherence bands

Ten texture metrics are produced per polarization

Textures are exported as individual GeoTIFFs.

Subswath handling

Subswath selection is controlled by cfg.subswath:

"AUTO" → automatically selects the best common subswath

"IW1", "IW2", "IW3" → force a specific subswath

"ALL" → process all subswaths independently

Subswaths are never merged implicitly.
Each subswath produces its own outputs.

Output structure

Outputs are written to:

Exports/SingleBands_<IWx>/

Each variable is exported as an individual GeoTIFF:

<Study>_<Year>_S1SLC_<Variable>_<IWx>_<Tile>.tif

Examples:

Monsanto_20200000_S1SLC_COHVV_IW2_A.tif

Monsanto_20200000_S1SLC_COHVVCONTRAST_IW2_A.tif

Monsanto_20200000_S1SLC_COHVHENTROPY_IW2_A.tif

These files are directly usable in GIS, ML pipelines, or multi-sensor workflows.

What this workflow guarantees

✔ Correct SNAP operator ordering
✔ Explicit interferometric assumptions
✔ Subswath-consistent processing
✔ Optional, transparent phase unwrapping
✔ No implicit merging or resampling
✔ Deterministic, script-driven execution

What this workflow does not do

✗ No deformation modeling
✗ No time-series analysis
✗ No feature selection
✗ No normalization or scaling
✗ No machine learning
✗ No ecological interpretation

Intended use

This pipeline is designed to feed:

Coherence-based structure metrics

Biomass and disturbance modeling

Fire and drought studies

Multi-sensor fusion (Sentinel-2, GEDI, ICESat-2, ALS/TLS)

Downstream statistical or ML workflows

without embedding interpretive or modeling decisions in the preprocessing stage.

Final note

This workflow defines a clear stopping point:
it produces interferometric variables and textures, and then stops.

Any interpretation, modeling, or inference should begin after this point, using these outputs as inputs.