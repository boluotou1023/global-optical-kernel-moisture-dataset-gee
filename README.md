# Global Optical Moisture Dataset: Google Earth Engine Processing Code

## Overview

This repository contains the Google Earth Engine (GEE) JavaScript scripts used to generate the monthly global optical moisture dataset described in:

The kNDMI and kSVMI dataset: Global 1 km characterization of moisture dynamics using nonlinear kernel methods (2000–2024)  
[Author list]  
[Journal name, year, DOI if available]

The repository provides the processing workflow for producing the three monthly humidity indices:

- `kNDMI_005_fixed`
- `kNDMI_020_max`
- `kSVMI`

as well as the ancillary layer:

- `snow_fraction`

and the paired monthly quality-control layers:

- `valid_mask`
- `n_valid_obs`
- `qa_reason`

The scripts were developed in the **Google Earth Engine JavaScript API** and are intended to support transparency, reproducibility, and reuse of the dataset generation workflow.

---

## Repository structure

```text
.
├── README.md
├── LICENSE
├── CITATION.cff
├── code/
│   ├── 01_preprocess_mod09a1.js
│   ├── 02_compute_indices.js
│   ├── 03_monthly_composite.js
│   ├── 04_generate_qc_layers.js
│   ├── 05_export_products.js
│   └── 06_validation_analysis.js
├── docs/
│   ├── workflow_overview.md
│   ├── variable_definitions.md
│   ├── qa_definition.md
│   └── figure_reproduction.md
└── examples/
    ├── sample_roi.geojson
    └── example_output_names.txt
