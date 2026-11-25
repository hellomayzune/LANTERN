<p align="left">
• <a href="https://github.com/hellomayzune"><strong>GitHub</strong></a> •
<a href="https://orcid.org/0000-0003-0282-2633"><strong>ORCID</strong></a> •
<a href="https://scholar.google.com/citations?user=LmP8B_4AAAAJ&hl=en"><strong>Google Scholar</strong></a> •
<a href="https://www.researchgate.net/profile/May-Zune"><strong>ResearchGate</strong></a> •
<a href="https://www.linkedin.com/in/mayzune/"><strong>LinkedIn</strong></a> •
<a href="https://mayzune.com/"><strong>Blog</strong></a> •

</p>

> ℹ️ LANTERN is a workflow for planning heat-pump rollout within the capacity of local electricity networks. It estimates how many gas-heated homes each primary substation can connect within its deployable headroom, the carbon cost of homes waiting for reinforcement, how conversions could be sequenced to minimise whole-life emissions, and which places and households are left on gas the longest. In Sheffield and Greater Nottingham, network limits mainly shaped where homes remained on gas, while the carbon objective mainly shaped who did.

# 🏮 LANTERN: Local Allocation of Network headroom for Thermal Electrification and Retrofit in Neighbourhoods

This repository contains the Jupyter notebooks, shared Python source code, configuration, input data documentation and shareable outputs for **LANTERN**, a modelling framework for urban heat electrification under distribution-network constraints. LANTERN combines a housing-stock model calibrated against measured gas use, published primary-substation headroom adjusted for the limits of the substations upstream, empirical heat-pump coincidence, year-by-year rollout scenarios and linear programmes that sequence conversions. Distributional and spatial statistics then describe who and where remains on gas under each strategy, and a counterfactual design separates the effect of network capacity from the effect of the order in which homes are converted.

The analysis covers every gas-heated home served by the study-area primary substations of Sheffield (Northern Powergrid, with one NGED primary) and Greater Nottingham (NGED) from 2025 to 2050. Study areas are defined by substation supply area rather than council boundary. All results are modelled, aggregated to primary substation, archetype, lower-layer super output area (LSOA) or city, and describe capacity potential at primary-substation level, not confirmed household connections.

