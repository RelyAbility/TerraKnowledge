# Terra -- Product Requirements Specification (PRS) v1.2 Mobile

## 1. Product Vision

Terra is a mobile-first ecological intelligence platform delivering
monthly satellite-based structural insights and mission-driven
restoration guidance, designed to evolve into a portfolio-scale
biodiversity infrastructure platform aligned to HHI governance.

## 2. Platform Strategy

Phase 1 is developed using React Native (shared iOS/Android codebase).
Platform-specific modules may be used to meet offline-first
requirements, push notification reliability, and OS-level behaviour.

## 3. Strategic Architecture & Phase Evolution (Must-Have Constraints)

Terra must embed network effects and future-phase readiness into the
product architecture from Phase 1. This section defines non-negotiable
constraints to avoid architectural dead-ends.

Network effects to support:

• Geographic concentration (10 km flagship) → credibility → sponsors →
density.

• Density → confidence tightening → Elite pathway readiness.

• Missions → structural change → later functional validation.

• Plot → Zone → Portfolio aggregation enabling institutional reporting
and later market activation.

Required schema objects (Phase 1 onward):

• tier_level (Tier 1/2/3)

• confidence_score (numeric + band)

• sensor_density (per km²)

• time_series_depth (months)

• elite_eligible (boolean)

• elite_review_status (enum: not_started/in_review/approved/rejected)

• method_version (HHI/Terra method version used)

Phase constraints:

• Phase 1: Structural intelligence only; Elite/threshold engines
dormant; no credits.

• Phase 2: Acoustic integration activates Tier 2 and functional metrics.

• Phase 3: Portfolio scoring, thresholds, and external Elite review
workflows activate.

## 4. Core Functional Requirements (Phase 1)

• Plot boundary interface (draw/upload).

• Monthly satellite composite pipeline (NDVI/EVI) with clear monthly
refresh rhythm.

• Regional ecosystem identification per plot, with native vegetation
preference outputs.

• Mission recommendation engine linked to structural gaps (no functional
claims without acoustics).

• Offline-first: briefs and missions available offline; queued sync on
reconnect.

• Push notifications: Monthly Brief drop (core), optional mission
reminders, optional disturbance alerts.

• Tier transparency: Tier 1 vs Tier 2 clearly displayed for every
plot/zone.

## 5. Non-Functional Requirements

• Monthly processing cadence (plants change slowly; avoid over-refresh).

• Secure local storage and encryption where appropriate.

• Background sync architecture with conflict handling.

• Scalable cloud-native backend and geospatial indexing.

• Performance optimisation for maps and layer rendering.

• Auditability: method/version metadata stored for all derived outputs.

## 6. Future Requirements (Phase 2+)

• Acoustic sensor ingestion + functional guild metrics (Tier 2).

• Portfolio aggregation dashboards and reporting exports.

• Elite eligibility tracking and progress indicators.

• External validation workflow integration (review panel / auditor).

• Threshold gating and issuance logic (dormant until approved).
