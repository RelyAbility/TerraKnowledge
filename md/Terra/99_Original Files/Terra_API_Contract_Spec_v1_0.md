# Terra -- API Contract Specification v1.0

Purpose: Define the external and internal API contracts for Terra Phase
1 mobile app, including request/response schemas, error handling, and
versioning. These contracts are designed to minimise rework and enable
parallel mobile/backend development.

## 1. Conventions

• All endpoints are versioned under /v1/

• All timestamps are ISO-8601 UTC unless specified.

• All geometries use WGS84 (EPSG:4326).

• Numeric fields specify units explicitly.

• Every response includes method_version and data_quality where
applicable.

• Idempotency: POST endpoints that create resources accept
Idempotency-Key header.

## 2. Authentication

• Bearer token (JWT) in Authorization header.

• Roles: user, org_admin, reviewer, system.

• Row-level permissions enforced by org_id/portfolio_id membership.

## 3. Core Resource Models (JSON)

### Plot

{\
\"plot_id\": \"uuid\",\
\"org_id\": \"uuid\",\
\"name\": \"string\",\
\"geometry\": {\
\"type\": \"Polygon\",\
\"coordinates\": \"\[\...\]\"\
},\
\"area_m2\": \"number\",\
\"re_mix\": \[\
{\
\"re_code\": \"string\",\
\"pct\": \"number\"\
}\
\],\
\"created_at\": \"datetime\",\
\"updated_at\": \"datetime\"\
}

### MonthlyBrief

{\
\"brief_id\": \"uuid\",\
\"plot_id\": \"uuid\",\
\"month\": \"YYYY-MM\",\
\"tier_level\": \"1\|2\|3\",\
\"metrics\": {\
\"ndvi_median\": \"number\",\
\"ndvi_iqr\": \"number\",\
\"het_ndvi_median\": \"number\",\
\"valid_pixel_fraction\": \"number\"\
},\
\"change\": {\
\"delta_mom\": \"number\|null\",\
\"delta_yoy\": \"number\|null\",\
\"disturbance_flag\": \"boolean\",\
\"disturbance_reason\": \"string\|null\"\
},\
\"confidence\": {\
\"confidence_score\": \"0..1\",\
\"confidence_band\": \"Low\|Medium\|High\",\
\"inputs\": {\
\"valid_pixel_fraction\": \"number\",\
\"time_series_depth_months\": \"int\",\
\"consistency_score\": \"0..1\"\
}\
},\
\"data_quality\": {\
\"status\": \"OK\|INSUFFICIENT_CLEAR_SKY\|NO_DATA\",\
\"valid_pixel_fraction\": \"number\",\
\"notes\": \"string\|null\"\
},\
\"method_version\": \"string\",\
\"published_at\": \"datetime\"\
}

### Mission

{\
\"mission_id\": \"uuid\",\
\"plot_id\": \"uuid\",\
\"mission_type\":
\"MIDSTOREY_INFILL\|EDGE_BUFFER\|UNDERSTOREY_DENSITY\|WEED_SUPPRESSION\|CORRIDOR_RECONNECT\",\
\"title\": \"string\",\
\"objective\": \"string\",\
\"geometry\": {\
\"type\": \"Polygon\|Point\|LineString\",\
\"coordinates\": \"\[\...\]\"\
},\
\"recommended_species\": \[\
{\
\"scientific\": \"string\",\
\"common\": \"string\",\
\"layer\": \"CANOPY\|MID\|GROUND\",\
\"provenance\": \"string\"\
}\
\],\
\"tasks\": \[\
{\
\"task_id\": \"uuid\",\
\"label\": \"string\",\
\"status\": \"TODO\|DONE\",\
\"evidence_required\": \"boolean\"\
}\
\],\
\"status\": \"DRAFT\|ACTIVE\|COMPLETED\|CANCELLED\",\
\"created_by\": \"uuid\",\
\"created_at\": \"datetime\",\
\"updated_at\": \"datetime\"\
}

### OfflineSyncEvent

{\
\"event_id\": \"uuid\",\
\"device_id\": \"string\",\
\"user_id\": \"uuid\",\
\"event_type\": \"MISSION_TASK_UPDATE\|NOTE\|PHOTO\",\
\"payload\": \"object\",\
\"created_at\": \"datetime\",\
\"synced_at\": \"datetime\|null\",\
\"status\": \"QUEUED\|SYNCED\|FAILED\"\
}

## 4. Endpoints

• POST /v1/auth/login --- Email/password or magic link exchange for
token.

• GET /v1/me --- Return current user profile, orgs, roles, notification
preferences.

• POST /v1/plots --- Create plot polygon.

• GET /v1/plots?bbox=&search=&page= --- List plots accessible to user.

• GET /v1/plots/{plot_id} --- Get plot details including RE mix.

• PUT /v1/plots/{plot_id} --- Update plot name/geometry (versioned).

• GET /v1/plots/{plot_id}/briefs?from=YYYY-MM&to=YYYY-MM --- List
monthly briefs.

• GET /v1/briefs/{brief_id} --- Get a single brief.

• POST /v1/plots/{plot_id}/briefs/refresh?month=YYYY-MM --- Trigger
(re)processing for month (admin/system).

• GET /v1/plots/{plot_id}/missions --- List missions for a plot.

• POST /v1/plots/{plot_id}/missions --- Create mission from
recommendation template.

• PUT /v1/missions/{mission_id} --- Update mission details/status.

• POST /v1/missions/{mission_id}/tasks/{task_id} --- Update task status;
supports offline sync replay.

• POST /v1/sync/events --- Upload queued offline events batch.

• GET /v1/notifications/preferences --- Get notification preferences.

• PUT /v1/notifications/preferences --- Set notification preferences.

• POST /v1/notifications/test --- Send test push notification
(dev/staging).

## 5. Error Model

{\
\"error\": {\
\"code\": \"string\",\
\"message\": \"string\",\
\"details\": \"object\|null\",\
\"request_id\": \"string\"\
}\
}

Standard codes: 400 INVALID_REQUEST, 401 UNAUTHENTICATED, 403 FORBIDDEN,
404 NOT_FOUND, 409 CONFLICT, 422 VALIDATION_FAILED, 429 RATE_LIMITED,
500 INTERNAL_ERROR.

## 6. Versioning

• API version in path (/v1/).

• method_version stored on derived outputs (briefs/metrics).

• Breaking changes require /v2/ with migration plan.
