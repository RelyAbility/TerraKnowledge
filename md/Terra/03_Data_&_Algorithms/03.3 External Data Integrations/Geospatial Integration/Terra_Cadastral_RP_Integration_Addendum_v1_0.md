# Terra -- Cadastral (RP) Integration Addendum v1.0

Purpose: This addendum formalises Terra's approach to cadastral
alignment (Registered Plan -- RP) integration for South East Queensland
and future regions. It updates the Product, API, and Database
specifications to support dual entry (draw or RP input) with canonical
parcel linkage and sub-RP restoration overlays.

## 1. Strategic Position

Terra supports both user-drawn boundaries and RP-number entry.
Internally, plots may optionally link to a canonical cadastral parcel.
Sub-RP restoration zones are explicitly supported (Model C).

Design Principles:

• User flexibility at onboarding.

• Canonical cadastral alignment internally.

• Support for full RP + sub-RP restoration overlays.

• No compliance-heavy messaging at sign-up.

## 2. User Experience Model

Option A -- Enter RP Number:

• User inputs RP identifier.

• System validates against cadastral dataset.

• Official boundary loads automatically.

• plot.rp_verified = true.

Option B -- Draw Boundary:

• User draws polygon.

• System checks overlap with cadastral parcels.

• If \>80% overlap with single RP → suggest match.

• If multiple overlaps → present selection.

• If declined → store as custom polygon (rp_verified = false).

Sub-RP Restoration Zones (Model C):

• A plot may represent full RP or a sub-area.

• If sub-area, system stores both:

\- Canonical RP geometry reference

\- User-defined restoration overlay geometry

• Enables full-property validation later without losing sub-zone
specificity.

## 3. Database Schema Updates

ALTER TABLE plots

ADD COLUMN rp_number TEXT NULL,

ADD COLUMN rp_verified BOOLEAN DEFAULT false,

ADD COLUMN rp_overlap_pct DOUBLE PRECISION,

ADD COLUMN parent_rp_geom GEOMETRY(POLYGON, 4326);

CREATE TABLE cadastral_parcels (

rp_number TEXT PRIMARY KEY,

geom GEOMETRY(POLYGON, 4326) NOT NULL,

area_m2 DOUBLE PRECISION NOT NULL,

lga TEXT,

lot TEXT,

plan TEXT

);

CREATE INDEX cadastral_geom_gix ON cadastral_parcels USING GIST (geom);

Notes:

• plots.geom = restoration zone (may be full RP or subset).

• parent_rp_geom stores canonical boundary when linked.

• rp_overlap_pct documents confidence of match.

• Structure supports future full-RP validation.

## 4. API Contract Updates

POST /v1/plots accepts either geometry or rp_number, or both.

Response includes rp_number, rp_verified, and rp_overlap_pct.

New endpoint: GET /v1/cadastral/{rp_number} returns canonical parcel
geometry and metadata.

## 5. Reporting & Density Analytics Implications

Enables measurement of:

• % of RPs onboarded within flagship zone.

• Sensor density per RP cluster.

• Participation coverage by cadastral area.

• Full-property vs sub-property restoration tracking.

• Future Nature Repair Market parcel alignment.

## 6. Governance Safeguards

• RP linkage optional at onboarding.

• Messaging focuses on ecological alignment, not compliance.

• No ownership verification stored in Phase 1.

• RP data treated as spatial metadata only.

## 7. Phase 2 & 3 Readiness

• Supports portfolio aggregation by legal parcel.

• Enables Elite evaluation at full-property scale.

• Reduces retrofitting risk as market rules mature.