- **Project**: LANTERN — Local Allocation of Network headroom for Thermal Electrification and Retrofit in Neighbourhoods
- **Authors**: [May Zune](https://orcid.org/0000-0003-0282-2633)
- **Notebook author**: [May Zune](https://orcid.org/0000-0003-0282-2633)
- **Funding**: Partly supported by the Research England Development Fund through the South Yorkshire Sustainability Centre (University of Sheffield) project *Development of a regional retrofit optimisation platform*.
- **Associated publication**: *Two geographies of waiting: how electricity-network constraints and carbon-optimal sequencing shape urban heat electrification* — Manuscript submitted for publication. Full bibliographic details and the associated Zenodo DOI will be added when available.

📬 **Contact**: Please open an issue or use the links above.

🆚 **Version**: v1

©️ **Licence**: Code licensed under the *MIT Licence*; authors' derived, aggregated outputs (tables and figures in `outputs/`) under *CC BY 4.0*. Raw input data, building-level and household-level intermediate files, and licensed third-party data are not redistributed (see [Excluded from Repository](#excluded-from-repository-non-distributable)).

📖 Contents

- [Research Questions](#-research-questions)
- [Repository Information](#️-repository-information)
- [Pipeline Notebooks](#️-pipeline-notebooks)
- [Software Requirements & Setup](#-software-requirements--setup)
- [Reproducibility & Execution Steps](#-reproducibility--execution-steps)
- [Project Documentation](#-project-documentation)
- [Manuscript and Supplementary Display Items](#-manuscript-and-supplementary-display-items)
- [Data Availability & Repository Scope](#-data-availability--repository-scope)

---

# ❓ Research Questions

| | Question | Notebooks |
|---|---|---|
| **RQ1** | To be updated upon publication | 02, 04a |
| **RQ2** | To be updated upon publication | 05, 06, 08 |
| **RQ3** | To be updated upon publication | 06, 08 |
| **RQ4** | To be updated upon publication | 06b, 08 |

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 🗂️ Repository Information

- **Contents**: Jupyter notebooks (`.ipynb`), Python modules (`.py`), study configuration (`.yaml`, `.csv`) and generated artefacts (tables and figures: `.csv`, `.png`).
- **Programming Language**: `Python` (`pandas`, `numpy`, `scipy`, `geopandas`, `shapely`, `pyproj`, `pyogrio`, `matplotlib`, `highspy`, `pulp`, `SALib`, `pyarrow`, `openpyxl`, `odfpy`, `pyyaml`).
- **Methodological Scope**: Archetype heating profiles come from building-energy simulations carried out before this study; LANTERN transfers them to local weather, calibrates them against measured gas use and combines them with network data, field-trial heat-pump data and optimisation. No machine learning is used.
- **Repository Metadata**: Created 28 October 2026.

📌 **Repository Map**

```text
├── README.md                                    # Project overview (this file)
├── DATASHEET.md                                 # Dataset documentation (Gebru et al., 2021)
├── ADR.md                                       # Architecture decision records
├── MODEL_CARD.md                                # Model cards (Mitchell et al., 2019)
├── environment.yml                              # Conda environment of the reported run
├── config/
│   ├── study.yaml                               # Every study setting, assumption and sensitivity (single source of truth)
│   ├── data_manifest.csv                        # Expected raw files, their folders and licences
│   └── paths.example.yaml                       # Template for the local data path (copy to paths.yaml, not committed)
├── src/lantern/                                 # Shared Python modules imported across notebooks
│   ├── config.py                                # Loads study.yaml and paths.yaml; locates data files
│   ├── stock.py                                 # EPC and Verisk readers, archetype classification, Census reconciliation
│   ├── spatial.py, areas.py                     # Supply-area assignment, output areas, LSOAs and urban filter
│   ├── network.py                               # Operator tables, primary and deployable headroom, upstream limits, planned works
│   ├── carbon.py                                # CCC benchmark, emission factors, embodied and refrigerant carbon
│   ├── demand.py                                # Weather transfer, design-day profiles, heat-pump COP, hot water
│   ├── calibration.py                           # LSOA gas readers, non-negative calibration, holdout validation
│   ├── capacity.py                              # Coincidence factors, Monte Carlo homes within headroom, binding year
│   ├── rollout.py                               # Rollout scenarios Ra–Rd, PV self-consumption, refrigerant cohorts
│   ├── sequencing.py                            # Linear programmes, rule-based orderings, rounding, shadow prices, waiting times
│   ├── equity.py                                # Deprivation and fuel-poverty readers, group outcomes, concentration index
│   ├── spatial_stats.py                         # Spatial weights, global, local and bivariate Moran's I, Holm and FDR adjustment
│   ├── screening.py                             # Morris design, elementary effects and bootstrap intervals
│   ├── maps.py                                  # Shared map styling: classes, rural dots, hatching, call-outs
│   ├── checks.py                                # PASS / WARN / FAIL / INFO checks written by every notebook
│   └── style.py                                 # Shared figure style at A4 text width
├── notebooks/                                   # One notebook per stage, run in order (00 to 08)
├── tools/
│   ├── build_notebooks.py                       # Builds the notebooks from one script, so every notebook shares one template
│   └── crop_study_areas.py                      # Candidate primaries and crop polygons for data extraction
├── docs/
│   ├── algorithms.md                            # Every rule-based, optimisation and statistical method, with its ADR and module
│   ├── decisions.md                             # Working log of decisions, newest first (ADR.md is the curated record)
│   ├── data_provenance.md                       # One row per raw file: source, version, licence, resolution, notebooks
│   └── frozen_run_20261117.csv                  # Checksums of configuration, code, notebooks, tables and figures of the run of record
└── outputs/
    ├── figures/                                 # Shareable figures (.png); maps based on licensed polygons are not included
    └── tables/                                  # Shareable derived tables (.csv), aggregated to primary, archetype, LSOA or city
```

Data are **not** in this repository. They live in a separate local folder, `LANTERN_data/`, whose structure is documented in `DATASHEET.md` and `config/data_manifest.csv`.

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# ⚙️ Pipeline Notebooks

The notebooks run in sequence and share code from `src/lantern/`. Each has one settings cell at the top, a plain-English note above every step, and a checks cell at the end that reports PASS, WARN, FAIL or INFO for each check. Notebooks 01a–01c build the housing stock; 02 the network; 03a–03c demand, carbon and calibration; 04a–04b capacity and rollout; 05 sequencing; 06 and 06b who and where remains on gas; 07 the screening of uncertain inputs; and 08 the spatial analysis and counterfactual decomposition.

### 📊 Notebook 00 — Setup
| Purpose | Key inputs | Produces |
|---|---|---|
| Check the Python environment, the local data path and every file listed in the data manifest; draw the study areas. | `config/paths.yaml`, `config/data_manifest.csv` | Environment and data checks |

### 📊 Notebooks 01a–01c — Housing stock
*See `DATASHEET.md` Datasets 3–4, `MODEL_CARD.md` Model 1, `ADR-009`–`ADR-014`.*

| Notebook | Purpose | Key inputs | Produces |
|---|---|---|---|
| `01a_stock_clean_locate` | Combine and clean EPC certificates; locate EPC and Verisk records with OS Open UPRN; assign buildings and output areas to primary substation supply areas. | EPC, Verisk UKBuildings, OS Open UPRN, supply-area polygons | Located stock (local only); `01a_counts_per_primary.csv` |
| `01b_stock_classify_filter` | Count dwellings in Verisk (residential and mixed-use); calibrate Verisk floor area to EPC; classify 14 house archetypes and flats; fill primaries where Verisk misses homes from EPC; reconcile with Census 2021; apply the urban inclusion rule. | 01a outputs, Census 2021 TS044, rural-urban classification | `01b_primaries.csv`, `01b_archetype_counts.csv`, `01b_dwellings_by_class.csv` and agreement tables |
| `01c_stock_profile` | Reconcile dwelling type, tenure and central heating with Census 2021; construction age bands and EPC coverage by primary. | 01b outputs, Census 2021 TS044, TS046, TS054 | `01c_*` tables and figures |

### 📊 Notebook 02 — Network headroom
*See `DATASHEET.md` Dataset 2, `MODEL_CARD.md` Model 3, `ADR-015`–`ADR-017`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Firm capacity, maximum demand, accepted connections and headroom per primary; deployable headroom after the limits of supply points, grid supply points and bulk supply points; register with source and confidence; cross-checks against operator tables and a GB research dataset; planned works. | Northern Powergrid LTDS and heat-map table; NGED Network Opportunity Map and LTDS | `02_headroom.csv`, `02_upstream_limits.csv`, `02_network_register.csv`, cross-check tables, `02_upstream_limits.png` |

### 📊 Notebooks 03a–03c — Carbon benchmark, demand and calibration
*See `DATASHEET.md` Datasets 5–7, `MODEL_CARD.md` Model 2, `ADR-018`–`ADR-024`.*

| Notebook | Purpose | Key inputs | Produces |
|---|---|---|---|
| `03a_carbon_benchmark` | Localise the CCC Seventh Carbon Budget pathway for heat in existing homes to the study primaries by dwelling share, with an emission-intensity sensitivity. | CCC full dataset and corrections log, UK dwelling stock, DESNZ local-authority emissions | `03a_benchmark_*.csv`, `03a_benchmark_by_year.png` |
| `03b_demand_carbon` | Annual space heating and hot water, design-day peaks with temperature-dependent COP, operational, embodied and refrigerant carbon, per archetype and pathway. | Archetype heating profiles, TMYx weather, SAP 10.2, Green Book and DESNZ factors, NESO FES, PVGIS | `03b_*` tables, `03b_design_peak_by_archetype.png` |
| `03c_calibration` | Share of each archetype heated by mains gas and the gas-heated pool; calibrate space heating by built-form group against DESNZ LSOA gas; holdout validation; calibration error against deprivation and fuel poverty; hot water at Census occupancy; NEED cross-check; calibrated energy and peaks. | 03b outputs, EPC, DESNZ LSOA gas, Census TS017, NEED, IMD 2025, fuel poverty | `03c_gas_pool_by_archetype.csv`, `03c_calibration_factors.csv`, `03c_holdout_validation.csv`, `03c_error_by_deprivation.csv`, `03c_peaks_calibrated.csv` and figures |

### 📊 Notebooks 04a–04b — Capacity and rollout
*See `DATASHEET.md` Dataset 8, `MODEL_CARD.md` Models 3–4, `ADR-025`–`ADR-030`.*

| Notebook | Purpose | Key inputs | Produces |
|---|---|---|---|
| `04a_capacity` | Coincidence factors from field-trial data; existing-demand shapes from primary transformer flows; homes within deployable headroom by Monte Carlo; binding year; stress tests and ordering sensitivities. | 02 and 03c outputs, Electrification of Heat, RHPP, NGED primary flows, HadCET, CCC rollout | `04a_homes_within_headroom.csv`, `04a_coincidence_cases.csv`, `04a_binding_year.csv`, `04a_summary.csv` and figures |
| `04b_rollout` | Year-by-year rollout scenarios (Ra unconstrained, Rb wait for reinforcement, Rc whole-house for all, Rd targeted whole-house); carbon cost of waiting; break-even delay; carbon value of added headroom; CCC alignment; observed deployment. | 04a outputs, 03b–03c carbon, PVGIS hourly, Elexon profiles, DESNZ deployment statistics | `04b_carbon_cost_of_constraints.csv`, `04b_scenarios_*.csv`, `04b_headroom_value.csv` and figures |

### 📊 Notebook 05 — Sequencing
*See `MODEL_CARD.md` Model 5, `ADR-031`–`ADR-034`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Linear programmes that schedule conversions by primary, archetype, pathway and year (lowest whole-life emissions; lowest benchmark overshoot); rule-based orderings; every schedule checked against non-linear capacity; rounding to whole homes; shadow prices; waiting times; annual emissions and remaining benchmark; archetype share of conversions. | 03c–04b outputs | `05_strategy_summary.csv`, `05_archetype_share_of_conversions.csv`, `05_schedule_check.csv`, `05_shadow_prices_*.csv`, `05_waiting_*.csv`, `05_strategies.png` and figures |

### 📊 Notebooks 06 and 06b — Who and where remains on gas
*See `DATASHEET.md` Dataset 9, `MODEL_CARD.md` Model 6, `ADR-035`–`ADR-039`.*

| Notebook | Purpose | Key inputs | Produces |
|---|---|---|---|
| `06_equity` | Share still on gas and waiting time by dwelling type, deprivation quintile, fuel poverty and tenure; concentration index across LSOAs; carbon cost of parity floors; the parity-constrained allocation (O4+). | 05 schedules, IMD 2025, fuel poverty, Census TS054 | `06_group_outcomes.csv`, `06_distribution.csv`, `06_fairness_cost.csv` and figures |
| `06b_maps` | LSOA maps of gas use, fuel poverty, deprivation, their bivariate classes, heat-demand density and homes still on gas by strategy; dense-and-late LSOAs; network maps (local only). | 06 outputs, LSOA and output-area boundaries, DESNZ LSOA gas | `06b_lsoa_indicators.csv`, `06b_dense_and_late.csv`, LSOA map figures |

### 📊 Notebook 07 — Screening of uncertain inputs
*See `MODEL_CARD.md` Model 7, `ADR-040`–`ADR-042`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Morris elementary-effects screening of ten inputs on seven outputs (220 runs per city), with bootstrap intervals and ranking stability; base-run reproduction; linear headroom at low, empirical and high coincidence; heat-pump efficiency against field evidence; cold-spell case. | 03c–06 outputs, field-trial efficiency evidence | `07_morris_indices.csv`, `07_ranking_*.csv`, `07_cold_spell.csv`, `07_mu_star_heatmap.png`, `07_mu_star_sigma.png` |

### 📊 Notebook 08 — Spatial analysis and counterfactual decomposition
*See `MODEL_CARD.md` Model 6, `ADR-043`–`ADR-047`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Each strategy (S0, O1, O1+, O4+) with and without the headroom limit; network, allocation and interaction components and the whole-house pathway effect; global, local and bivariate Moran's I with alternative weights; headroom and deprivation by primary; houses-only check; flat-parity scenarios; contiguous dense-and-late zones. | 05–06b outputs | `08_decomposition.csv`, `08_pathway_effect.csv`, `08_moran_global.csv`, `08_local_clusters_summary.csv`, `08_primary_table.csv`, `08_flat_parity.csv` and figures |

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 💻 Software Requirements & Setup

### **Environment Specifications**
- **Python Version**: Python 3.11 (`environment.yml`).
- **Interface**: JupyterLab, with the kernel **Python (lantern)**.
- **Key Dependencies** (as listed in `environment.yml`): `pandas` (≥ 2.1), `numpy`, `scipy` (≥ 1.9), `geopandas` (≥ 1.0), `shapely` (≥ 2.0), `pyproj`, `pyogrio`, `pyarrow`, `pyyaml`, `openpyxl`, `odfpy`, `matplotlib`, `highspy`, `pulp`, `salib`, `jupyterlab`, `ipykernel`.
- **Runtime Tracking**: Notebook 00 reports the versions of the packages in use; checksums of the run of record are kept in `docs/frozen_run_20261117.csv`.

### **Installation & Setup**
1. Install [Miniforge](https://github.com/conda-forge/miniforge) (or Anaconda).
2. Open *Miniforge Prompt* in this folder and create the environment:
   ```bash
   conda env create -f environment.yml
   conda activate lantern
   python -m ipykernel install --user --name lantern --display-name "Python (lantern)"
   ```
3. Copy `config/paths.example.yaml` to `config/paths.yaml` and set `data_root` to the local `LANTERN_data` folder (forward slashes, e.g. `D:/LANTERN_data`). Keep `LANTERN_data` outside OneDrive or any other synced folder.
4. Start JupyterLab from the `lantern` environment each time:
   ```bash
   conda activate lantern
   jupyter lab
   ```
   On Windows, starting JupyterLab from `base` can make the kernel load libraries from `base`, which stops matplotlib without an error message.
5. Open `notebooks/00_setup.ipynb`, check the kernel shown top right is **Python (lantern)**, and run all cells. Every check should show PASS before moving on.

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 🔁 Reproducibility & Execution Steps

### **Data Placement & Architecture Rules**
* Place raw files in `LANTERN_data/raw/` in the folders listed in `config/data_manifest.csv`. Notebook 00 checks that every required file is present.
* **Central Configuration**: All settings, assumptions, sensitivities and seeds are in `config/study.yaml` (`ADR-003`). The only per-machine setting is the data path in `config/paths.yaml`.
* **Immutability**: Raw files are never modified (`ADR-002`). Notebooks read from `raw/` and earlier `processed/` or `interim/` folders, and write to their own `processed/` folder, with shareable tables and figures copied to `outputs/`.
* **Deterministic Execution**: Every stochastic step uses a seed set in `config/study.yaml` (Monte Carlo ordering, bootstrap resampling, holdout splits, the Morris design and spatial permutations). Reported values are read from saved tables (`ADR-007`).
* **Notebook source**: The notebooks are generated by `tools/build_notebooks.py` from one script, so that every notebook shares the same settings cell, checks and style. Edits to a notebook are carried back into the script.

### **Sequential Execution Order**
Run the notebooks in sequence from a fresh kernel:

1. **`00_setup`**
2. **`01a_stock_clean_locate`** → **`01b_stock_classify_filter`** → **`01c_stock_profile`**
3. **`02_network`**
4. **`03a_carbon_benchmark`** → **`03b_demand_carbon`** → **`03c_calibration`**
5. **`04a_capacity`** → **`04b_rollout`**
6. **`05_sequencing`** *(saves the schedules used by 06, 07 and 08)*
7. **`06_equity`** → **`06b_maps`**
8. **`07_screening`** *(the longest run; completed runs are saved and resumed)*
9. **`08_spatial`**

A change to configuration, code or input data requires re-running the affected notebook and every notebook after it (`ADR-005`).

### **Re-running from Public Files**
Because several inputs are licensed for local use only and building-level intermediate files are withheld (`ADR-008`), the notebooks cannot be re-run from the repository alone. The derived tables and figures of the reported run are provided in `outputs/`. The notebooks run unchanged on a local `LANTERN_data` folder that follows the structure in `DATASHEET.md` and `config/data_manifest.csv`.

*Tip: Before a clean replication, clear `outputs/` so that files from earlier runs are not mistaken for newly generated outputs.*

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 📖 Project Documentation

| Document | Framework / Standard | Scope & Coverage |
| :--- | :--- | :--- |
| [`DATASHEET.md`](./DATASHEET.md) | Gebru et al. (2021) | Motivation, composition, collection, preprocessing, uses, distribution limits and maintenance of the input datasets (boundaries, network, housing stock, Census, energy use, carbon, weather, field-trial heat-pump data, deprivation and fuel poverty) and the derived data products. |
| [`ADR.md`](./ADR.md) | Lightweight MADR | Design decisions across the pipeline, each with context, decision, alternatives considered and consequences. |
| [`MODEL_CARD.md`](./MODEL_CARD.md) | Mitchell et al. (2019) | Intended use, factors, metrics, data and limitations of seven components: (1) housing stock and archetypes, (2) demand, carbon and calibration, (3) network headroom and homes within headroom, (4) rollout scenarios, (5) sequencing strategies, (6) distributional and spatial measures with the counterfactual decomposition, and (7) Morris screening. |
| [`docs/algorithms.md`](./docs/algorithms.md) | — | Every rule-based, optimisation and statistical method in the pipeline, with its decision record and the module and function that implement it. LANTERN uses no machine learning. |
| [`docs/data_provenance.md`](./docs/data_provenance.md) | — | One row per raw file: source, version, download date, licence, resolution, measured or modelled, and notebooks that use it. |
| [`docs/decisions.md`](./docs/decisions.md) | — | Dated working log of decisions, newest first; the source of the records in `ADR.md`. Not published. |

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 🧾 Manuscript and Supplementary Display Items

**Article**

| Item | File | Notebook |
|---|---|---|
| Figure 1 | Prepared outside the notebooks (modelling framework) | — |
| Figure 2 | `figures/02_upstream_limits.png` | 02 |
| Figure 3 | `figures/08_clusters.png` | 08 |
| Figure 4 | `figures/05_strategies.png` | 05 |
| Figure 5 | `figures/08_network_vs_ordering.png` | 08 |
| Figure 6 | `figures/06_carbon_cost_of_fairness.png` | 06 |
| Figure 7 | `figures/06b_heat_density.png` | 06b |
| Figure 8 | `figures/07_mu_star_heatmap.png` | 07 |
| Table 1 | `tables/01b_primaries.csv`, `tables/02_headroom.csv`, `tables/04a_summary.csv`, `tables/04a_binding_year.csv` | 01b, 02, 04a |
| Table 2 | `tables/05_strategy_summary.csv` | 05 |
| Table 3 | `tables/08_decomposition.csv`, `tables/08_pathway_effect.csv` | 08 |

**Supplementary tables** (`tables/`)

| Item | File | Notebook |
|---|---|---|
| S1 | `03c_gas_pool_by_archetype.csv` (archetype descriptions from the earlier archetype studies) | 03c |
| S2 | `02_upstream_limits.csv` | 02 |
| S3 | `03c_calibration_factors.csv`, `03c_holdout_validation.csv` | 03c |
| S4 | `06_fairness_cost.csv` | 06 |
| S5 | `08_flat_parity.csv` | 08 |
| S6 | `05_archetype_share_of_conversions.csv` | 05 |
| S7 | `08_local_clusters_summary.csv` | 08 |
| S8 | `08_moran_global.csv` | 08 |
| S9 | `08_primary_table.csv` | 08 |
| S10 | Screening inputs and ranges in `config/study.yaml` (`screening`); design in `07_design.csv` | 07 |
| S11 | `07_cold_spell.csv` | 07 |

**Supplementary figures** (`figures/`)

| Item | File | Notebook |
|---|---|---|
| S1 | `03c_lsoa_observed_vs_modelled.png` | 03c |
| S2 | `04a_coincidence.png` | 04a |
| S3 | `04b_cost_of_constraints.png` | 04b |
| S4 | `05_remaining_budget.png` | 05 |
| S5 | `05_annual_emissions.png` | 05 |
| S6 | `06_still_on_gas_over_time.png` | 06 |
| S7 | `06_wait_by_group.png` | 06 |
| S8 | `06b_context_maps.png` | 06b |
| S9 | `06b_still_on_gas_by_strategy.png` | 06b |
| S10 | `06b_o4_minus_o1.png` | 06b |
| S11 | `08_primary_deprivation.png` | 08 |
| S12 | `07_mu_star_sigma.png` | 07 |

**Supporting data files.** `outputs/tables/` also holds files that underlie the numbered items or the checks but are not reproduced in the Supplementary document, including the checks of every notebook (`*_checks.csv`), `03c_gas_share_by_archetype.csv`, `03c_error_by_deprivation.csv`, `04a_coincidence_cases.csv`, `05_conversions_by_archetype.csv`, `05_schedule_check.csv`, `05_nonlinear_correction.csv`, `05_shadow_prices_headroom.csv`, `06_group_outcomes.csv`, `06b_lsoa_indicators.csv`, `07_morris_indices.csv`, `07_ranking_stability.csv`, `08_local_clusters_lsoa.csv` and `08_dense_late_zones.csv`.

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 📦 Data Availability & Repository Scope

### **Included in Repository**
* **Source Code & Documentation**: Notebooks, Python modules in `src/lantern/`, configuration in `config/` (except the local `paths.yaml`), the notebook builder and the documentation (licensed under the **MIT Licence**).
* **Generated Artefacts**: Derived tables and figures in `outputs/`, aggregated to primary substation, archetype, LSOA or city, are distributed under **CC BY 4.0**.

### **Excluded from Repository (Non-Distributable)**
* **Licensed building data** (Verisk UKBuildings): licensed for local use only. Building-level extracts and every intermediate file derived from them at building or address level are withheld.
* **NGED primary supply-area polygons**: shared under a licence for local use only. Maps drawn on them are saved under `processed/06_maps/local_only/` and are not distributed.
* **Intermediate files** (`LANTERN_data/processed/`, `LANTERN_data/interim/`): located and classified stock at building and address level, schedules by unit and run files of the screening; withheld because they contain or can reconstruct licensed or address-level data (`ADR-008`).
* **Open and third-party inputs** (EPC, Census, ONS boundaries, DESNZ statistics, operator open data, CCC, NESO, field-trial data from the UK Data Service, weather files and others): not redistributed; each source, version and licence is listed in `docs/data_provenance.md` so that they can be obtained from the publisher.
* **Own data from earlier studies** (archetype heating profiles, operational and embodied carbon by archetype): not distributed with this release; available from the author on reasonable request.

[Back to top](#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)
