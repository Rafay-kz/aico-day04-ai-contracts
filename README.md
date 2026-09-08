# AICO Day 4 — Structured AI Contracts

Day 4 sits on the Day 3 Model Gateway. The gateway is still the only model-call boundary. Day 4 decides what the application is allowed to accept back from a model: a **typed Pydantic contract** or a **typed failure**. Never an unchecked dictionary.

## Setup

Never commit, zip, or share `.venv` or `.env`. Recreate the venv from `requirements.txt`. Put the Foundry endpoint and optional API key in `.env` (see `.env.example`).

Run commands from the **project root**.

**macOS / Linux (bash or zsh)**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export PYTHONPATH=src
```

**Windows (PowerShell)**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:PYTHONPATH = "src"
```

Chat alias for this lab is `gpt-4.1-mini`. Live chat uses the Foundry **Responses** API through `FoundryAdapter`. Failure-path tests use `FakeTransport` and make no network calls.

## Commands

Run from the project root after activating the venv (see Setup). Day 4 failure-path tests use fixtures and a fake gateway. They make no network calls.

**macOS / Linux (bash or zsh)**

```bash
export PYTHONPATH=src
python -c "from aico.contracts.models import export_json_schemas; export_json_schemas()"
pytest -q tests/test_day04_contracts.py tests/test_day04_semantic_validation.py tests/test_day04_repair.py tests/test_day04_compatibility.py
pytest -q
```

**Windows (PowerShell)**

```powershell
$env:PYTHONPATH = "src"
python -c "from aico.contracts.models import export_json_schemas; export_json_schemas()"
pytest -q tests/test_day04_contracts.py tests/test_day04_semantic_validation.py tests/test_day04_repair.py tests/test_day04_compatibility.py
pytest -q
```

Day 1–3 retrieval and gateway commands (regression):

**macOS / Linux (bash or zsh)**

```bash
export PYTHONPATH=src
python -m aico.retrieval.ingest --input data/documents --out data/index --tokens 300 --overlap 50
python -m aico.retrieval.embed --index data/index --out data/vectors
python -m aico.retrieval.search --query "termination notice period" --mode hybrid --top-k 5
pytest -q tests/test_model_gateway.py tests/test_model_gateway_retry.py tests/test_model_gateway_routing.py
```

**Windows (PowerShell)**

```powershell
$env:PYTHONPATH = "src"
python -m aico.retrieval.ingest --input data/documents --out data/index --tokens 300 --overlap 50
python -m aico.retrieval.embed --index data/index --out data/vectors
python -m aico.retrieval.search --query "termination notice period" --mode hybrid --top-k 5
pytest -q tests/test_model_gateway.py tests/test_model_gateway_retry.py tests/test_model_gateway_routing.py
```

## Validation pipeline

```text
raw model text
  → bounded markdown unwrap (opening and closing fence required)
  → parse JSON
  → contract / schema validation (Pydantic)
  → semantic validation (S1–S5)
  → typed CitedAnswer
```

If contract or semantic validation fails, **one** repair call may go through the Model Gateway. The repaired text is revalidated. Parse failures are non-repairable. Repair never loops.

Markdown-wrapped JSON is accepted only when the entire payload is a single fenced block with both an opening fence and a closing fence. An incomplete fence is not unwrapped. Arbitrary prose around JSON is rejected.

## Layout

```
src/aico/
  platform/                 Day 3 Model Gateway (still the only SDK boundary)
  contracts/
    models.py               CitedAnswer + ResponseEnvelope
    errors.py               typed failures
    validator.py            parse + schema only
    semantic.py             S1–S5
    repair.py               one gateway repair
    service.py              pipeline
contracts/schema/
  cited_answer.v1.schema.json
  response_envelope.v1.schema.json
tests/fixtures/day04/
tests/test_day04_*.py
artifacts/day04/validation_report.md
```

Day 4 does not implement grounded answering, citation verification against retrieved chunks, or prompt-injection defense.
