# SNAP Preprocessing Workflows

This repository contains **reproducible, SNAP-based remote sensing preprocessing workflows** implemented as **Jupyter notebooks**.

The focus of this repository is **transparent, auditable preprocessing pipelines**, not packaged software.  
All workflows are intentionally published as notebooks to preserve **epistemic openness**: parameters, design decisions, and processing boundaries remain visible and modifiable by downstream users.

---

## Scope and Philosophy

This repository is designed around the following principles:

- **Workflow over package**  
  These are not Python packages or command-line tools. They are documented, executable workflows meant to be read, inspected, adapted, and extended.

- **SNAP as the processing backbone**  
  All preprocessing relies on ESA SNAP operators executed via `gpt`, with XML graphs generated programmatically where needed.

- **Reproducibility without abstraction loss**  
  The notebooks expose all relevant parameters (resampling, compositing logic, masking rules, texture settings, etc.) instead of hiding them behind APIs.

- **Sensor-specific separation**  
  Each sensor has its own folder and documentation. No attempt is made to force a unified abstraction across fundamentally different data products.

- **Publication-grade restraint**  
  These workflows stop at *preprocessing*. No modeling, attribution, or inference is embedded here.

---

## Intended Use

This repository is intended for:

- Researchers who need **transparent SNAP preprocessing** rather than black-box tools
- Projects where **parameter traceability** matters
- Workflows that may later be extended to new sensors (e.g. NISAR) without rewriting the conceptual pipeline
- Users comfortable with adapting notebooks to their own study areas, projections, and resolutions

It is **not** intended to be:
- A one-click processing tool
- A general remote sensing library
- A production data service

---

## Computational Environment

The workflows assume:

- ESA SNAP installed and accessible via `gpt`
- Python environment capable of running Jupyter notebooks
- Sufficient disk space for intermediate SNAP products
- Familiarity with SNAP operators and remote sensing preprocessing concepts

Exact environment requirements are documented **inside each sensor-specific folder**.

---

## Provenance and Use Context

These workflows were developed and validated while preprocessing satellite data for **study plots across the Iberian Peninsula**, as part of a larger biomass-mapping research effort.

They are published here as **generalizable preprocessing references**, not as project-specific artifacts.

---

## License

This repository is released under the **MIT License**.

You are free to:
- Use
- Modify
- Adapt
- Extend

with appropriate attribution.

---

## Notes on Extension

The structure of this repository is intentionally conservative.  
If SNAP support or community tooling evolves (e.g. future sensors or operators), these notebooks are designed to be **extended**, not replaced.

---




