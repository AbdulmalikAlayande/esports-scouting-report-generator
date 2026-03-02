# Esports Scouting Report Generator

Polyglot scouting system that turns natural-language prompts into tactical esports reports.

## What This Repository Is

This root repository is a **submodule orchestrator**. It defines how the platform fits together and links to shared contracts/RFCs, while active implementation lives in submodules.

Submodule references are pinned in [`.gitmodules`](.gitmodules).

## What This Repository Is Not

This root repo is not the main implementation surface for API, worker, or UI internals. Do not place module-specific production logic here.

## System Modules

- [`scouting-api`](https://github.com/AbdulmalikAlayande/esports-scouting-report-generator-api): Java/Spring Boot control plane, report lifecycle/status API, and read-model delivery.
- [`scouting-engine`](https://github.com/AbdulmalikAlayande/esports-scouting-report-generator-engine): Python worker pipeline for ingestion, featurization, synthesis, and report composition.
- [`scouting-ui`](https://github.com/AbdulmalikAlayande/esports-scouting-report-generator-ui): Next.js frontend for report submission, status tracking, and report viewing.

## Architecture Snapshot

```text
User (Next.js UI)
    -> Java API (request validation + orchestration)
        -> Job lifecycle (QUEUED -> INGESTING -> FEATURIZING -> SYNTHESIZING -> COMPOSING -> READY/FAILED)
            -> Python worker stages
                -> Storage planes (raw, normalized, feature, report artifact)
                    -> API read model
                        -> UI report retrieval
```

At a high level:
- API owns workflow state and public response contracts.
- Worker owns ingestion/transforms/synthesis/composition execution.
- Contract and lineage fields keep frontend compatibility while backend evolves.

## Contracts and RFCs

- [Contracts Overview](./contracts/README.md)
- [Report Status Contract v1](./contracts/schemas/report-status.v1.schema.json)
- [Scouting Report Contract v1](./contracts/schemas/scouting-report.v1.schema.json)
- [RFC-0001: Architecture v2 Baseline](./docs/RFC-0001-architecture-v2-baseline.md)
- [RFC-0002: Storage Plane Separation](./docs/RFC-0002-storage-planes.md)

## Quick Start (Orchestrator-Level)

1. Clone with submodules:

```bash
git clone --recurse-submodules <your-repo-url>
```

2. If already cloned, initialize/update submodules:

```bash
git submodule update --init --recursive
```

3. Start system components in this order:
- Start PostgreSQL.
- Start API (`scouting-api`).
- Start worker (`scouting-engine`).
- Start UI (`scouting-tool`).

Use each module README for exact setup commands and environment variables.

## Module Entry Points

### API (`scouting-api`)
- [API README](./scouting-api/README.md)
- [API Test Commands](./scouting-api/API_TEST_COMMANDS.md)

### Worker (`scouting-engine`)
- [Worker README](./scouting-engine/README.md)
- Worker control entrypoint from `scouting-engine`:

```bash
python main.py on
python main.py off
python main.py status
python main.py once
```

### UI (`scouting-tool`)
- [UI README](./scouting-tool/README.md)

## Current Phase Snapshot

- Phase 1 complete: contracts and workflow-state baseline.
- Phase 2 complete: orchestration and idempotent job execution.
- Phase 3 complete: storage-plane separation (raw/normalized/feature/report).
- Phase 4 complete: feature-registry/report-composer direction and contract hardening in progress.
- Phase 5 pending: real-time status delivery, scale tuning, and SLO enforcement.

## Contributing (Submodule Workflow)

1. Make implementation changes inside the relevant submodule:
- `scouting-api`
- `scouting-engine`
- `scouting-tool`

2. Commit and push in that submodule first.
3. Return to root repo and commit updated submodule pointers.

This keeps module history clean while preserving orchestrator-level traceability.
