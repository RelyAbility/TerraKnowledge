# Terra -- Algorithm & Data Processing Specification v1.0

Purpose: Specify the algorithms, calculations, compositing rules, and
processing workflow for Terra Phase 1 (satellite-led), with explicit
assumptions, edge cases, and versioning requirements. This document is
intended to prevent implementation drift and ensure reproducible
outputs.

## 1. Scope (Phase 1)

• Monthly vegetation composites (Sentinel-2) and derived indices (NDVI,
optional EVI).

• Structural proxy metrics (NDVI heterogeneity; optional Sentinel-1
radar backscatter summaries).

• Disturbance detection (fire, clearing/change) using VIIRS and
Landsat/Sentinel change signals.

• Connectivity and fragmentation proxies (structural-only in Phase 1).

• Plot-level aggregation, Zone-level aggregation, Portfolio-level
scaffolding.

• Confidence scoring inputs for structural-only evidence (Tier 1).

• Method and data-source versioning for reproducibility.

## Out of Scope (Phase 1)

• Acoustic functional metrics (Tier 2).

• HHI composite scoring weights and threshold engines (dormant until
Phase 3).

• Credit issuance logic.

• Automated Elite eligibility assessment (schema fields exist but
evaluation dormant).

## 2. Data Inputs & Band Definitions

2.1 Sentinel-2 (Level-2A Surface Reflectance recommended)

Required bands (10 m):

• Red: Band B4 (665 nm)

• NIR: Band B8 (842 nm)

Optional bands (20 m, resample to 10 m if used):

• Blue: Band B2 (490 nm) -- for EVI

• SWIR: Bands B11/B12 -- for moisture/burn proxies (optional)

Quality layers (recommended):

• Scene Classification Layer (SCL) OR QA60/cloud mask

• Cloud probability (if available)

2.2 Sentinel-1 (optional in Phase 1)

• GRD products; VV and/or VH backscatter (dB).

• Used for structural proxy robustness through cloud seasons.

2.3 Landsat 8/9 + Archive (optional in Phase 1, recommended for annual
change)

• Surface reflectance products used for long-term trend and change
detection.

2.4 VIIRS (disturbance)

• Active fire / thermal anomaly products used for fire event tagging and
disturbance context.

2.5 Queensland Regional Ecosystem (RE) GIS layers

• Used for plot ecosystem identification and native vegetation
preference mapping.

## 3. Pre-processing Rules

### 3.1 Spatial & Temporal Normalisation

• All raster outputs are aligned to a consistent grid at 10 m where
possible.

• If 20 m bands are used, resample to 10 m using bilinear interpolation
for continuous reflectance.

• All monthly products use a calendar month window (local time zone
reference for reporting).

### 3.2 Cloud/Shadow Masking (Sentinel-2)

Preferred approach: SCL-based masking (Level-2A).

Mask out pixels where SCL indicates:

• Cloud (high/medium probability)

• Cirrus

• Cloud shadow

• Snow/ice (rare in SEQ, but keep for generality)

Fallback approach (if SCL not available): QA60 mask and/or cloud
probability thresholding.

Minimum valid pixel rule per plot per month: configurable (default 20%
of plot pixels).

## 4. Core Index Calculations

### 4.1 NDVI

Definition: NDVI = (NIR − Red) / (NIR + Red)

Sentinel-2 implementation:

• NIR = B8, Red = B4 (surface reflectance)

Handling:

• If denominator (NIR + Red) = 0, set NDVI to NoData.

• Clip NDVI to \[-1, 1\].

### 4.2 EVI (Optional)

Definition: EVI = G \* (NIR − Red) / (NIR + C1\*Red − C2\*Blue + L)

Default constants (standard): G=2.5, C1=6, C2=7.5, L=1

Bands: NIR=B8, Red=B4, Blue=B2 (resample B2 to 10 m if needed).

### 4.3 Structural Heterogeneity Proxy (NDVI Variance)

Compute local heterogeneity as rolling-window variance of NDVI.

Default window: 5×5 pixels (50 m × 50 m) -- configurable.

Outputs:

• HET_NDVI raster (variance)

• Plot summary: median HET_NDVI and upper quartile HET_NDVI

