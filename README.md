# IRIS

This project provides a web frontend and Python backend utilities to audit and manage OSCAL/JSON artifacts for the german OSCAL/JSON controls catalogs.

Key features (per mission):
- Import OSCAL/JSON catalogs such as the [Grundschutz++ catalog](https://github.com/BSI-Bund/Stand-der-Technik-Bibliothek/tree/main/Kompendien/Grundschutz%2B%2B-Kompendium)
- Explore control relationships as a clickable graph
- Bootstrap/import/filter/edit/export OSCAL System Security Plans (SSP)
- Import component-definitions into SSPs with preview
- Manage assessment plans, assessment results, and POAMs
- Import/export a simple asset database and map to Zielobjekt classes

See `docs/technical_design.md` and ADRs for decisions and architecture.

Quick start:
1. Configure `config.yaml` (or point CLI to an alternate file with `--config path/to/config.yaml`).
2. Run self-test: `python3 oss-requirements-auditor.py --config config.yaml selftest`
3. Visit `http://127.0.0.1:8080/` for the Mission Control dashboard (after starting the API/compose stack) to inspect health + quick links.
4. Run tests: `python3 tests/test_harness/runner.py`
5. Import/validate catalog locally:
   - `python3 oss-requirements-auditor.py catalog-validate tests/data/fixtures/catalog_merge_a.json`
   - `python3 oss-requirements-auditor.py catalog-import tests/data/fixtures/catalog_merge_test.zip`
   - Inspect conflicts: `python3 oss-requirements-auditor.py catalog-merge-report`
6. Manage assets: `python3 oss-requirements-auditor.py assets import tests/data/fixtures/assets.csv` then export via `assets export --format csv --output var/assets.csv --force`.
7. Generate SSPs from cached catalog: `python3 oss-requirements-auditor.py ssp-bootstrap --mode prozesse`.
8. Start API (serve FastAPI/uvicorn): `python3 oss-requirements-auditor.py serve`.

## Container Usage

Build and run the containerized API (non-root, offline-ready):

```bash
# Build image
docker build -t oss-requirements-auditor:local .

# Run API with persistent var/ + config.yaml (image defaults OSSRA_HOST=0.0.0.0)
docker run --rm -p 8080:8080 \
  -v "$PWD/var:/app/var" \
  -v "$PWD/config.yaml:/app/config.yaml:ro" \
  oss-requirements-auditor:local serve
```

Or use Docker Compose (bind-mounts `./var` and `config.yaml` automatically):

```bash
docker compose up --build
# CLI commands
docker compose run --rm ossra catalog-merge-report
# Smoke tests inside container
docker compose run --rm --entrypoint python3 ossra tests/test_harness/runner.py
```
