# QA Definition

## Overview

This document describes the quality-control (QC) layers provided together with each monthly global optical moisture product.

The monthly QC product contains three layers:

- `valid_mask`
- `n_valid_obs`
- `qa_reason`

These layers are intended to help users determine whether a pixel is usable, evaluate the observational support behind each monthly value, and diagnose the main causes of invalidation or missing data.

---

## 1. `valid_mask`

`valid_mask` is a binary usability flag for the monthly index layers.

### Definition

A pixel is assigned:

- `valid_mask = 1` if the pixel is considered valid for the month
- `valid_mask = 0` if the pixel is considered invalid for the month

### Validity criteria

A pixel is treated as valid only when:

1. it is not identified as snow-affected during the month, and  
2. at least one quality-screened MOD09A1 observation contributes to the monthly mean (`n_valid_obs >= 1`)

### Recommended use

For most applications, `valid_mask = 1` is recommended as the default screening criterion before further analysis.

---

## 2. `n_valid_obs`

`n_valid_obs` records the number of valid MOD09A1 8-day composite observations contributing to the monthly mean at each pixel.

### Definition

An observation is counted as valid only if it passes all relevant screening procedures, including:

- reflectance range screening
- cloud filtering
- cloud-shadow filtering
- snow masking

### Interpretation

Higher values indicate stronger observational support for the monthly mean.  
Lower values indicate that the monthly value is based on fewer valid observations and may therefore be less robust.

### Recommended use

Users may optionally impose stricter filtering criteria such as:

- `n_valid_obs >= 2`
- `n_valid_obs >= 3`

This is especially useful in analyses that are sensitive to the stability of monthly averages.

---

## 3. `qa_reason`

`qa_reason` is a 16-bit diagnostic bitmask describing the reasons why a pixel is marked invalid, particularly when `valid_mask = 0`.

### General rule

Multiple invalidation factors may co-occur in the same month.  
When this happens, the corresponding decimal values are summed to form the final `qa_reason` code.

For example:

- `qa_reason = 8` means snow-related invalidation only
- `qa_reason = 10` means snow (`8`) + cloud (`2`)
- `qa_reason = 9` means snow (`8`) + no valid observations under non-snow conditions (`1`)

### Bit definitions

| Bit | Decimal value | Flag name       | Meaning |
|-----|---------------|-----------------|---------|
| 0   | 1             | `noObs`         | No valid observations are available in the month under non-snow conditions |
| 1   | 2             | `cloud`         | Evidence of cloud contamination within the month |
| 2   | 4             | `shadow`        | Evidence of cloud-shadow contamination within the month |
| 3   | 8             | `snow`          | Pixel is affected by snow during the month |
| 4   | 16            | `out_of_range`  | Reflectance values fall outside the accepted range |

### Priority treatment of snow

In the current implementation, snow is treated as a priority invalidation factor.  
Pixels identified as snow-affected within a month are explicitly flagged as snow-related invalidation.

### Recommended use

`qa_reason` is intended primarily for:

- diagnosing the causes of missing or invalid pixels
- distinguishing snow-related gaps from cloud-related gaps
- understanding whether invalidation is caused by insufficient valid observations or reflectance anomalies

It is not intended to replace `valid_mask` as the main data-screening variable.

---

## Practical recommendations

For most analyses, we recommend the following workflow:

1. Apply `valid_mask = 1`
2. If needed, impose an additional threshold on `n_valid_obs`
3. Use `qa_reason` only when diagnosing missing data patterns or evaluating invalidation mechanisms

This combination provides a transparent and practical framework for filtering and interpreting the monthly optical moisture dataset.
