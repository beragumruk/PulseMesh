# PulseMesh

PulseMesh is a distributed critical-care telemetry platform for ingesting physiologic signals, prioritizing alarm response, and preserving auditability across the inference path. The system combines high-frequency event streaming, policy-constrained risk scoring, and proof-oriented artifact generation in a single research and engineering stack.

## Why PulseMesh

Modern bedside monitoring pipelines generate more alarms than care teams can review with equal attention. PulseMesh focuses on the systems problem behind that burden:

- low-latency telemetry transport for numeric observations, waveforms, and alarm events
- conservative inference that can assist prioritization without bypassing hard safety rules
- privacy-preserving downstream artifacts that avoid exposing raw feature vectors
- operator-facing visualization for replay, inspection, and explanation workflows

## System Overview

PulseMesh is organized as a multi-service monorepo:

- `apps/web`: Next.js operator console for telemetry playback, waveform inspection, and alarm review workflows
- `services/gateway-rust`: Axum-based event gateway that accepts ingest traffic and fans out normalized envelopes over WebSocket
- `services/inference-python`: FastAPI inference service for feature extraction, actionability scoring, and neuro-symbolic routing
- `services/proof-service-node`: proof artifact service that commits feature vectors and returns verifiable payloads for downstream audit checks
- `infra/db/migrations`: PostgreSQL, TimescaleDB, and pgvector schema initialization

At runtime, the platform follows a straightforward control path:

1. Telemetry enters the Rust gateway through HTTP ingest or simulated streaming.
2. The web client subscribes to normalized gateway envelopes over WebSocket.
3. The inference service derives compact features, scores alarm actionability, and applies policy overrides before returning a routing decision.
4. The proof service emits deterministic commitment artifacts tied to the scored event.
5. Persisted time-series and relational records support replay, review, and operational analysis.

## Architectural Characteristics

- **Low-latency transport**: Rust gateway with broadcast fan-out for real-time consumers
- **Conservative decision policy**: explicit hard-rule overrides for critical and sustained deterioration scenarios
- **Separation of concerns**: gateway, inference, proof, storage, and UI are independently deployable services
- **Audit-oriented outputs**: proof payloads preserve verification signals without retaining unnecessary raw detail in downstream consumers
- **Research-friendly stack**: simulation, replay, and explainability surfaces are built into the platform rather than bolted on afterward

## Repository Layout

```text
.
|-- apps/web
|-- services/gateway-rust
|-- services/inference-python
|-- services/proof-service-node
|-- infra/db/migrations
|-- db
|-- docs
`-- scripts
```

## Local Development

### Docker Compose

```bash
cp .env.example .env
docker compose up --build
```

Default service endpoints in the committed repository configuration:

- Web UI: `http://localhost:3000`
- Gateway API and WebSocket: `http://localhost:8080`
- Inference API: `http://localhost:8000`
- Proof service: `http://localhost:7000`
- PostgreSQL: `localhost:5432`

### Service-by-service

Rust gateway:

```bash
cd services/gateway-rust
cargo run
```

Python inference:

```bash
cd services/inference-python
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Node proof service:

```bash
cd services/proof-service-node
npm install
npm run dev
```

Web console:

```bash
cd apps/web
npm install
npm run dev
```

Database migrations:

```bash
bash scripts/run-migrations.sh
```

## Safety and Verification Notes

- The inference path is intentionally policy-constrained. Model output is advisory and can be overridden by hard clinical safety rules.
- Explanation payloads are feature-level summaries rather than raw patient data dumps.
- Proof generation is deterministic and structured for audit demonstrations and systems integration.

## Provenance

PulseMesh originated as a HackTJ 2026 prototype and has since been expanded into a production-style research platform.
