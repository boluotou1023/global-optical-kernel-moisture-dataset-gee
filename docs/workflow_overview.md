# Workflow Overview

## Purpose

This document summarizes the workflow used to generate the monthly global optical moisture dataset from Google Earth Engine (GEE) source data and to perform the main validation analyses.

## Overview of the processing chain

The dataset production workflow consists of six main stages:

1. Source data preparation
2. Quality screening and snow masking
3. Optical mositure index calculation
4. Monthly compositing and ancillary layer generation
5. Quality-control layer generation
6. Validation and evaluation

---

## 1. Source data preparation

The primary optical input data are derived from MODIS products, mainly:

- **MOD09A1**: surface reflectance
- **MOD10A1**: snow cover information
- **MOD13Q1**: NDVI
Additional datasets are used for validation and analysis, including:

- **ESA CCI SM**
- **GLDAS**
- **CHIRPS**
- **ISMN**
- **SMAPEx-5**

These datasets are accessed and processed within the Google Earth Engine environment where applicable, while some validation data are handled outside GEE after download and reorganization.

---

## 2. Quality screening and snow masking

Before index calculation, MOD09A1 reflectance data are screened using the following procedures:

- reflectance range screening
- cloud and cloud-shadow filtering based on MOD09A1 quality information
- snow masking using MOD10A1 `NDSI_Snow_Cover`

Pixels identified as snow-affected within a month are excluded from the monthly compositing process to reduce contamination of optical moisture signals by snow cover.

---

## 3. Optical moisture index calculation

After screening, the optical moisture indices are calculated from the valid MOD09A1 reflectance bands.

The main dataset products include:

- `kNDMI_005_fixed`
- `kNDMI_020_max`
- `kSVMI`

These indices are generated at the monthly scale and exported as GeoTIFF files.

---

## 4. Monthly compositing and ancillary layer generation

For each month, valid observations are composited to generate monthly index mosaics.

In addition to the three main indices, the workflow also produces:

- `snow_fraction`

The `snow_fraction` layer records the fraction of days within a month that are classified as snow-covered, providing additional transparency for interpreting coverage and seasonal data gaps.

---

## 5. Quality-control layer generation

A monthly quality-control product is generated together with each monthly index mosaic.

The QC product includes:

- `valid_mask`
- `n_valid_obs`
- `qa_reason`

These layers indicate whether a pixel is usable, how many valid MOD09A1 observations contributed to the monthly mean, and the dominant reasons for invalidation.

For most applications, `valid_mask = 1` is recommended as the default screening criterion.

---

## 6. Validation and evaluation

The generated dataset is evaluated using multiple validation strategies:

### 6.1 Spatial consistency with benchmark soil moisture products
- comparison with ESA CCI SM
- comparison with GLDAS

### 6.2 Temporal consistency with hydroclimatic forcing
- long-term trend analysis
- correlation with CHIRPS precipitation under different Köppen climate zones

### 6.3 Validation with in situ and airborne observations
- validation against selected ISMN sites
- comparison with SMAPEx-5 data

### 6.4 Drought-response evaluation
- comparison between humidity indices and NDVI in relation to drought-related variability

---

## Output files

The final released dataset includes:

### Main layers
- `kNDMI_005_fixed`
- `kNDMI_020_max`
- `kSVMI`

### Ancillary layer
- `snow_fraction`

### Quality-control layers
- `valid_mask`
- `n_valid_obs`
- `qa_reason`

All output products are organized as monthly GeoTIFF files with consistent naming conventions and grid alignment.

---

## Reproducibility notes

This repository provides the archived GEE processing scripts corresponding to the released dataset version.  
To reproduce the workflow, users may need to update:

- region of interest definitions
- asset paths
- export folder names
- time range settings
- output naming rules

Because the workflow depends on the Google Earth Engine environment and on the continued availability of input datasets, minor adjustments may be required when reproducing the code in a different account or project setting.