### 4.4 Sentinel-1 Structural Proxy (Optional)

Compute monthly median backscatter (VV and/or VH) within plot.

Use: add robustness during cloudy months; interpret as canopy/structure
proxy.

Note: Radar proxies are supplementary and must not be marketed as direct
canopy height.

## 5. Monthly Compositing Strategy

Within each calendar month, compute a cloud-masked composite for each
pixel.

Default composite operator: median (robust to outliers).

Optional alternative: percentile (e.g., 60th) for greener bias -- only
if documented and versioned.

Output products per month:

• NDVI_monthly (raster)

• EVI_monthly (optional raster)

• HET_NDVI_monthly (variance raster)

• ValidPixelFraction_monthly (quality raster)

## 6. Disturbance & Change Detection

### 6.1 Fire Tagging (VIIRS)

• Tag plots/zones with fire events based on VIIRS detections within
buffer distance.

• Store event date, confidence, and proximity.

• Use fire tags as context for NDVI drops; do not automatically label as
degradation.

### 6.2 Vegetation Change Signals (Sentinel/Landsat)

Compute month-on-month and year-on-year NDVI deltas:

• ΔNDVI_mom = NDVI_t − NDVI\_(t−1)

• ΔNDVI_yoy = NDVI_t − NDVI\_(t−12) (when available)

Flag potential disturbance where ΔNDVI exceeds negative threshold
(configurable), subject to valid pixel coverage.

Annual change detection using Landsat archive is recommended for
historical context and clearing patterns.

## 7. Connectivity & Fragmentation Proxies (Phase 1)

Compute structural connectivity using classified vegetation masks
derived from NDVI thresholds or land cover products.

Suggested outputs (conceptual):

• Vegetation cover fraction within plot and buffers.

• Edge-to-area ratio of vegetated patches (within defined
neighbourhood).

• Nearest-neighbour distance between vegetated patches (within zone).

Note: These are structural proxies. Functional connectivity requires
Tier 2+ validation.

## 8. Aggregation Rules (Plot → Zone → Portfolio)

### 8.1 Plot-level summaries

For each plot and month compute:

• NDVI_median, NDVI_IQR (robust summary)

• HET_NDVI_median, HET_NDVI_upperQ

• ValidPixelFraction

If plot intersects multiple RE types, store RE composition percentages.

### 8.2 Zone-level summaries

Aggregate plot metrics using area-weighted medians or robust
aggregators.

Store zone density and time-series depth for confidence computations.

### 8.3 Portfolio scaffolding

Portfolio is an aggregation container (catchment/council).

Phase 1 stores portfolio membership but does not compute credit-related
metrics.

## 9. Confidence Scoring (Tier 1 -- Structural Only)

Confidence is stored as an explicit object, composed from measurable
inputs:

• Spatial coverage completeness (ValidPixelFraction)

• Temporal depth (months of data available)

• Data consistency (variance of monthly summaries)

Phase 1 confidence does not include acoustic density; that is added in
Phase 2.

Confidence output:

• confidence_score (0--1) and confidence_band (e.g., Low/Medium/High)

• confidence_inputs persisted for audit

## 10. Versioning, Auditability & Reproducibility

Every derived output must store:

• method_version (Terra algorithm version)

• data_source_id and data_source_version

• processing_timestamp and period covered (month)

• parameters used (cloud mask method, composite operator, thresholds)

Reprocessing: if method_version changes, historic products must be
reproducible under their original version.

## 11. Quality Assurance & Validation Checks

Minimum QA checks per month:

• ValidPixelFraction thresholds met before publishing plot brief.

• Cloud mask sanity: unexpected spikes in NDVI flagged for review.

• Edge cases: small plots, multi-RE plots, prolonged cloud periods.

User-facing: communicate data quality constraints clearly (e.g.,
\'insufficient clear-sky pixels this month\').

## 12. Output Products (Phase 1)

Per plot:

• Monthly Brief: NDVI trend, hotspot notes, mission suggestions,
RE-based planting preferences.

• Offline cache: store last brief + mission checklist for offline
access.

Per zone:

• Monthly zone composite summaries for reporting and sponsor narratives.
