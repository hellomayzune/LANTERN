<p align="left">
• <a href="https://github.com/hellomayzune"><strong>GitHub</strong></a> •
<a href="https://orcid.org/0000-0003-0282-2633"><strong>ORCID</strong></a> •
<a href="https://scholar.google.com/citations?user=LmP8B_4AAAAJ&hl=en"><strong>Google Scholar</strong></a> •
<a href="https://www.researchgate.net/profile/May-Zune"><strong>ResearchGate</strong></a> •
<a href="https://www.linkedin.com/in/mayzune/"><strong>LinkedIn</strong></a> •
<a href="https://mayzune.com/"><strong>Blog</strong></a> •

</p>

<p align="left">
📄 Companion to <a href="./README.md"><strong>README.md</strong></a> •
<a href="./DATASHEET.md"><strong>DATASHEET.md</strong></a> •
<a href="./ADR.md"><strong>ADR.md</strong></a>
</p>

# 🧠 LANTERN — MODEL CARD

This document presents model cards for the seven analytical components of the **LANTERN** (Local Allocation of Network headroom for Thermal Electrification and Retrofit in Neighbourhoods) notebooks, following the reporting principles proposed by **Mitchell et al. (2019)** in *Model Cards for Model Reporting*. LANTERN contains no machine-learning models; its components are rule-based classifications, calibrated physical models, Monte Carlo estimates, linear programmes and statistical measures. Rule-based components are documented alongside fitted ones because downstream results depend on both.

Each model card documents the component's specifications, intended uses, relevant factors, evaluation metrics, data and limitations. In accordance with `ADR-007`, numerical results are not reproduced in this document; they are reported in the files listed under *Quantitative analyses*, all in `outputs/tables/` unless stated otherwise.

📖 Contents

