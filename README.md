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


## Code and data availability

The Google Earth Engine processing code used to generate the global monthly optical kernel moisture dataset is archived on Zenodo: https://doi.org/10.5281/zenodo.19687889.

The source repository is available at GitHub and corresponds to the archived software release associated with this DOI.

The dataset itself will be archived separately in Zenodo as dataset records with dataset-specific DOI(s), which will be added here after publication.

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
