# Terra -- Algorithm & Data Processing Spec v1.0 Addendum

This addendum provides (1) explicit default parameters, (2) a
step-by-step pipeline sequence, and (3) test cases and acceptance
criteria for Phase 1 modules. It complements the main Algorithm & Data
Processing Specification.

## 1. Default Parameters (Proposed Baseline)

All parameters are version-controlled. Defaults below are starting
points and may be tuned with documented rationale.

  ---------------------------------------------------------------------------
  Module            Parameter         Default               Notes
  ----------------- ----------------- --------------------- -----------------
  Sentinel-2        Product level     L2A surface           Use SCL when
                                      reflectance           available

  Cloud masking     Mask classes      SCL                   Fallback:
                                      cloud+shadow+cirrus   QA60/cloud-prob

  Plot validity     Min valid pixels  20%                   Below this:
                                                            'insufficient
                                                            clear-sky'

  NDVI              Bands             B8 (NIR), B4 (Red)    Clip to \[-1,1\];
                                                            denom=0 → NoData

  EVI (opt)         Constants         G=2.5, C1=6, C2=7.5,  Bands: B8,B4,B2
                                      L=1                   

  Compositing       Operator          Median                Alternative
                                                            percentile must
                                                            be versioned

  Compositing       Temporal window   Calendar month        Timezone set for
                                                            reporting

  Heterogeneity     Window size       5×5 pixels            Configurable;
                                                            default \~50m

  Change detection  ΔMoM threshold    -0.10 NDVI            Flag only if
                                                            valid pixels ok

  Change detection  ΔYoY threshold    -0.10 NDVI            Requires t-12
                                                            available

  Fire tagging      Buffer distance   1 km                  Store proximity +
                                                            confidence

  Aggregation       Plot summary      Median + IQR          Robust to
                                                            outliers

  Aggregation       Zone rollup       Area-weighted medians Store
                                                            density/depth

  Confidence (Tier  Inputs            valid pixels, depth,  0--1 score + band
  1)                                  consistency           

  Publishing        Update cadence    Monthly               Push 'Monthly
                                                            Brief'
  ---------------------------------------------------------------------------

## 2. Pipeline Sequence (Phase 1)

The following sequence is the authoritative order of operations for
Phase 1 processing.

![](media/image1.png){width="7.2in" height="1.9795352143482066in"}

1\) Ingest sources for the target month (Sentinel-2 required; others
optional).

2\) Spatial normalisation: align grids, resample as needed, clip to
AOI/plots.

3\) Apply cloud/shadow masking and compute ValidPixelFraction.

4\) Compute per-scene indices (NDVI; EVI optional) and per-scene
structural proxies.

5\) Monthly composite: median per pixel; generate quality layers.

6\) Compute change signals (ΔMoM, ΔYoY where available).

7\) Disturbance context: apply VIIRS fire tags and store events.

8\) Aggregate to plot summaries; roll up to zone and portfolio
scaffolding.

9\) Compute Tier 1 confidence object from coverage/depth/consistency.

10\) Persist outputs with method_version + parameters; publish Monthly
Brief + push notifications.

## 3. Test Cases & Acceptance Criteria

Acceptance criteria are written to be verifiable by Emergent during
implementation.

  ----------------------------------------------------------------------------
  Module                       Test Case               Acceptance Criteria
  ---------------------------- ----------------------- -----------------------
  NDVI calculation             Given a sample pixel    Pass if unit tests
                               with known Red and NIR  cover normal, denom=0,
                               reflectance values,     and clipping bounds.
                               NDVI matches formula    
                               output within ±1e-6.    
                               Denominator=0 returns   
                               NoData.                 

  Cloud masking                Pixels marked           Pass if masked pixels
                               cloud/shadow in SCL are do not contribute to
                               excluded from           NDVI_monthly and
                               composites;             coverage metric is
                               ValidPixelFraction      correct.
                               reflects remaining      
                               pixels.                 

  Monthly compositing          For a pixel with N      Pass if reproducible
                               clear observations,     across runs with same
                               composite equals median inputs and parameters.
                               of observations (or     
                               documented operator).   

  Plot validity gate           If ValidPixelFraction   Pass if UI/API flags
                               \< 20% for a            quality state and
                               plot-month, Monthly     suppresses unreliable
                               Brief is generated with insights.
                               'insufficient           
                               clear-sky' status and   
                               no misleading trend     
                               arrow.                  

  Heterogeneity metric         HET_NDVI output equals  Pass if window-size
                               variance of NDVI within parameter is honoured
                               the defined window;     and validated by test
                               changing window size    raster.
                               changes result          
                               predictably.            

  Change detection             ΔMoM and ΔYoY are       Pass if flags never
                               computed correctly;     trigger when
                               disturbance flags only  ValidPixelFraction
                               trigger when validity   below threshold.
                               gate passed.            

  Fire tagging                 VIIRS detections within Pass if events are
                               1 km buffer create a    stored and displayed as
                               fire event record with  context, not
                               date and confidence;    automatically as
                               plot-month marked as    degradation.
                               'fire context'.         

  Aggregation rollups          Plot summaries use      Pass if aggregation
                               median + IQR; zone      outputs match reference
                               rollup uses             calculations for test
                               area-weighted medians;  set.
                               portfolio scaffolding   
                               stores membership       
                               without credit logic.   

  Confidence object            Confidence_score is     Pass if confidence is
                               produced for every      monotonic with
                               plot-month with stored  synthetic test inputs
                               inputs; increases with  and stored for audit.
                               greater valid pixels    
                               and time depth.         

  Versioning/reproducibility   Re-running the pipeline Pass if outputs are
                               with identical inputs   traceable and
                               and method_version      reproducible under both
                               produces identical      versions.
                               outputs; changing a     
                               parameter increments    
                               method_version and      
                               preserves prior         
                               results.                

  Offline cache &              Latest Monthly Brief is Pass if QA checklist
  notifications                cached locally; user    confirms offline open +
                               receives monthly push   queued sync + push
                               notification; offline   delivery in staging.
                               viewing works without   
                               connectivity.           
  ----------------------------------------------------------------------------

## 4. Notes on Canopy Height

Phase 1 does not claim canopy height from Sentinel-2/Sentinel-1
directly. Any canopy height product requires separate data sources
(e.g., LiDAR/photogrammetry) and will be specified in a future Tier 3
document. In Phase 1, the app should label outputs as 'structural
proxies'.
