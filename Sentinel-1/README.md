# Sentinel-1 GRD — SNAP-based Preprocessing & Dataset Builder

This folder contains a fully reproducible SNAP-based preprocessing workflow for  
**Sentinel-1 IW GRD (Ground Range Detected) data**.

The goal is to transform raw GRD scenes into spatially consistent, analysis-ready
radar variables (Gamma0, Sigma0, Beta0, LIA, DEM, and GLCM textures), without
embedding any modeling, interpretation, or ML assumptions.

This workflow was developed and validated on Iberian Peninsula study areas
as part of the **BLS-BIOMASS project**.

The implementation is a **Jupyter Notebook**, not a Python package.  
All processing steps are explicit and editable.

---

## Philosophy

This repository does **preprocessing only**.

It:
- Reads GRD products
- Applies SNAP operators
- Produces physically meaningful radar variables
- Writes GeoTIFFs

It does **not**:
- Perform feature scaling
- Do PCA or embeddings
- Train models
- Infer biomass, fire, or structure

Those belong downstream.

---

## Inputs

- Sentinel-1 IW GRD (`*.SAFE` or `*.zip`)
- VH and VV polarizations
- Multiple acquisition dates over the same area

---

## Coordinate systems

The notebook supports two projections:

| Region | CRS |
|------|-----|
| Portugal | EPSG:3763 |
| Spain | EPSG:25830 |

Set via `CFG.country`.

---

## Processing pipeline

The workflow is split into two physically distinct branches:

---

### 1) Gamma0 & Sigma0 branch

For each GRD scene:

1. Apply orbit file  
2. Thermal noise removal  
3. Radiometric calibration → Gamma0 + Sigma0  
4. Terrain flattening  
5. Speckle filtering  
6. Terrain correction to 25 m  

Each scene becomes a `.dim` product with bands:

Gamma0_VH
Sigma0_VH
Gamma0_VV
Sigma0_VV
elevation
localIncidenceAngle


These are **validated by preflight checks** before further processing.

Then:

7. All scenes are collocated
8. Temporal averages are computed using SNAP BandMaths
9. One composite product is created
10. GLCM textures are computed for all backscatter bands
11. Originals + textures are merged into a BigTIFF

---

### 2) Beta0 branch

For each GRD scene:

1. Apply orbit file  
2. Thermal noise removal  
3. Calibration → Beta0  
4. Speckle filtering  
5. Terrain correction to 25 m  

Each scene produces:

Beta0_VH
Beta0_VV
elevation
localIncidenceAngle


Then:

6. Scenes are collocated
7. Temporal averages are computed
8. A composite Beta0 product is created
9. GLCM textures are computed
10. Originals + textures are merged into a BigTIFF

---

## Preflight safety checks

The notebook enforces a **band-name contract** before any compositing occurs.

For Gamma/Sigma TC products, it requires:

Gamma0_VH
Sigma0_VH
Gamma0_VV
Sigma0_VV
elevation
localIncidenceAngle

For Beta TC products, it requires:

Beta0_VH
Beta0_VV
elevation
localIncidenceAngle


If SNAP changes naming or a scene is malformed, the workflow aborts with a clear
error instead of silently producing corrupted composites.

This guarantees reproducibility.

---

## Output structure

After processing, outputs are written to:
Exports/SingleBands/
Each variable is exported as an individual GeoTIFF: 
"<Study><Year>S1GRD<Variable><Tile>.tif"

Examples:
Monsanto_20200000_S1GRD_GAMMA0VH_A.tif
Monsanto_20200000_S1GRD_GLCMGAMMAVHCONTRAST_A.tif
Monsanto_20200000_S1GRD_LIA_A.tif


These files are directly usable in ML, GIS, or biomass pipelines.

---

## What this workflow guarantees

✔ Radiometric correctness  
✔ Geometric consistency  
✔ Temporal consistency  
✔ SNAP-operator transparency  
✔ No hidden resampling  
✔ No implicit modeling  

---

## What this workflow does not do

✗ No classification  
✗ No feature selection  
✗ No normalization  
✗ No PCA or embeddings  
✗ No ecological interpretation  

---

## Intended use

This pipeline is designed to feed:

- Biomass modeling
- Structure modeling
- Fire and drought studies
- Multi-sensor fusion (e.g. with Sentinel-2, GEDI, ICESat-2)
- Machine-learning pipelines

without contaminating the data with modeling decisions.

---

## Final note

If you need a package, build one on top of this.  
If you need a model, apply it downstream.  
If you need interpretation, that belongs in the paper.

This repository stops exactly where responsible radar preprocessing should stop.















