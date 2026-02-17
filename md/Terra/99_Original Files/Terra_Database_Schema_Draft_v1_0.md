# Terra -- Database Schema Draft v1.0

Purpose: Provide a concrete relational schema (PostgreSQL + PostGIS
recommended) to support Terra Phase 1 with future-phase readiness
(Tier/Confidence/Elite scaffolding).

## 1. Core Tables (DDL-style)

\-- organisations

CREATE TABLE organisations (

org_id UUID PRIMARY KEY,

name TEXT NOT NULL,

created_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

\-- users

CREATE TABLE users (

user_id UUID PRIMARY KEY,

org_id UUID NOT NULL REFERENCES organisations(org_id),

email TEXT NOT NULL UNIQUE,

display_name TEXT,

role TEXT NOT NULL CHECK (role IN
(\'user\',\'org_admin\',\'reviewer\',\'system\')),

created_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

\-- plots (polygons)

CREATE TABLE plots (

plot_id UUID PRIMARY KEY,

org_id UUID NOT NULL REFERENCES organisations(org_id),

name TEXT NOT NULL,

geom GEOMETRY(POLYGON, 4326) NOT NULL,

area_m2 DOUBLE PRECISION NOT NULL,

re_mix JSONB, \-- \[{re_code, pct}\]

created_by UUID REFERENCES users(user_id),

created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

updated_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

CREATE INDEX plots_geom_gix ON plots USING GIST (geom);

\-- zones (e.g., 10km flagship)

CREATE TABLE zones (

zone_id UUID PRIMARY KEY,

name TEXT NOT NULL,

geom GEOMETRY(POLYGON, 4326) NOT NULL,

created_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

CREATE INDEX zones_geom_gix ON zones USING GIST (geom);

\-- plot-zone membership (supports many-to-many)

CREATE TABLE plot_zones (

plot_id UUID REFERENCES plots(plot_id),

zone_id UUID REFERENCES zones(zone_id),

PRIMARY KEY (plot_id, zone_id)

);

\-- portfolios (catchment/council)

CREATE TABLE portfolios (

portfolio_id UUID PRIMARY KEY,

org_id UUID NOT NULL REFERENCES organisations(org_id),

name TEXT NOT NULL,

portfolio_type TEXT CHECK (portfolio_type IN
(\'catchment\',\'council\',\'sponsor_cluster\',\'other\')),

created_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

CREATE TABLE portfolio_plots (

portfolio_id UUID REFERENCES portfolios(portfolio_id),

plot_id UUID REFERENCES plots(plot_id),

PRIMARY KEY (portfolio_id, plot_id)

);

\-- method versions

CREATE TABLE method_versions (

method_version TEXT PRIMARY KEY,

description TEXT,

released_at TIMESTAMPTZ NOT NULL DEFAULT now(),

parameters JSONB

);

\-- monthly briefs (derived)

CREATE TABLE monthly_briefs (

brief_id UUID PRIMARY KEY,

plot_id UUID NOT NULL REFERENCES plots(plot_id),

month TEXT NOT NULL, \-- YYYY-MM

tier_level INT NOT NULL CHECK (tier_level IN (1,2,3)),

ndvi_median DOUBLE PRECISION,

ndvi_iqr DOUBLE PRECISION,

het_ndvi_median DOUBLE PRECISION,

valid_pixel_fraction DOUBLE PRECISION,

delta_mom DOUBLE PRECISION,

delta_yoy DOUBLE PRECISION,

disturbance_flag BOOLEAN NOT NULL DEFAULT false,

disturbance_reason TEXT,

data_quality_status TEXT NOT NULL CHECK (data_quality_status IN
(\'OK\',\'INSUFFICIENT_CLEAR_SKY\',\'NO_DATA\')),

method_version TEXT NOT NULL REFERENCES method_versions(method_version),

published_at TIMESTAMPTZ,

created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

UNIQUE (plot_id, month, method_version)

);

\-- confidence object (stored separately for audit)

CREATE TABLE confidence_scores (

confidence_id UUID PRIMARY KEY,

brief_id UUID NOT NULL REFERENCES monthly_briefs(brief_id) ON DELETE
CASCADE,

confidence_score DOUBLE PRECISION NOT NULL,

confidence_band TEXT NOT NULL CHECK (confidence_band IN
(\'Low\',\'Medium\',\'High\')),

inputs JSONB NOT NULL, \-- {valid_pixel_fraction,
time_series_depth_months, consistency_score, \...}

created_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

\-- elite scaffolding (dormant Phase 1)

CREATE TABLE elite_status (

plot_id UUID PRIMARY KEY REFERENCES plots(plot_id),

elite_eligible BOOLEAN NOT NULL DEFAULT false,

elite_review_status TEXT NOT NULL DEFAULT \'not_started\' CHECK
(elite_review_status IN
(\'not_started\',\'in_review\',\'approved\',\'rejected\')),

last_evaluated_at TIMESTAMPTZ

);

\-- missions

CREATE TABLE missions (

mission_id UUID PRIMARY KEY,

plot_id UUID NOT NULL REFERENCES plots(plot_id),

mission_type TEXT NOT NULL,

title TEXT NOT NULL,

objective TEXT,

geom GEOMETRY(GEOMETRY, 4326),

recommended_species JSONB,

status TEXT NOT NULL CHECK (status IN
(\'DRAFT\',\'ACTIVE\',\'COMPLETED\',\'CANCELLED\')),

created_by UUID REFERENCES users(user_id),

created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

updated_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

CREATE INDEX missions_geom_gix ON missions USING GIST (geom);

CREATE TABLE mission_tasks (

task_id UUID PRIMARY KEY,

mission_id UUID NOT NULL REFERENCES missions(mission_id) ON DELETE
CASCADE,

label TEXT NOT NULL,

status TEXT NOT NULL CHECK (status IN (\'TODO\',\'DONE\')),

evidence_required BOOLEAN NOT NULL DEFAULT false,

updated_at TIMESTAMPTZ NOT NULL DEFAULT now()

);

\-- offline sync events

CREATE TABLE offline_events (

event_id UUID PRIMARY KEY,

device_id TEXT NOT NULL,

user_id UUID NOT NULL REFERENCES users(user_id),

event_type TEXT NOT NULL,

payload JSONB NOT NULL,

status TEXT NOT NULL CHECK (status IN
(\'QUEUED\',\'SYNCED\',\'FAILED\')),

created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

synced_at TIMESTAMPTZ

);

## 2. Indexing & Performance

• PostGIS GiST index on plot and mission geometries for spatial queries.

• UNIQUE(plot_id, month, method_version) ensures reproducibility and
multi-version support.

• Consider BRIN index on monthly_briefs.created_at for large
time-series.

• Partition monthly_briefs by month if scaling to many regions.

## 3. Future Tables (Phase 2+ placeholders)

• acoustic_observations (sensor_id, timestamp, species/guild,
confidence)

• sensors (sensor_id, provider, location, api_key_ref, status)

• external_reviews (review_id, plot/zone/portfolio, decision, notes,
artefacts)
