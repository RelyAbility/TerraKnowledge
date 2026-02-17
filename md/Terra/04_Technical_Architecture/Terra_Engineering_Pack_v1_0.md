# Terra -- Engineering Pack v1.0

This Engineering Pack consolidates all core specifications required for
Terra Phase 1 (Satellite-Led, Mobile-First) implementation. It is
intended to guide Emergent's build sequencing, reduce ambiguity, and
minimise future architectural rework.

## 1. Included Specifications

• 1. Strategic Architecture & Evolution Framework v1.0

• 2. Product Requirements Specification (PRS) v1.2 Mobile

• 3. Functional Specification -- Phase 1 Mobile

• 4. Algorithm & Data Processing Specification v1.1 (Consolidated)

• 5. API Contract Specification v1.0

• 6. Database Schema Draft v1.0

• 7. Security & Governance Specification v1.0

• 8. Performance Envelope & Cost Model v1.0

• 9. Conceptual Data Model Diagram v1.0

## 2. Recommended Build Order

• Step 1 -- Database schema implementation (Postgres + PostGIS)

• Step 2 -- Method versioning framework and core tables

• Step 3 -- Satellite processing pipeline (NDVI, compositing, QA)

• Step 4 -- Confidence object calculation (Tier 1 structural)

• Step 5 -- API layer (plots, briefs, missions, auth)

• Step 6 -- Offline sync engine and event queue

• Step 7 -- Mobile app implementation (React Native)

• Step 8 -- Push notification integration

• Step 9 -- QA & reproducibility validation

• Step 10 -- Security review and deployment hardening

## 3. Phase 1 Acceptance Checklist

✔ NDVI calculation verified against reference inputs

✔ Cloud masking implemented and validated

✔ Monthly composites reproducible under identical inputs

✔ Plot → Zone → Portfolio aggregation operational

✔ Confidence score computed and stored per brief

✔ Tier label enforced server-side

✔ Offline brief viewing functional

✔ Mission updates sync correctly after reconnect

✔ Push notifications delivered for Monthly Brief

✔ Versioning preserves historic outputs

## 4. Architectural Guardrails (Non-Negotiable)

• No functional inference from structural-only data (Phase 1).

• Elite eligibility fields exist but remain dormant.

• All derived outputs store method_version and data source metadata.

• No manual editing of derived metrics without version increment.

• API and DB changes must not break version reproducibility.

## 5. Transition to Phase 2 Readiness

The system must remain compatible with acoustic ingestion, density
scoring, and portfolio-level HHI activation. Phase 1 completion does not
imply credit issuance capability.
