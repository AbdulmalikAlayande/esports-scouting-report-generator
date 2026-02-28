# RFC-0002: Phase 3 Storage Plane Separation

## Status
Accepted (Phase 3)

## Context
Phase 2 introduced orchestration state and idempotent job execution. The storage model still mixed ingestion, transform, and report concerns into a single report payload path.

## Decision
Separate persistence into additive storage planes keyed by `report_request_id`:

1. Raw Plane: `report_raw_payloads`
- Stores source snapshots captured during ingestion.
- Keyed by `(report_request_id, payload_key)`.

2. Normalized Plane: `report_normalized_payloads`
- Stores normalized/intermediate structures used by transforms.
- Keyed by `(report_request_id, payload_key)`.

3. Feature Plane: `report_feature_payloads`
- Stores transform feature outputs and tactical aggregates.
- Includes `feature_version` for feature lineage.
- Keyed by `(report_request_id, payload_key)`.

4. Report Plane: `report_artifacts`
- Stores final composed report artifact plus lineage metadata (`contract_version`, `model_version`, `feature_version`).
- One row per request.

## Compatibility
- Existing `scouting_reports` remains in place for current API reads.
- Engine now dual-writes final output to both `scouting_reports` (legacy compatibility) and `report_artifacts` (new report plane).
- API endpoints and response contracts remain unchanged.

## Implementation Notes
- Plane writes are idempotent upserts by request + payload key.
- Worker persists raw/normalized/feature planes before synthesis/composition.
- Worker persists final report artifact after composition.
- Migrations are mirrored in both repos to keep bootstrap symmetry:
  - `scouting-engine/migrations/006_add_storage_planes.sql`
  - `scouting-api/src/main/resources/migrations/012_add_storage_planes.sql`

## Deferred
- TTL/retention policies per plane.
- Compression and archival for large raw payloads.
- Dedicated read APIs for each storage plane.
