# Terra Ecological Operating System (EOS v1)

## Technical Implementation Brief

Date: 16 February 2026

Project: Brookfield -- 4.2 km Ecological Boundary (Symbolic Birthplace)

## 1. Purpose

This document defines the technical architecture, stack requirements,
performance standards, governance controls, and non‑negotiable
principles for the Terra Ecological Operating System (EOS v1). It
accompanies the 8 Build Packs and serves as the execution framework for
Emergent.

## 2. Core Architectural Principles (Non‑Negotiable)

- 1 km × 1 km grid-first (landscape-first UX).

- 4.2 km ecological boundary centred on 81 Boscombe Road, Brookfield.

- Edge-only adjacency (no corner-touch logic).

- Public grid map; private property boundaries.

- BPPF_v1.0 scoring logic frozen and versioned.

- All scoring runs logged with dataset versions.

- No manual score overrides without event log entry.

- Append-only history for score runs, missions, and events.

## 3. Technology Stack

- Frontend: React Native (grid-first UX).

- Backend: Supabase (PostgreSQL + PostGIS).

- API Layer: REST endpoints / Supabase functions.

- Spatial Reference System: GDA2020 (EPSG:7844).

- Mapping: MapLibre or equivalent high-performance RN mapping library.

- Authentication: Supabase Auth with role-based access control (RLS).

- Hosting: Secure cloud deployment with staging and production
  environments.

## 4. Performance Requirements

- Grid map load time \< 2 seconds on standard mobile connection.

- Public grid access without authentication.

- Geometry simplified for mobile rendering.

- Monthly minimum score recalculation cadence.

- Efficient spatial indexing using GIST indexes.

- Cluster detection must execute within acceptable batch runtime.

## 5. Security & Privacy

- Public layer: grids, tiers, cluster states only.

- Private layer: property polygons visible only to owner and authorised
  roles.

- Row Level Security (RLS) policies enforced in Supabase.

- Audit log immutable and append-only.

- No exposure of RP ownership data publicly.

## 6. Versioning & Governance

- Model version labelled BPPF_v1.0 at launch.

- Each Score_Run stores dataset versions and timestamp.

- Any change to scoring logic requires version increment.

- All mission deltas logged and verifiable.

- Confidence score displayed alongside Priority Index.

## 7. Deliverables Expected from Emergent

- Fully functional Supabase project with PostGIS enabled.

- Deterministic grid generation within 4.2 km boundary.

- Edge-only adjacency table generated.

- BPPF scoring engine operational with Explain payload.

- Mission engine lifecycle implemented.

- Cluster detection and Collective Uplift Index functioning.

- Public map accessible without login.

- Comprehensive staging → production deployment.

## Founder Intent Statement

Brookfield is the symbolic birthplace of the Terra ecological operating
system. Transparency, auditability, and landscape-first coordination are
foundational. All implementation decisions must strengthen Terra as a
repeatable ecological coordination model.
