# Terra -- Strategic Architecture & Evolution Framework v1.0

Purpose: Encode the network effects, phased evolution constraints, and
integrity guardrails required for Terra to mature from a Phase 1
satellite-led mobile product into a portfolio-scale biodiversity
infrastructure platform aligned to HHI governance.

## 1. Core Design Principle

Terra is a density-driven biodiversity intelligence network. Value
increases as geographic concentration, sensor density, portfolio
aggregation, and longitudinal time-series depth increase.

## 2. Embedded Network Effects (Flywheel)

### 2.1 Geographic Concentration Effect

• Flagship-first: concentrate within the 10 km zone before expanding.

• Concentration increases signal power, stakeholder trust, and sponsor
appeal.

Architectural requirements:

• All entities are geo-indexed and zone-aware.

• Zone-level aggregation is a first-class capability (not an
afterthought).

### 2.2 Density → Confidence Effect

• Higher sensor density tightens confidence bands and reduces
uncertainty.

• Time-series depth reduces volatility noise and supports stability
claims.

Architectural requirements:

• Confidence score is an explicit object, not a UI label.

• Sensor density is computed and stored per zone and per portfolio.

• Tier classification is dynamic, based on evidence present.

### 2.3 Mission → Structure → Function Effect

• Missions produce structural interventions (Phase 1).

• Structural change is measurable via satellite time-series.

• Functional response is validated via acoustics (Phase 2+).

Architectural requirements:

• Missions are persisted with geospatial footprint, date, and method
version.

• Structural metrics are stored as time-series and remain reproducible.

• Functional metrics plug into the same plot/zone/portfolio hierarchy.

### 2.4 Portfolio Aggregation Effect

• Individual plots roll up into zones; zones roll up into portfolios.

• Portfolio aggregation increases institutional relevance and market
readiness.

Architectural requirements:

• Data model supports Plot → Zone → Portfolio aggregation.

• Aggregations are versioned and reproducible.

## 3. Phase Evolution Constraints

### 3.1 Phase 1 -- Structural Intelligence (Satellite-Led)

• Tier 1 dominant: satellite composites and structural proxies.

• Missions: structural optimisation and mobilisation.

• Confidence: structural-only; no functional inference.

• Credits: out of scope; Elite evaluation dormant.

Constraints to build now (even if dormant):

• Elite eligibility fields exist but remain inactive.

• Acoustic ingestion interfaces scaffolded for Phase 2.

### 3.2 Phase 2 -- Functional Validation (Acoustic Integration)

• Tier 2: acoustic integration enables guild persistence and volatility
measures.

• Confidence begins incorporating density and time-series depth.

Constraints:

• Acoustic schema plugs into existing Plot objects.

• Tier updates automatically based on evidence present.

### 3.3 Phase 3 -- Market Activation (Portfolio & Verification)

• Portfolio-level scoring and threshold gating become active.

• Elite eligibility evaluation and external review workflow integrated.

Constraints:

• Threshold engine exists (dormant until Phase 3).

• All scores remain historically reproducible under method versioning.

## 4. Elite & Confidence Pathway (Must Exist in Schema Now)

• tier_level (Tier 1/2/3)

• confidence_score (numeric + band)

• sensor_density (per km²)

• time_series_depth (months)

• elite_eligible (boolean)

• elite_review_status (enum: not_started/in_review/approved/rejected)

• method_version (HHI/Terra method version used)

## 5. Versioning & Reproducibility

Every computed metric must store:

• Data source identifier and version

• Processing method version

• Timestamp / period of measure

• Confidence band inputs (density, depth, coverage)

## 6. Integrity Guardrails

• No functional inference from structural-only evidence.

• Tier 1 vs Tier 2 must be explicit in UI and data.

• No manual overrides that change score outputs without audit trail.

• Thresholds and scoring logic must be version-controlled.
