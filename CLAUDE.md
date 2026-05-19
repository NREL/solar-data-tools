# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Solar Data Tools is an open-source Python library for analyzing PV power and irradiance time-series data. It uses statistical signal processing to analyze unlabeled PV data (no model, weather data, or performance index required).

**Monorepo with four packages:**
- `solardatatools/` — Core library. Entry point: `DataHandler.run_pipeline()` for the main processing pipeline (preprocessing, cleaning, clear-day detection, clipping detection, capacity change detection)
- `pvsystemprofiler/` — System parameter estimation (latitude, longitude, tilt, azimuth fitting from unlabeled data)
- `sdt_dask/` — Dask-based parallelization layer for running SDT pipelines at scale on local or cloud infrastructure (AWS Fargate, Azure VMs). Three components: ClientPlug (cluster setup), DataPlug (data retrieval), Runner (orchestration)
- `anomalydetector/` — Outage detection via `OutagePipeline` and `MultiDataHandler` for training/testing splits

## Development Commands

| Task | Command |
|------|---------|
| Install in editable mode | `pip install -e .` |
| Install with dev extras | `pip install -e ".[dev]"` (adds ruff, pre-commit) |
| Install with docs extras | `pip install -e ".[docs]"` |
| Install with Dask support | `pip install -e ".[dask]"` |
| Install with MOSEK solver | `pip install -e ".[mosek]"` |
| Run tests | `python -m unittest discover -v` (from repo root) |
| Run a single test | `python -m unittest tests.solardatatools.test_data_handler -v` |
| Run lint & format | `pre-commit run --all-files` (requires `pip install pre-commit`) |
| Build docs locally | `cd docs && make html` |

**Linter:** ruff (via pre-commit hooks). Configure in `.pre-commit-config.yaml`. Format and fix are applied on commit.

**Test framework:** `unittest` (standard library). Tests live in `tests/` mirroring package structure (`solardatatools/`, `pvsystemprofiler/`, `anomalydetector/`). Test fixture data is in `tests/fixtures/`.

**IMPORTANT — do NOT run `pytest tests/` or `python -m unittest discover -s tests`**. The `tests/solardatatools/` directory shadows the real `solardatatools/` package, causing `from solardatatools import DataHandler` to import from the empty `tests/solardatatools/__init__.py` instead of the actual package. Always run discovery from the repo root without `-s tests`.

## Key Architecture

### `DataHandler` (solardatatools/data_handler.py)
The primary user-facing class. Accepts a pandas DataFrame with a datetime index. `run_pipeline(power_col=...)` executes the full signal processing pipeline: standardize time axis, make 2D matrix, compute data quality scores, detect clear days, detect clipping, detect capacity changes. Uses `sig-decomp` (Signal Decomposition) with CLARABEL solver by default; MOSEK via CVXPY as optional alternative.

### `solardatatools/algorithms/`
Pipeline stages implemented as classes: `CapacityChange`, `TimeShift`, `ClippingDetection`, `ShadeAnalysis`, `SoilingAnalysis`, `Dilation`, `LossFactorAnalysis`, `PVQuantiles`, `ClearSkyDetection`, `SunriseSunset`. Instantiated and called by `DataHandler`.

### `sdt_dask` Three-Component Architecture
1. **ClientPlug** (`sdt_dask/clients/`) — Sets up Dask clusters. Inherit from `ClientPlug`, implement `init_client()`. Options: `LocalClient`, `FargateClient`, `AzureClient`
2. **DataPlug** (`sdt_dask/dataplugs/`) — Retrieves and cleans data. Inherit from `DataPlug`, implement `get_data(keys: tuple) -> pd.DataFrame`. Options: `LocalFiles`, `S3Bucket`. DataPlugs run inside Dask workers — avoid non-serializable objects (create S3 clients inside `get_data`, not in `__init__`)
3. **Runner** (`sdt_dask/dask_tool/runner.py`) — Orchestrates: receives a Dask Client, DataPlug, and list of keys. `set_up(keys, dataplug, **kwargs)` schedules pipeline tasks. `get_result()` collects and saves outputs

### `pvsystemprofiler`
Standalone parameter estimation: `tilt_azimuth_study.py`, `latitude_study.py`, `longitude_study.py`. Uses `estimator.py` and `ground_truth_estimator.py` with algorithms in `algorithms/` and utilities in `utilities/`.

## Important Notes

- Version is managed via `setuptools_scm` — `_version.py` is auto-generated from git tags
- Python 3.10–3.13 supported
- `tests/fixtures/` contains archival Jupyter notebooks (from deprecated `dataio` module) — do not attempt to run
- Docs are built with Sphinx + pydata-sphinx-theme + nbsphinx. Source in `docs/source/`, built to `docs/_build/html/`
- Docker images for cloud Dask deployment are documented in `docker/README.md`. Pre-built image: `slacgismo/sdt-v1:latest`
