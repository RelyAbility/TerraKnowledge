# Terra -- Performance Envelope & Cost Model v1.0

Purpose: Define expected performance, throughput, storage and compute
envelope for Terra Phase 1, and provide a scaling model for later
phases. Assumes monthly composites (not near real-time).

## 1. Coverage Assumptions

• Primary AOI: 10 km radius flagship (\~314 km²) and/or pilot AOIs
(10--25 km²).

• Update cadence: monthly composites.

• Products stored: derived rasters + plot summaries; avoid raw scene
retention where possible.

## 2. Storage Targets

• Store monthly NDVI composite tiles (COG) and quality layers.

• Store plot summaries (DB rows) and briefs (JSON) for fast mobile
retrieval.

• Offline cache on device: latest brief + small thumbnails + missions;
not full imagery.

## 3. Processing Throughput

Monthly job steps per AOI: ingest → mask → index → composite → change →
aggregate → publish.

Target SLA (Phase 1): publish monthly briefs within 24 hours of
month-end processing trigger.

Concurrency: process plots in parallel; compute raster composites by
tile.

Backfill: initial 12 months of history can be processed as a batch job
with throttling.

## 4. Latency Targets (Mobile APIs)

• GET /plots list: p95 \< 300 ms.

• GET /plots/{id}/briefs: p95 \< 500 ms (paginated).

• GET /briefs/{id}: p95 \< 400 ms.

• Sync events: accept batch in \< 1 s; async processing allowed.

## 5. Cost Model (Order-of-Magnitude)

Costs scale \~linearly with AOI area and number of derived layers:
O(area × layers × months).

Primary cost drivers in Phase 1 are engineering and QA, not storage.

Cloud costs typically remain low at flagship scale when storing derived
products only.

Recommended cost controls:

• Avoid raw imagery storage; compute from cloud-native sources when
possible.

• Store only monthly composites and summaries.

• Use spot/batch compute for backfills.

## 6. Scaling Notes (Phase 2+)

• Acoustic ingestion increases event volume significantly; design with
streaming + partitions.

• Portfolios across multiple regions may require partitioning and
caching strategy.

• Add rate limiting and per-org quotas for bulk exports and reprocessing
triggers.