- [Model 1 — Housing stock and archetypes](#-model-1--housing-stock-and-archetypes)
- [Model 2 — Demand, carbon and calibration](#-model-2--demand-carbon-and-calibration)
- [Model 3 — Network headroom and homes within headroom](#-model-3--network-headroom-and-homes-within-headroom)
- [Model 4 — Rollout scenarios](#-model-4--rollout-scenarios)
- [Model 5 — Sequencing strategies](#-model-5--sequencing-strategies)
- [Model 6 — Distributional and spatial measures](#-model-6--distributional-and-spatial-measures)
- [Model 7 — Morris screening](#-model-7--morris-screening)

[Back to README](./README.md#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 🏠 Model 1 — Housing stock and archetypes

### Model details
- **Notebooks:** `01a_stock_clean_locate`, `01b_stock_classify_filter`, `01c_stock_profile`; functions in `src/lantern/stock.py`, `spatial.py` and `areas.py`.
- **Type:** deterministic rules; one calibrated ratio (Verisk to EPC floor area by built form).
- **Rules:**
  - **Dwellings:** Verisk buildings with at least one address, Residential or Mixed Use; Mixed Use addresses counted as flats (`ADR-009`).
  - **Archetypes:** 14 house archetypes by built form and floor area, plus flats; Verisk gross area calibrated to EPC floor area before classification; unspecified terraces resolved from matched EPC records or local ratios (`ADR-010`).
  - **Multi-address houses:** at least 30 m² per address → houses of the building's form; below → flats (`ADR-011`).
  - **EPC fill:** primaries with Verisk dwellings below 60% of Census households are built from EPC, scaled to Census (`ADR-012`).
  - **Gas-heated pool:** dwellings × EPC share of each archetype heated by mains gas, by city (computed in `03c_calibration`).
  - **Study area:** primaries with at least 50% of their dwellings in urban output areas of the core councils (`ADR-001`, `ADR-013`).

### Intended use
- **Uses:** counting gas-heated homes by primary and archetype as the rollout pool; describing the stock of each primary.
- **Users:** the study authors; researchers building area-level stock models from open and licensed data.
- **Out of scope:** building-level predictions; homes off the gas grid; new-build homes.

### Factors
City, operator, primary, archetype, data source (Verisk or EPC fill), inclusion threshold.

### Metrics
Agreement of flats and houses with Census 2021 by primary; agreement of archetypes between EPC and Verisk for matched addresses; EPC coverage and certificate age; Census reconciliation of type, tenure and heating.

### Data
Datasets 1, 3 and 4 (`DATASHEET.md`).

### Quantitative analyses
`01a_counts_per_primary.csv`, `01b_primaries.csv`, `01b_archetype_counts.csv`, `01b_dwellings_by_class.csv`, `01b_archetype_agreement.csv`, `01b_archetype_agreement_matrix.csv`, `01b_area_calibration.csv`, `01b_flat_threshold.csv`, `01b_residential_without_address.csv`, `01c_census_types_by_lsoa.csv`, `01c_tenure_heating_by_primary.csv`, `01c_age_bands_by_primary.csv`, `01c_epc_coverage_by_primary.csv`, `03c_gas_share_by_archetype.csv`, `03c_gas_pool_by_archetype.csv`.

### Ethical considerations
Building- and address-level records are processed locally and aggregated before release; no output identifies a dwelling.

### Caveats and recommendations
- Mixed Use addresses include some non-domestic units; check the sensitivity case without them.
- Flats are not simulated and use a scaled profile (Model 2).
- EPC and Verisk construction age bands differ; one archetype's age cut differs between sources.
- Primaries filled from EPC are listed in the checks and carry more uncertainty.

[Back to top](#-lantern--model-card)

---

# 🔥 Model 2 — Demand, carbon and calibration

### Model details
- **Notebooks:** `03a_carbon_benchmark`, `03b_demand_carbon`, `03c_calibration`; functions in `src/lantern/demand.py`, `carbon.py` and `calibration.py`.
- **Type:** physically based archetype profiles transferred to local weather and calibrated against measured gas use.
- **Specification:**
  - **Profiles:** hourly delivered heat per archetype and fabric from earlier EnergyPlus simulations (`ADR-019`).
  - **Weather transfer:** energy signature per archetype and fabric applied to each city's typical year; design day at the 99.6% heating design temperature with the shape of the coldest simulated days (`ADR-020`).
  - **Heat pump:** coefficient of performance as a function of temperature lift; flow 55 °C (system-only) and 45 °C (whole-house).
  - **Hot water:** SAP 10.2 Appendix J at Census occupancy; gas combi before conversion; heat-pump cylinder reheated off-peak after (`ADR-023`).
  - **Calibration:** one non-negative space-heating factor per built-form group fitted across LSOAs to DESNZ gas, with bootstrap intervals and a city-wide factor for small or poorly determined groups (`ADR-021`).
  - **Carbon:** gas, electricity (Green Book long-run marginal factor), self-consumed PV, refrigerant leakage and embodied carbon less residual value beyond 2050 (`ADR-024`).
  - **Benchmark:** CCC Balanced Pathway for heat in existing homes, localised by dwelling share (`ADR-018`).
- **Flats:** mid-terrace profile scaled by median EPC flat floor area (`ADR-022`).

### Intended use
- **Uses:** annual energy, design-day peaks and carbon per home of each archetype and pathway, as inputs to capacity, rollout and sequencing.
- **Out of scope:** dwelling-level energy assessment; hourly demand forecasting; bills.

### Factors
City, archetype, built-form group, pathway (existing, system-only, whole-house), weather case, design temperature, grid-factor scenario, occupancy case, hot-water timing.

### Metrics
Energy-signature fit; observed against modelled mean gas per meter by LSOA before and after calibration; holdout bias, mean absolute percentage error and R² over 50 splits; error by dominant group; calibration error by IMD quintile and its correlation with IMD score and fuel poverty; NEED cross-check.

### Data
Datasets 4–7 and 10 (`DATASHEET.md`).

### Quantitative analyses
`03a_benchmark_by_primary.csv`, `03a_benchmark_annual_study_area.csv`, `03b_energy_signatures.csv`, `03b_annual_energy_by_archetype.csv`, `03b_peaks_by_archetype.csv`, `03b_hot_water_by_archetype.csv`, `03b_carbon_summary_by_archetype.csv`, `03b_embodied_by_archetype.csv`, `03b_emission_factors.csv`, `03c_calibration_factors.csv`, `03c_holdout_validation.csv`, `03c_holdout_by_group.csv`, `03c_error_by_deprivation.csv`, `03c_lsoa_comparison.csv`, `03c_need_crosscheck.csv`, `03c_annual_energy_calibrated.csv`, `03c_peaks_calibrated.csv`.

### Ethical considerations
LSOA consumption statistics are published aggregates; no household data are used.

### Caveats and recommendations
- Calibration validates spatial variation in mean annual gas use, not hourly profiles or design-day demand.
- Factors vary by built form only; check `03c_error_by_deprivation.csv` before interpreting results by deprivation.
- The calibration factor combines fabric and occupant behaviour; the uncalibrated archetypes are the upper bound.
- DESNZ domestic gas includes cooking, and meters are not dwellings.

[Back to top](#-lantern--model-card)

---

# ⚡ Model 3 — Network headroom and homes within headroom

### Model details
- **Notebooks:** `02_network`, `04a_capacity`; functions in `src/lantern/network.py` and `capacity.py`.
- **Type:** deterministic headroom allocation; empirical coincidence factors; Monte Carlo estimate of homes within headroom.
- **Specification:**
  - **Primary headroom:** firm capacity − maximum demand − accepted connections (`ADR-015`).
  - **Deployable headroom:** the lower of a primary's own headroom and its pro-rata share of each upstream substation's headroom; imputation by operator mean ratio where upstream data are missing (`ADR-016`).
  - **Coincidence:** c(N) from the Electrification of Heat trial on cold days, with low, empirical and high cases; RHPP as a check (`ADR-025`).
  - **Spare capacity:** deployable headroom plus the hourly margin of existing demand below its maximum, from primary transformer flows (`ADR-026`).
  - **Homes within headroom:** homes added in random order until diversified load exceeds spare capacity at any hour; median over orders (`ADR-027`).
  - **Binding year:** first year in which the rollout exceeds homes within headroom.

### Intended use
- **Uses:** estimating primary-level capacity potential for heat pumps and when each primary runs out of headroom.
- **Out of scope:** connection decisions for individual homes; secondary-substation or low-voltage assessment; operator planning.

### Factors
Operator, primary, upstream level, headroom definition (deployable, primary-level, firm), coincidence case, peak metric (substation profile, highest hour, evening window), ordering (random, proportional, smallest- or largest-peak first), pathway, headroom stress.

### Metrics
Headroom lost to upstream limits; homes within headroom and as a share of gas-heated homes, with the Monte Carlo range; binding year; borrowed against own substation shape; cross-checks with operator tables and the GB dataset.

### Data
Datasets 2 and 8, with Models 1–2 (`DATASHEET.md`).

### Quantitative analyses
`02_headroom.csv`, `02_upstream_limits.csv`, `02_network_register.csv`, `02_nged_table3_crosscheck.csv`, `02_gb_dataset_crosscheck.csv`, `02_planned_works.csv`, `04a_coincidence_cases.csv`, `04a_coincidence_eoh.csv`, `04a_coincidence_rhpp.csv`, `04a_substation_shapes.csv`, `04a_borrowed_vs_own_shape.csv`, `04a_homes_within_headroom.csv`, `04a_headroom_range.csv`, `04a_headroom_stress.csv`, `04a_binding_year.csv`, `04a_summary.csv`.

### Ethical considerations
Results could be misread as connection availability; outputs use the wording of `ADR-006`.

### Caveats and recommendations
- Report deployable headroom with primary-level headroom as the upper bound.
- Secondary substations and low-voltage feeders can bind before the primary.
- Northern Powergrid and one NGED primary use a borrowed demand shape from a lockdown winter.
- Coincidence beyond the largest observed trial group is extrapolated and flagged.
- Competing load (electric vehicles, new homes) is not included.

[Back to top](#-lantern--model-card)

---

# 📈 Model 4 — Rollout scenarios

### Model details
- **Notebook:** `04b_rollout`; functions in `src/lantern/rollout.py`.
- **Type:** deterministic year-by-year scenarios, 2025–2050.
- **Scenarios:**
  - **Ra:** proportional conversion as uptake arrives, no headroom limit.
  - **Rb:** as Ra, with connections held at homes within headroom from the binding year until reinforcement, when deferred homes connect.
  - **Rc:** whole-house for all homes, constrained by whole-house homes within headroom.
  - **Rd:** system-only first, whole-house from a primary-specific switch year chosen for the lowest emissions (`ADR-029`).
- **Inputs:** rollout pathway scaled to each city (`ADR-028`); reinforcement delay D = 0, 2, 5, 8, 10 years (central 5); PV self-consumption from hourly generation and demand; refrigerant cohorts with replacement.
- **Indicators:** carbon cost of waiting (Rb − Ra); whole-house against waiting (Rc − Rb, Rd − Rb); household and primary break-even delay; carbon value of added headroom with fixed reinforcement dates (`ADR-030`); CCC alignment.

### Intended use
- **Uses:** the carbon consequence of network constraints with the order of conversion held fixed; when whole-house retrofit beats waiting.
- **Out of scope:** investment appraisal; programme costs; boiler replacement while waiting.

### Factors
City, primary, delay, pathway, rollout pace, embodied-carbon accounting, PV credit, refrigerant, heat-pump life, horizon.

### Metrics
Whole-life and direct emissions by scenario; Rb − Ra; Rc − Rb and Rd − Rb; break-even delays; tCO₂e per kilowatt of added headroom; benchmark overshoot; modelled against observed deployment.

### Data
Models 1–3; Datasets 6–8 (`DATASHEET.md`).

### Quantitative analyses
`04b_scenarios_by_primary.csv`, `04b_scenarios_city_annual.csv`, `04b_city_summary.csv`, `04b_carbon_cost_of_constraints.csv`, `04b_break_even_household.csv`, `04b_break_even_primary.csv`, `04b_headroom_value.csv`, `04b_prioritisation_indicator.csv`, `04b_ccc_alignment.csv`, `04b_accounting_boundaries.csv`, `04b_sensitivities.csv`, `04b_pv_self_consumption.csv`, `04b_heat_pump_refrigerant.csv`, `04b_observed_deployment.csv`, `04b_weather_years.csv`.

### Ethical considerations
As Model 3.

### Caveats and recommendations
- Rb is not bound by the annual allowance once reinforcement arrives; compare it with Ra, not with the sequencing strategies.
- Reinforcement dates are assumptions; read results across delays.
- The headroom value is a carbon indicator only; no costs are used.

[Back to top](#-lantern--model-card)

---

# 🧭 Model 5 — Sequencing strategies

### Model details
- **Notebook:** `05_sequencing`; functions in `src/lantern/sequencing.py`.
- **Type:** linear programmes solved with HiGHS; greedy rule-based allocations; rounding to whole homes.
- **Specification:**
  - **Decision:** homes of each archetype at each primary converting in each year by each pathway (`ADR-031`).
  - **Constraints:** pool; each city's annual allowance; design-day load at every hour against spare capacity until reinforcement, for every primary that runs out of headroom; after reinforcement the headroom limit is removed.
  - **Linear capacity:** hourly load per home scaled so the proportional system-only mix reaches the non-linear capacity exactly; every schedule checked against the non-linear capacity of its own mix and re-solved where it exceeds it by more than 1% (`ADR-032`).
  - **Strategies:** S0 proportional (reference); S1 highest existing gas emissions per home first; S2 largest saving per kilowatt of peak; S3 most spare capacity; S4 smallest peak; O1 lowest whole-life emissions; O2 lowest benchmark overshoot then whole-life; O3 overshoot-limited; "+" = whole-house allowed (`ADR-033`).
  - **Outputs:** shadow prices of headroom and delivery; waiting times by matching desired and scheduled conversions (`ADR-034`).

### Intended use
- **Uses:** comparing the emissions and timing of conversion orders under the same network, allowance and accounting.
- **Out of scope:** household-level programmes; cost optimisation; delivery logistics.

### Factors
City, strategy, pathway availability, headroom definition, reinforcement case, allowance (±20%), whole-house share limit, per-primary installation limit.

### Metrics
Whole-life and direct emissions relative to S0; benchmark overshoot and year of exhaustion; conversions by archetype and pathway; linear-constraint error by mix; largest exceedance after correction; shadow prices; median and 90th-percentile waits; share still on gas in 2050.

### Data
Models 2–4.

### Quantitative analyses
`05_strategy_summary.csv`, `05_annual_by_strategy.csv`, `05_installs_by_strategy.csv`, `05_conversions_by_archetype.csv`, `05_archetype_share_of_conversions.csv`, `05_budget_exhaustion.csv`, `05_tradeoff.csv`, `05_headroom_factors.csv`, `05_headroom_factor_validity.csv`, `05_linear_check_summary.csv`, `05_linear_check_excluded.csv`, `05_schedule_check.csv`, `05_nonlinear_correction.csv`, `05_rounding.csv`, `05_shadow_prices_headroom.csv`, `05_shadow_prices_delivery.csv`, `05_waiting_times.csv`, `05_waiting_summary.csv`, `05_sensitivities.csv`.

### Ethical considerations
A schedule that minimises emissions can defer some groups; distributional outcomes are assessed in Model 6 rather than assumed.

### Caveats and recommendations
- The schedules are aggregate planning schedules, not programmes for individual homes.
- Comparisons between S0 and "+" strategies combine order and pathway; use O1 to separate them.
- Shadow prices are local to the optimum.

[Back to top](#-lantern--model-card)

---

# 📐 Model 6 — Distributional and spatial measures

### Model details
- **Notebooks:** `06_equity`, `06b_maps`, `08_spatial`; functions in `src/lantern/equity.py`, `spatial_stats.py` and `maps.py`.
- **Type:** descriptive distributional indices; spatial autocorrelation statistics with permutation inference; counterfactual decomposition; parity-constrained linear programmes.
- **Specification:**
  - **Groups:** flats and houses; national IMD 2025 quintiles by LSOA; estimated fuel-poor households; tenure from Census shares; outcomes as mixtures of primary–archetype cells (`ADR-035`).
  - **Measures:** share still on gas 2030–2050 (report year 2035) and ratio to the city; mean wait among converted homes; concentration index across LSOAs ranked by IMD score (`ADR-036`).
  - **Parity:** O1+ re-solved with floors for single groups and all together; O4+, the parity-constrained allocation, = floor 1.0 for the most deprived 20% and fuel-poor households; O5+ adds a flat floor (`ADR-037`, `ADR-047`).
  - **Maps:** urban parts of LSOAs, pooled class breaks; dense-and-late LSOAs (`ADR-038`, `ADR-039`).
  - **Spatial statistics:** global, local and bivariate Moran's I on queen contiguity, six-nearest-neighbour alternative; Holm adjustment of global p-values; Benjamini–Hochberg for local clusters (`ADR-044`).
  - **Decomposition:** network, allocation and interaction components and the whole-house pathway effect, for S0, O1, O1+ and O4+ with and without the headroom limit (`ADR-043`).
  - **Checks:** primary-level correlation of headroom with deprivation (`ADR-045`); houses only (`ADR-046`).

### Intended use
- **Uses:** describing who and where remains on gas under each strategy, the carbon cost of parity rules, and whether the network or the order of conversion produces the pattern.
- **Out of scope:** household-level probabilities of conversion; procedural or recognition justice; lived experience; causal claims.

### Factors
City, strategy, network condition, group, year, weights matrix, parity floor, houses only.

### Metrics
Group shares and ratios; mean waits; concentration index; additional whole-life emissions of parity floors; global Moran's I, bivariate Moran's I and local cluster counts with adjusted p-values; decomposition components; Spearman correlation by primary with permutation p-value; dense-and-late zone composition.

### Data
Models 1–5; Datasets 1, 4, 5 and 9 (`DATASHEET.md`).

### Quantitative analyses
`06_group_outcomes.csv`, `06_group_composition.csv`, `06_distribution.csv`, `06_fairness_cost.csv`, `06_fairness_outcomes.csv`, `06_headroom_range.csv`, `06_tenure_check_epc.csv`, `06b_lsoa_indicators.csv`, `06b_bivariate_classes.csv`, `06b_still_on_gas_summary.csv`, `06b_dense_and_late.csv`, `08_network_vs_ordering.csv`, `08_network_vs_ordering_houses.csv`, `08_group_outcomes.csv`, `08_decomposition.csv`, `08_pathway_effect.csv`, `08_moran_global.csv`, `08_local_clusters_summary.csv`, `08_local_clusters_lsoa.csv`, `08_primary_deprivation.csv`, `08_primary_table.csv`, `08_flat_parity.csv`, `08_dense_late_zones.csv`.

### Ethical considerations
Area-level results could be read as statements about individual households or as evidence of discriminatory network practice; neither is supported. Results are reported as modelled, area-level outcomes, and the reference strategy S0 is stated with every comparison.

### Caveats and recommendations
- Outcomes are area-level and subject to ecological inference and the modifiable areal unit problem.
- The concentration index's attainable range depends on the city-wide share still on gas; compare runs with the same share.
- The decomposition is a model-based counterfactual; for Moran's I the interaction absorbs the non-additivity of the index.
- Primary-level tests rest on few units per city.
- O5+ is not re-checked against non-linear capacity.
- Dense-and-late LSOAs identify places for assessment, not feasible heat networks.

[Back to top](#-lantern--model-card)

---

# 🧪 Model 7 — Morris screening

### Model details
- **Notebook:** `07_screening`; functions in `src/lantern/screening.py`.
- **Type:** Morris elementary-effects screening (`ADR-040`).
- **Design:** ten inputs (coincidence, headroom, space-heating demand, heat-pump efficiency, grid factor, embodied carbon, PV credit, refrigerant leakage, rollout pace, reinforcement delay); four levels; 20 trajectories selected for spread from 200 candidates; 220 runs per city; seed in `config/study.yaml`. Inputs defined by alternatives are encoded on a 0–1 scale; ranges from evidence (`ADR-041`).
- **Model per run:** homes within headroom, headroom factors, per-home emissions, reinforcement dates, S0, O1+ and O4+ recomputed, including the non-linear capacity check; conditional on the stock, network and benchmark of Models 1–3.
- **Outputs:** homes within headroom; Rb − Ra; S0 − O1+; O1+ overshoot; O4+ − O1+; O1+ concentration index; O1+ share still on gas in 2035.
- **Additional cases:** base-run reproduction; linear headroom at low, empirical and high coincidence; heat-pump efficiency against field evidence; cold-spell case (`ADR-042`).

### Intended use
- **Uses:** ranking the influence of uncertain inputs on each output, and checking whether strategy rankings hold across the input ranges.
- **Out of scope:** probabilities of outcomes; uncertainty beyond the specified ranges; structural uncertainty (omitted network levels, archetype aggregation, allocation rule).

### Factors
City, output, input, number of trajectories.

### Metrics
μ*, μ and σ; μ* normalised by the largest μ* per output with 95% bootstrap intervals over trajectories; rankings resolved or not resolved; stability between 10 and 20 trajectories; ranking flags across all runs; output ranges; base-run reproduction error.

### Data
Models 2–6; Dataset 8 for efficiency evidence.

### Quantitative analyses
`07_design.csv`, `07_runs.csv`, `07_morris_indices.csv`, `07_ranking_stability.csv`, `07_ranking_flags.csv`, `07_output_ranges.csv`, `07_base_reproduction.csv`, `07_linear_check_cases.csv`, `07_coincidence_cases_constrained.csv`, `07_efficiency_evidence.csv`, `07_schedule_check_O4plus.csv`, `07_cold_spell.csv`.

### Ethical considerations
As Model 6.

### Caveats and recommendations
- Normalised μ* below 0.01 is a reporting convention for negligible influence, not a significance threshold.
- Screening results depend on the stated ranges; the grid range includes only scenarios of lower intensity than the main factor.
- Completed runs are reused on re-run when the seed and schedule tolerance are unchanged; delete the saved runs in `interim/07_screening/` after changing any other input.

[Back to top](#-lantern--model-card)
