**NDVI_Method_v1_0**

**Terra -- Phase 1 Satellite Vegetation Method**

**1. Purpose**

This document defines the NDVI processing methodology for Phase 1 of the
Terra ecological intelligence platform.

Phase 1 uses satellite-derived NDVI as the primary structural vegetation
condition signal.

This method supports:

- Vegetation health scoring

- Structural complexity proxy

- Biodiversity uplift trend modelling

No species-level inference is made in Phase 1.

**2. Satellite Data Source**

Primary Source:

- **Sentinel-2 (Level-2A Surface Reflectance)**

Spatial Resolution:

- 10 metres (Bands 4 and 8)

Temporal Resolution:

- 5-day revisit cycle

- Monthly composite used for reporting

Fallback (if required):

- Landsat 8/9 archive for historical baselining

**3. NDVI Formula**

NDVI is calculated as:

NDVI = (NIR − Red) / (NIR + Red)

Where:

- NIR = Band 8 (842 nm)

- Red = Band 4 (665 nm)

Value range:

- -1.0 to +1.0

Typical vegetation range:

- 0.2 to 0.9

**4. Monthly Composite Method**

To reduce cloud interference:

1.  Filter imagery by cloud mask (Sentinel Scene Classification Layer).

2.  Remove pixels flagged as:

    - Cloud

    - Cloud shadow

    - Snow (if applicable)

3.  Generate monthly composite using:

    - Median NDVI value per pixel across available scenes.

4.  If insufficient valid pixels:

    - Carry forward previous month with "low confidence" flag.

**5. Baseline Establishment**

Baseline NDVI for each plot is calculated using:

- 12-month rolling median (initial year of operation)

- Adjusted for seasonal variation

Baseline is ecosystem-relative, not global.

**6. Seasonal Normalisation**

Because vegetation cycles seasonally:

- Compare current month NDVI to same-month historical median where
  possible.

- Avoid comparing wet-season peaks to dry-season troughs.

- Use 3-month smoothing to reduce volatility.

**7. Plot-Level Aggregation**

For each plot:

1.  Clip NDVI raster to plot boundary.

2.  Exclude edge pixels below 50% overlap threshold.

3.  Compute:

    - Mean NDVI

    - NDVI variance (structural heterogeneity proxy)

Store:

- mean_ndvi

- ndvi_stddev

- valid_pixel_pct

- composite_confidence_flag

**8. Output Fields (Phase 1)**

For each plot/month:

- mean_ndvi

- ndvi_trend_delta

- seasonal_adjusted_delta

- structural_complexity_proxy

- composite_confidence_score

- method_version = \"NDVI_v1_0\"

**9. Known Limitations**

- NDVI measures greenness, not biodiversity.

- Cannot distinguish native vs invasive vegetation.

- Sensitive to prolonged cloud cover.

- Does not detect understorey complexity directly.

All biodiversity claims in Phase 1 are structural proxies only.

**10. Phase 1 Boundary**

This method supports:

- Tier 1 structural intelligence only.

It does not support:

- Species validation

- Habitat guild detection

- Functional biodiversity claims

Those are reserved for later phases.
