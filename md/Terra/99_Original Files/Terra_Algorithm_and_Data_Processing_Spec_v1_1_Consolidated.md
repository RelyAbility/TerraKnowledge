# Terra -- Algorithm & Data Processing Specification v1.1 (Consolidated)

This consolidated specification merges the original v1.0 Algorithm &
Data Processing Spec with the v1.0 Addendum. It serves as the
authoritative engineering reference for Terra Phase 1 (satellite-led),
including calculations, compositing logic, parameters, pipeline
sequencing, confidence logic, QA, and acceptance criteria.

## 1. Scope (Phase 1)

• Monthly vegetation composites (Sentinel-2) and derived indices (NDVI,
optional EVI).

• Structural proxy metrics (NDVI heterogeneity; optional Sentinel-1
radar summaries).

• Disturbance detection (VIIRS fire tagging; NDVI change signals).

• Connectivity and fragmentation proxies (structural-only in Phase 1).

• Plot → Zone → Portfolio aggregation scaffolding.

• Tier 1 structural confidence scoring.

• Full method versioning and reproducibility requirements.

## 2. Core Index Calculations

### 2.1 NDVI

Formula: NDVI = (NIR − Red) / (NIR + Red)

Sentinel-2 Bands: B8 (NIR), B4 (Red)

• If denominator = 0 → NoData

• Clip to range \[-1,1\]

• Use Level-2A surface reflectance products

### 2.2 EVI (Optional)

Formula: EVI = 2.5 \* (NIR − Red) / (NIR + 6\*Red − 7.5\*Blue + 1)

Bands: B8 (NIR), B4 (Red), B2 (Blue)

• Blue band resampled to 10 m if required

### 2.3 Structural Heterogeneity

• Compute NDVI variance using rolling window (default 5×5 pixels)

• Store median + upper quartile per plot

## 3. Preprocessing & Cloud Masking

Preferred: Sentinel-2 SCL mask (cloud, shadow, cirrus removed).

Fallback: QA60 or cloud probability threshold.

Minimum valid pixel fraction per plot-month: 20% (default).

## 4. Monthly Composite Strategy

• Calendar month temporal window.

• Composite operator: median (default).

• Store NDVI_monthly, HET_NDVI_monthly, ValidPixelFraction.

## 5. Change & Disturbance Detection

ΔNDVI_mom = NDVI_t − NDVI\_(t−1)

ΔNDVI_yoy = NDVI_t − NDVI\_(t−12) (if available)

Default negative disturbance threshold: -0.10 NDVI (configurable).

VIIRS fire detections stored as contextual tags (not automatic
degradation labels).

## 6. Aggregation (Plot → Zone → Portfolio)

Plot summaries: median NDVI + IQR; heterogeneity median + upper
quartile.

Zone rollups: area-weighted medians; store density & depth.

Portfolio: aggregation container only (no credit logic in Phase 1).

## 7. Tier 1 Confidence Object

Confidence inputs:

• ValidPixelFraction

• Temporal depth (months available)

• Data consistency (variance of monthly summaries)

Outputs:

• confidence_score (0--1)

• confidence_band (Low/Medium/High)

All inputs stored for audit.

## 8. Default Parameters (Version-Controlled)

  -----------------------------------------------------------------------
  Parameter                           Default Value
  ----------------------------------- -----------------------------------
  Min valid pixels                    20%

  Composite operator                  Median

  Heterogeneity window                5×5 pixels

  ΔNDVI disturbance threshold         -0.10

  Fire buffer distance                1 km

  Update cadence                      Monthly
  -----------------------------------------------------------------------

## 9. Pipeline Sequence (Authoritative Order)

![](media/image1.png){width="6.8in" height="1.8695603674540682in"}

## 10. Acceptance Criteria (Engineering QA)

• NDVI formula verified against reference pixel values.

• Cloud-masked pixels excluded from composites.

• Monthly composites reproducible under identical inputs.

• Versioning preserves historical outputs when parameters change.

• Offline Monthly Brief cache functional.

• Push notification delivered on Monthly Brief release.
