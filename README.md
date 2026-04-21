# # Global Optical Kernel Moisture Dataset: Google Earth Engine Processing Code

## Overview

This repository contains the Google Earth Engine (GEE) JavaScript code used to generate the monthly global optical kernel moisture dataset for 2000–2024.

The workflow produces three monthly optical moisture indices:

- `kNDMI_005_fixed`
- `kNDMI_020_max`
- `kSVMI`

It also generates the ancillary layer:

- `snow_fraction`

and the paired monthly quality-control layers:

- `valid_mask`
- `qa_reason`
- `n_valid_obs`

The code is provided to support transparency, reproducibility, and reuse of the dataset-generation workflow. The associated data paper is currently being prepared for submission to *Earth System Science Data* (ESSD).

---

## Repository structure

```text
.
├── README.md
├── LICENSE
├── CITATION.cff
├── code/
│   └── export_kndmi_ksvmi_monthly_v1.js
└── docs/
    ├── workflow_overview.md
    ├── variable_definitions.md
    └── qa_definition.md
