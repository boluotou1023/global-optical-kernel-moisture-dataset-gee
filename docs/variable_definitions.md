# Variable Definitions

## Overview

This document describes the variables included in the released monthly global optical moisture dataset and their associated ancillary and quality-control layers.

The dataset contains three main humidity indices, one ancillary snow-related layer, and three quality-control layers:

- `kNDMI_005_fixed`
- `kNDMI_020_max`
- `kSVMI`
- `snow_fraction`
- `valid_mask`
- `n_valid_obs`
- `qa_reason`

All layers are co-registered and provided as monthly GeoTIFF files with consistent spatial extent, projection, and grid alignment.

---

## Main humidity index layers

### `kNDMI_005_fixed`

A kernel-based normalized difference moisture index generated using a fixed kernel parameter setting of 0.05.  
This layer is designed to provide a moisture-sensitive optical indicator with enhanced contrast across dry-to-wet gradients while maintaining a relatively stable response range.

**Type:** continuous raster  
**Unit:** dimensionless  
**Temporal resolution:** monthly  
**Interpretation:** higher values generally indicate wetter surface conditions, whereas lower values indicate drier conditions.

---

### `kNDMI_020_max`

A kernel-based normalized difference moisture index generated using a maximum-constrained kernel parameter setting of 0.20.  
Compared with `kNDMI_005_fixed`, this layer generally exhibits a narrower effective response range in humid conditions and may show stronger saturation in some high-moisture environments.

**Type:** continuous raster  
**Unit:** dimensionless  
**Temporal resolution:** monthly  
**Interpretation:** higher values generally indicate wetter surface conditions, whereas lower values indicate drier conditions.

---

### `kSVMI`

A kernel-based surface vegetation moisture index that incorporates additional shortwave infrared information relative to the kNDMI-based formulations.  
This layer is intended to enhance sensitivity to soil–vegetation moisture dynamics and often shows stronger temporal responsiveness in validation analyses.

**Type:** continuous raster  
**Unit:** dimensionless  
**Temporal resolution:** monthly  
**Interpretation:** higher values generally indicate wetter surface conditions, whereas lower values indicate drier conditions.

---

## Ancillary layer

### `snow_fraction`

The fraction of days within a given month that are classified as snow-covered based on MOD10A1 `NDSI_Snow_Cover`.

This layer is provided to improve the transparency of the snow-screening procedure and to help users interpret seasonal data gaps, spatial coverage differences, and potential missing-value patterns in snow-affected regions.

**Type:** continuous raster  
**Unit:** fraction (0–1)  
**Temporal resolution:** monthly  
**Interpretation:**  
- `0`: no snow detected during the month  
- values close to `1`: snow detected during most or all days of the month

---

## Quality-control layers

### `valid_mask`

A binary indicator of pixel usability for the monthly index product.

A pixel is assigned:

- `1` if it is considered valid for the month
- `0` if it is considered invalid

In the current processing workflow, a pixel is treated as valid only when:

- it is not snow-affected during the month, and
- at least one quality-screened MOD09A1 observation contributes to the monthly mean

**Type:** binary raster  
**Unit:** none  
**Temporal resolution:** monthly  
**Recommended use:** use `valid_mask = 1` as the default screening criterion in most analyses.

---

### `n_valid_obs`

The number of valid MOD09A1 8-day composite observations contributing to the monthly mean at each pixel.

An observation is counted as valid only if it passes all screening procedures, including:

- reflectance range screening
- cloud and cloud-shadow filtering
- snow masking

This layer provides a measure of observational support for each monthly index value.

**Type:** integer raster  
**Unit:** count  
**Temporal resolution:** monthly  
**Recommended use:**  
- use as an indicator of monthly estimate robustness  
- optionally apply stricter filters such as `n_valid_obs >= 2` or `n_valid_obs >= 3`

---

### `qa_reason`

A diagnostic bitmask layer recording the reasons why a pixel is marked invalid, especially when `valid_mask = 0`.

Multiple invalidation factors may occur simultaneously. In such cases, the decimal value is obtained by summing the corresponding bit values.

**Type:** integer raster  
**Unit:** bitmask code  
**Temporal resolution:** monthly  
**Recommended use:** use for diagnosing the main causes of invalid or missing pixels.

The bit definitions are described in detail in `docs/qa_definition.md`.

---

## Notes for users

1. The three humidity indices are the primary scientific variables in the dataset.
2. The ancillary and QC layers are provided to support transparent filtering and interpretation.
3. For most applications, users should first apply `valid_mask = 1`.
4. For analyses sensitive to monthly mean stability, an additional constraint on `n_valid_obs` is recommended.
5. The `qa_reason` layer is primarily intended for diagnosis rather than direct scientific interpretation of moisture conditions.
