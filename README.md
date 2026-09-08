# CoolLink SG: CPTL research archive

Code, configurations and processed data supporting **Behavior-calibrated, time-dependent pedestrian routing reveals district-dependent opportunities to reduce modeled heat exposure**.

Authors, in order: Xun Zhang; Kexin Song; Hongwei Zhang; Xidong Wang; Wenlong Yu.
Repository: https://github.com/1realikun1/coollink-sg-cptl

Version: 1.0.0 (2026-09-08). This is a research artifact, not a pedestrian navigation service.

## Download this version

This repository distributes the complete code tree as `CPTL_code_v1.0.0.zip`. Download that file and the `CPTL_processed_data_v1.0.0.zip` release asset, then extract **both into the same working directory**. The folders listed below refer to the extracted code archive. The code ZIP contains the Python modules, YAML configurations, tests, citation metadata and verification reports; the data ZIP contains the processed inputs and full result tables. Check both downloads against `SHA256SUMS.txt`.

## Contents and analysis boundaries

| Location | Purpose |
|---|---|
| `src/`, `config/`, `tests/` | Primary CPTL implementation; 152 synthetic OD pairs, five departures, 760 weighted cases and 12,122 candidate routes |
| `coverage_reference/` | Separate historical control implementation, including behavior calibration, multistation forcing, fixed caps and temporal thinning; do not merge its routing module with the primary module |
| `scripts/verify_results.py` | Read-only integrity and numerical reproduction checks for the archived primary recommendations |
| `DATA_DICTIONARY.md` | Dataset keys, units, provenance and interpretation |
| `provenance/` | Checksums of source snapshots and files actually deposited |
| Release attachment `CPTL_processed_data_v1.0.0.zip` | Processed data, routing inputs, full candidate/segment tables, machine-readable supplementary tables and additional Figure 7 route examples |

The separately developed RS-CPTL/RS-PTL satellite models are outside this archive.
Earlier shade-proxy/preparation utilities remain as upstream dependencies and historical context; they are not alternative evidence for the reported CPTL results.

## Reproduce summary results

Use Python 3.11 or 3.12. Download the processed-data release attachment and extract its contents into the repository root, so that `data/processed/` and `outputs/tables/` exist here.

```bash
python -m venv .venv
# Activate the environment using the command appropriate to your operating system.
python -m pip install -e ".[dev]"
python scripts/verify_results.py --data-root .
python -m pytest
```

The verification script uses pandas and NumPy, checks the fixed candidate pool and eligibility rule, and reproduces the district summaries without rerunning SOLWEIG.
The package dependencies are declared in `pyproject.toml`; `environment-tested.txt` records the versions used for archive verification. A new dependency resolution can differ from that environment.

## Recompute routing from archived inputs

The processed-data attachment includes the four road networks, 37-time-step thermal tables, synthetic OD sample and 500 behavior bootstrap estimates needed by this command:

```bash
python -m coollink_sg.routing.behavior_probability_routing --config config/behavior_informed_probability_routing.yaml --output-root work/rerun
```

This reruns the primary routing search and can be expensive. It was not rerun during packaging. Outputs should be directed to a new folder to preserve the deposited results.
To run historical controls, use a separate environment/package root under `coverage_reference/`; do not import both implementations into the same Python process. The temporal-thinning implementation is `coverage_reference/src/coollink_sg/heat/temporal_thinning.py`, with its configuration in `coverage_reference/config/temporal_thinning_audit.yaml`. It evaluates a fixed candidate pool and does not measure wall-clock speedup.

## Upstream data and limitations

Raw OSM snapshots, national raster products, and individual-level records from the external choice experiment are not duplicated. Their source URLs and licenses are recorded in `DATA_DICTIONARY.md` and `THIRD_PARTY_NOTICES.md`. Reconstructing SOLWEIG fields from scratch requires those upstream products and the corresponding preparation configurations. Processed thermal and routing inputs are included so route analysis does not depend on re-downloading mutable APIs.

The 760 cases represent behavior-informed synthetic demand, not observed trips. The 240-case network-coverage controls, the fixed 5% distance-cap controls, and the additional Jurong East Figure 7 illustration are distinct panels. The additional illustration is excluded from aggregate statistics. UTCI, CPTL and behavioral adoption are modeled quantities, not in-study field validation.

## Citation and rights

Use `CITATION.cff` for the five confirmed authors and this artifact version. A manuscript DOI has not been assigned. Repository URLs and any archival DOI must be cited only after the corresponding records exist.
Code licensing was not assigned in the source project. Public access does not by itself grant a new blanket software license. See `RIGHTS.md` and the third-party notices; original data licenses continue to apply.

## Archive verification adjustments

Scientific Python modules are copied without changing their algorithms. The pytest temporary directory was changed to a portable root-relative location. Two small domain registry GeoPackages are included as integration fixtures. One upstream CHMv2 acquisition test is explicitly skipped when the original global raw tile index is absent; all other included tests remain active. See `provenance/test_results.md` for the actual run outcomes.
