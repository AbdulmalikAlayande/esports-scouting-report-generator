# RFC-0001: Esports Scouting System V2 Control Plane and Contract Baseline

## Status
Accepted (Phase 1 baseline)

## Context
The current system works but has coupling across API, worker execution, and frontend contract parsing:
- The frontend consumes loosely structured report/status payloads.
- Worker and API status lifecycles are mapped to a small legacy state set.
- Error handling and retryability are implicit in free-form error strings.
- Contract evolution lacks explicit version signaling.

This RFC establishes the production baseline without breaking existing clients.

## Decisions
1. The Java API owns workflow lifecycle/state mapping and public response contracts.
2. Existing endpoints remain unchanged:
   - `POST /api/reports/generate`
   - `GET /api/reports/{id}/status`
   - `GET /api/reports/{id}`
3. Additive contract fields are introduced in v1 responses:
   - `contractVersion`
   - `workflowState` (status endpoint)
   - `errorCode`, `retryable` (status endpoint)
   - `modelVersion`, `featureVersion`, `generatedAt`, `lineage` (report endpoint)
4. Legacy fields stay intact (`status`, `progress`, `currentStep`, `sections`, etc.).
5. Canonical workflow state enum is introduced:
   - `QUEUED`, `INGESTING`, `FEATURIZING`, `SYNTHESIZING`, `COMPOSING`, `READY`, `FAILED`, `CANCELLED`
6. Legacy status mapping used in Phase 1:
   - `PENDING` -> `QUEUED`
   - `PROCESSING` -> `INGESTING`
   - `COMPLETED` -> `READY`
   - `FAILED` -> `FAILED`
7. Error taxonomy baseline:
   - `RETRYABLE_PROVIDER`
   - `RETRYABLE_INFRA`
   - `NON_RETRYABLE_DATA`
   - `NON_RETRYABLE_CONTRACT`
   - `NON_RETRYABLE_AUTH`
   - `NON_RETRYABLE_CONFIG`

## Public Contract Versions
- Status response version: `report-status.v1`
- Scouting report response version: `scouting-report.v1`

## Compatibility Guarantees
1. Existing frontend polling behavior continues to work with no endpoint changes.
2. New fields are additive and optional from consumer perspective.
3. Legacy status values and existing section rendering semantics remain preserved.

## Implementation Scope in this Phase
- Add contract/version/state/error primitives in `scouting-api`.
- Populate additive fields in service mapping layer.
- Update frontend TS interfaces to accept additive fields.
- Add schema artifacts under `contracts/schemas`.
- Add compatibility tests to ensure legacy+additive coexistence.

## Deferred to Later Phases
- Postgres workflow FSM tables and claim/lease semantics.
- Idempotency keys and retry scheduling.
- WebSocket event delivery.
- Feature registry and full report composer separation.
