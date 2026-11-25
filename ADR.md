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
<a href="./MODEL_CARD.md"><strong>MODEL_CARD.md</strong></a>
</p>

# 🗺️ LANTERN — Architecture Decision Records

This document records the main design decisions behind the **LANTERN** (Local Allocation of Network headroom for Thermal Electrification and Retrofit in Neighbourhoods) notebooks, in a lightweight **MADR** (Markdown Architectural Decision Records) format. Each record gives the **Context**, the **Decision**, the **Alternatives considered** and the **Consequences**. Repository-wide decisions come first, followed by the decisions of each stage in run order. The dated working log behind these records is `docs/decisions.md`.

📖 Contents

- [Cross-cutting decisions](#-cross-cutting-decisions) (ADR-001–008)
- [Notebooks 01a–01c — Housing stock](#-notebooks-01a01c--housing-stock) (ADR-009–014)
- [Notebook 02 — Network headroom](#-notebook-02--network-headroom) (ADR-015–017)
- [Notebooks 03a–03c — Carbon benchmark, demand and calibration](#-notebooks-03a03c--carbon-benchmark-demand-and-calibration) (ADR-018–024)
- [Notebooks 04a–04b — Capacity and rollout](#-notebooks-04a04b--capacity-and-rollout) (ADR-025–030)
- [Notebook 05 — Sequencing](#-notebook-05--sequencing) (ADR-031–034)
- [Notebooks 06 and 06b — Who and where remains on gas](#-notebooks-06-and-06b--who-and-where-remains-on-gas) (ADR-035–039)
- [Notebook 07 — Screening of uncertain inputs](#-notebook-07--screening-of-uncertain-inputs) (ADR-040–042)
- [Notebook 08 — Spatial analysis and counterfactual decomposition](#-notebook-08--spatial-analysis-and-counterfactual-decomposition) (ADR-043–047)

[Back to README](./README.md#-lantern-local-allocation-of-network-headroom-for-thermal-electrification-and-retrofit-in-neighbourhoods)

---

# 🌐 Cross-cutting decisions

### ADR-001 — Study Areas Defined by Substation Supply Area

**Context.**

Network capacity is published per primary substation, while housing, deprivation and energy statistics are published per council, LSOA or output area. A study area drawn on council boundaries would cut primaries in two and count capacity that serves homes outside the area.

**Decision.**

The unit of analysis is the primary substation supply area. Core councils are Sheffield, and Nottingham City, Broxtowe, Gedling and Rushcliffe for Greater Nottingham (the built-up area, not a planning partnership). A primary is in the study area if at least 50% of its dwellings are in urban output areas of the core councils; once included, all of its dwellings count. Where two operators' supply areas overlap, homes are assigned to the Northern Powergrid primary.

**Alternatives considered.**

Council boundaries were rejected because they split primaries. Inclusion thresholds of 30% and 70% are reported as sensitivities rather than adopted, as they change the study area without changing the method.

**Consequences.**

Every result is attributable to whole primaries. Area statistics (LSOA, output area) are assigned to primaries by dwellings, and LSOA-level analysis is restricted to LSOAs with at least half of their dwellings in study-area primaries.

---

### ADR-002 — Data Outside the Repository, `raw/` → `interim/` → `processed/` → `outputs/`

**Context.**

Several inputs are licensed for local use only, and the raw files are large. The repository must be shareable without them.

**Decision.**

Data live in a separate local folder, `LANTERN_data/`, set in `config/paths.yaml`. Notebooks read raw files from `raw/` and never modify them. Each notebook writes to its own `processed/<stage>/` folder; work files reused between notebooks (schedules, screening runs) go to `interim/`. Small derived tables and figures that are safe to share are also written to the repository's `outputs/` folder. `.gitignore` excludes every data format, the local path file and the data folder.

**Alternatives considered.**

Keeping data inside the repository was rejected because licensed files could be committed by accident. Writing results back into the raw files was rejected because the authoritative source of a value would become unclear.

**Consequences.**

The repository can be published as it stands. A notebook can be re-run from a fresh kernel once the files of the notebooks before it exist, and the hand-offs between notebooks are visible from file names, which start with the number of the notebook that writes them.

---

### ADR-003 — One Configuration File for Every Setting

**Context.**

The model has many assumptions, sensitivities and seeds across 15 notebooks; values set in notebook cells would be repeated and could diverge.

**Decision.**

Every setting, assumption, sensitivity case, file pattern and seed is set once in `config/study.yaml`. Shared computations live in `src/lantern/` rather than in notebook cells. Each notebook has one settings cell that reads the configuration; the only per-machine value is the data path.

**Alternatives considered.**

Constants in each notebook were rejected because the same value would be maintained in several places.

**Consequences.**

A change to an assumption is made in one place and applies everywhere it is used, which is intended; sensitivity cases are defined as named alternatives in the configuration rather than by editing the main values.

---

### ADR-004 — Notebooks Generated From One Script

**Context.**

Fifteen notebooks need the same settings cell, checks cell, figure style and wording conventions.

**Decision.**

The notebooks are generated by `tools/build_notebooks.py`. Edits made in a notebook are carried back into the script before the next build. Each notebook opens with one settings cell, explains each step in a plain-English note, and ends with a checks cell that reports every check as PASS, WARN, FAIL or INFO and saves them as a table.

**Alternatives considered.**

Editing notebooks independently was rejected because shared conventions drifted between them.

**Consequences.**

Every notebook has the same structure and the same check reporting. Notebook markdown describes methods and does not quote results (`ADR-007`).

---

### ADR-005 — Run of Record With Checksums

**Context.**

The manuscript needs one fixed set of results, while the notebooks continue to change.

**Decision.**

A run of record is declared when every check is PASS, INFO or a documented WARN. Checksums of the configuration, code, executed notebooks, tables and figures are saved (`docs/frozen_run_20261117.csv`). Any later change to configuration, code or input data requires re-running the affected notebook and every notebook after it, with a new entry in `docs/decisions.md` and new checksums for the files that change.

**Alternatives considered.**

Updating figures and numbers piecemeal was rejected because the manuscript could then combine results from different runs.

**Consequences.**

Each reported number can be traced to a file of a recorded run. Additions that do not change earlier notebooks (new code read only by a later notebook, new settings blocks) leave earlier results valid, and this is stated in the log.

---

### ADR-006 — Capacity Potential, Not Connections

**Context.**

Counts of homes within headroom could be read as homes that can connect, and modelled delays as households' waiting experiences.

**Decision.**

Results are described as homes within deployable headroom (capacity potential at primary-substation level). Secondary substations and low-voltage feeders are not modelled and can bind first. "Waiting" is the modelled interval between a home's desired conversion year and the year a schedule converts it. Group and area outcomes are weighted mixtures of primary–archetype cells, not observations of households. Carbon differences between strategies are carbon differences, not programme costs.

**Alternatives considered.**

Reporting connectable homes or household waiting times was rejected because the model has neither the network levels nor the household data to support them.

**Consequences.**

The same wording is used in tables, figures and text. Competing load (electric vehicles, new homes) is out of scope, so homes within headroom are an upper bound in that respect.

---

### ADR-007 — Reported Values Generated From Saved Tables

**Context.**

Values typed into documentation or figure labels become outdated when notebooks are re-run.

**Decision.**

Notebook markdown describes methods and interpretation and does not quote results; results are printed by code cells and saved to `processed/` and `outputs/tables/`. Temperatures, energy and carbon are reported to one decimal place where possible, other results to two, and statistical results may keep more. Figures and tables do not refer to notebook numbers or to the rules or names of earlier studies; the mapping of display items to files is kept in the README.

**Alternatives considered.**

Quoting headline values in markdown cells was rejected because it is the main source of inconsistencies between runs.

**Consequences.**

Specific values are read from the regenerated files after a clean run.

---

### ADR-008 — Withholding Licensed and Address-Level Data

**Context.**

Verisk UKBuildings and the NGED primary supply-area polygons are licensed for local use only, and the located stock contains address-level records.

**Decision.**

The code is released under the MIT Licence and the derived tables and figures in `outputs/` under CC BY 4.0. Only tables aggregated to primary substation, archetype, LSOA or city leave the local machine. Maps drawn on NGED primary polygons are saved only under `processed/06_maps/local_only/`. Building-level and address-level intermediate files are withheld. Open and third-party inputs are not redistributed; their sources, versions and licences are listed in `docs/data_provenance.md`.

**Alternatives considered.**

Publishing an anonymised building-level stock was rejected because Verisk-derived attributes remain licensed after aggregation to buildings.

**Consequences.**

The notebooks cannot be re-run from public files alone. The results of the reported run are provided in `outputs/`, and the notebooks run unchanged on any local data folder of the documented structure.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebooks 01a–01c — Housing stock

### ADR-009 — Verisk for Dwellings, EPC for Attributes, Census for Checks

**Context.**

No single source covers every home with its built form, floor area and heating fuel. Verisk UKBuildings covers every building but not heating; EPC records cover heating and floor area for only part of the stock; Census 2021 gives counts by output area but no building attributes.

**Decision.**

Dwellings are counted from Verisk buildings with at least one address that are Residential or Mixed Use; Mixed Use addresses are counted as flats. EPC records are matched by UPRN for built form, floor area and fuel, and the share of each archetype heated by mains gas comes from EPC. Census 2021 is used to check flat and house counts by primary, scaled to the Verisk total.

**Alternatives considered.**

EPC alone was rejected because coverage varies by area and tenure. Residential-only Verisk buildings were rejected because they miss city-centre flats above shops and offices.

**Consequences.**

Mixed Use addresses include some non-domestic units; a sensitivity case without them is saved. Residential buildings without an address are not counted and are listed by type.

---

### ADR-010 — Fourteen House Archetypes and Flats

**Context.**

The demand profiles are defined for house archetypes by built form and size; flats use substation headroom too but had no profile of their own.

**Decision.**

EPC and Verisk records are classified into 14 house archetypes using the floor-area thresholds of the archetype set, with Verisk gross area first calibrated to EPC floor area by the median EPC/Verisk ratio of UPRN-matched houses of each built form. Flats and maisonettes are kept as a separate class. An unspecified Verisk "Terrace" is assigned end or mid from its matched EPC record, otherwise by the end/mid ratio of EPC homes of the same size in the same primary.

**Alternatives considered.**

Dropping flats was rejected because they draw on the same headroom. Matching the EPC and Verisk age bands exactly was not possible because the two sources use different bands; the age cut used for one archetype differs between them and is stated as a limitation.

**Consequences.**

Every home belongs to one of 15 classes. Flats are represented by a scaled profile (`ADR-022`), so results for flats depend on that proxy.

---

### ADR-011 — House-Type Buildings With Several Addresses

**Context.**

Verisk sometimes records a semi-detached pair or a terrace row as one building with several addresses, and elsewhere a house divided into flats.

**Decision.**

If the calibrated floor area per address is at least 30 m², each address is treated as a house with the building's form, sized by its area per address (an unspecified terrace row of n addresses becomes two end and n − 2 mid terraces); below 30 m², the addresses are flats. Residential buildings without a usable type, floor count or area are flats if they have several addresses and are spread over the primary's house archetypes if they have one.

**Alternatives considered.**

Fitting the threshold to Census flat counts was rejected because the fit kept moving towards the lowest value, driven by the Mixed Use addresses rather than by houses; 30 m² is a physical floor below the smallest space-standard flat.

**Consequences.**

The fit against Census is kept as a diagnostic table, not used to set the threshold.

---

### ADR-012 — EPC Fill Where Verisk Misses Homes

**Context.**

At a few primaries Verisk has the building footprints but labels them as non-residential with unknown type and use.

**Decision.**

Where Verisk dwellings are below 60% of Census 2021 households, the primary's stock is built from EPC (latest certificate per UPRN), with houses and flats scaled to the Census totals times a vacancy factor (the median Verisk/Census ratio of the city's other primaries, bounded between 1.00 and 1.15).

**Alternatives considered.**

Leaving the gap was rejected because it would understate demand at those primaries; scaling Verisk up was rejected because the missing buildings have no attributes.

**Consequences.**

The affected primaries are listed in the checks and stated as a limitation.

---

### ADR-013 — Urban Inclusion Computed on Census Households

**Context.**

The inclusion rule (`ADR-001`) needs the urban share of each primary's homes.

**Decision.**

Each output area is placed in the primary containing its representative point, and the urban share is the share of the primary's Census 2021 households in output areas classed urban in the 2021 rural-urban classification of the core councils.

**Alternatives considered.**

Area-weighted shares were rejected because large rural output areas would dominate.

**Consequences.**

The rule is reproducible from open data; the 30% and 70% thresholds are reported as sensitivities.

---

### ADR-014 — Weather Files and Their Limits

**Context.**

Demand is transferred to local weather (`ADR-020`), and the nearest typical-year files differ in quality.

**Decision.**

TMYx 2011–2025 files from Climate.OneBuilding.org are used: Watnall for Nottingham and Doncaster Sheffield airport for Sheffield, whose record ends in 2022 when the airport closed and whose station is lower and probably milder than the city. Design temperatures come from each file's statistics; a colder design temperature is a sensitivity.

**Alternatives considered.**

A single regional file for both cities was rejected because the comparison between cities is part of the study.

**Consequences.**

Sheffield peaks may be understated; the cold sensitivity and the cold-spell case (`ADR-042`) cover this.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebook 02 — Network headroom

### ADR-015 — Primary Headroom After Accepted Connections

**Context.**

The two operators publish headroom in different tables and on different definitions.

**Decision.**

Primary headroom is firm capacity minus maximum demand minus accepted connections. For Northern Powergrid, firm capacity and observed maximum load come from the Long Term Development Statement and accepted demand from the heat-map table, which reproduces the operator's headroom definition; MVA are converted to MW at a power factor of 0.95. For NGED, connected and contracted demand headroom come from the Network Opportunity Map, matched by substation number; the NGED LTDS table, which splits many primaries into transformer rows, is a cross-check only. Headroom before accepted connections is a sensitivity.

**Alternatives considered.**

Firm capacity minus maximum demand alone was rejected as the main case because accepted connections will use part of the capacity.

**Consequences.**

A register records, for every primary, the source, data vintage and a confidence level (high where firm capacity and demand are published, medium where only published headroom is available, low where headroom is missing).

---

### ADR-016 — Deployable Headroom From Upstream Limits

**Context.**

Primary-level headroom alone counts the same upstream capacity several times and treats primaries behind a constrained upstream substation as available.

**Decision.**

Deployable headroom is the lower of a primary's own headroom and its share of the headroom of each substation upstream. Each upstream substation's published headroom is shared among the substations it feeds directly in proportion to their own positive headroom, so the shares never add up to more than the upstream headroom. For Northern Powergrid, the grid supply point is shared among its supply points and directly fed primaries, and a supply point's usable headroom (the lower of its own and its grid supply point share) is shared among its primaries. For NGED, the bulk supply point's headroom is shared among all the primaries it feeds in the published map, including those outside the study area; grid supply point headroom is not published there and is not applied. Where a primary's upstream substation is not in the published tables, its headroom is scaled by the operator's mean ratio of deployable to own headroom.

**Alternatives considered.**

Treating missing upstream data as unconstrained was rejected. Optimising the allocation among primaries was rejected because it would represent a dispatch or connection policy that the operators have not stated.

**Consequences.**

Deployable headroom is the main metric and primary-level headroom is reported alongside it as the upper bound for every headline result. The proportional allocation is a modelling heuristic, and the imputed primaries are listed in the checks.

---

### ADR-017 — Planned Works Do Not Add Capacity

**Context.**

Both operators list planned works, but most are replacements or have uncertain completion dates.

**Decision.**

Planned works are matched to primaries and reported, but no capacity is added. Works already completed are taken as included in the published firm capacity.

**Alternatives considered.**

Adding planned capacity at the stated completion date was rejected because the works listed are mostly switchgear replacements and the dates are not commitments.

**Consequences.**

Reinforcement enters the model only through assumed dates (`ADR-028`), not through the operators' plans.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebooks 03a–03c — Carbon benchmark, demand and calibration

### ADR-018 — CCC Pathway for Heat in Existing Homes as the Benchmark

**Context.**

Cumulative emissions need an external reference, and the earlier benchmark calculation for the region used milestone values held flat across budget periods and a heating share from an older carbon budget.

**Decision.**

The benchmark is the CCC Seventh Carbon Budget Balanced Pathway for direct emissions from heat in existing homes, every year from 2025 to 2050, with the corrections log applied. It is localised to the study primaries by their share of the UK dwelling stock, summed from the four nations' statistics; a sensitivity weights the share by each council's domestic emissions per household relative to the national average.

**Alternatives considered.**

Budgets covering CO₂ only and all sectors were dropped because they require an assumed residential share. The earlier calculation is reproduced as a reconciliation, not used.

**Consequences.**

Direct (gas) emissions are compared with the benchmark; whole-life emissions are reported separately (`ADR-024`). The benchmark is an analytical reference, not a statutory local budget.

---

### ADR-019 — Archetype Profiles From Earlier Simulations

**Context.**

Hourly heating profiles are needed per archetype and fabric level; simulating them was outside the scope of this study.

**Decision.**

Hourly delivered-heat and cooling/ventilation profiles from the author's earlier EnergyPlus simulations of the archetype set are used: existing fabric for existing homes and system-only conversions, retrofitted fabric for whole-house conversions. Operational and embodied carbon by archetype come from the author's earlier archetype study.

**Alternatives considered.**

New simulations for both cities were outside the scope; generic national profiles would lose the archetype detail on which sequencing depends.

**Consequences.**

The profiles are transferred to local weather (`ADR-020`) and calibrated (`ADR-021`). The earlier studies' pathway results are not used.

---

### ADR-020 — Energy Signatures for Weather Transfer

**Context.**

The simulated profiles use a different weather year from the study cities.

**Decision.**

An energy signature (daily heat against daily mean temperature on days below 15.5 °C) is fitted for each archetype and fabric and applied to each city's typical weather. Design-day heat is the signature at the 99.6% heating design temperature, spread with the within-day shape of the ten coldest simulated days. Cooling/ventilation is scaled by cooling degree days.

**Alternatives considered.**

A degree-day ratio was rejected because it does not preserve the non-linear response at low temperatures.

**Consequences.**

The fit of each signature is reported. Peaks are per dwelling, before diversity (`ADR-025`).

---

### ADR-021 — Calibration by Built-Form Group Against LSOA Gas

**Context.**

The archetype profiles assume standard heating patterns and overstate measured gas use.

**Decision.**

Space heating is calibrated by one non-negative factor per built-form group (detached, semi-detached, terraced, bungalow, flat), fitted across LSOAs by non-negative least squares against DESNZ weather-corrected domestic gas per meter, with bootstrap intervals. LSOAs are used if at least 95% of their dwellings are in study-area primaries, they have at least 50 meters and the modelled gas-heated dwellings are within 0.75–1.33 of the meter count. Groups below 10% of modelled space-heating gas, or with a poorly determined factor, take the city-wide factor, and the larger groups are fitted with that part held fixed. The same factor is applied to whole-house space heat. Holdout validation refits the calibration on a random 80% of LSOAs and applies it to the other 20%, over 50 splits. The calibration error is also reported against deprivation and fuel poverty.

**Alternatives considered.**

Factors by deprivation or tenure were not fitted because LSOA data cannot separate them from built form with the available sample. Free fits for small groups were rejected because they absorb area effects such as communal meters; they are reported for comparison.

**Consequences.**

The factor combines fabric and occupant behaviour. The holdout validates spatial variation in mean gas use, not hourly profiles or design-day demand. The uncalibrated archetypes are kept as the upper bound.

---

### ADR-022 — Flats as a Scaled Mid-Terrace Profile

**Context.**

No simulated profile exists for flats.

**Decision.**

Flats use the mid-terrace profile scaled by the median EPC floor area of flats in the study primaries relative to the mid-terrace treated floor area; embodied carbon is scaled the same way. Flats have no rooftop PV.

**Alternatives considered.**

Excluding flats was rejected (`ADR-010`); a national average flat profile would lose the link to local floor areas.

**Consequences.**

Results for flats depend on this proxy. The distributional analysis is repeated for houses only (`ADR-046`).

---

### ADR-023 — Hot Water From SAP at Census Occupancy

**Context.**

The heating profiles exclude hot water, and hot-water reheat can coincide with the network peak.

**Decision.**

Hot water follows SAP 10.2 Appendix J month by month, with occupants from floor area scaled so that the dwelling-weighted mean matches Census 2021 household size. Existing homes use an instantaneous gas combi; heat-pump homes use an unvented cylinder reheated in an off-peak window (00:00–07:00), with storage and primary losses from SAP. Reheat at the time of each draw is a sensitivity.

**Alternatives considered.**

SAP occupancy from the archetypes' treated floor areas was rejected as the main case because it exceeds Census household size; it is kept as a sensitivity.

**Consequences.**

Off-peak reheat moves the daily maximum per dwelling to early morning, where it overlaps the heating ramp. Calibration is applied to space heating only, so occupancy is not absorbed into the factor.

---

### ADR-024 — Whole-Life Carbon Boundary

**Context.**

Strategies trade gas savings against embodied carbon of heat pumps, fabric and PV, at different times.

**Decision.**

Whole-life emissions over 2025–2050 include gas in unconverted homes; electricity for heating, hot water and cooling/ventilation at the Green Book long-run marginal factor (main) less self-consumed PV; refrigerant leakage (R290 main); and embodied carbon of the heat pump, fabric and PV, counted in the install year less the share of each item's life after the horizon. A heat-pump coefficient of performance that depends on the temperature lift is used, with flow temperatures of 55 °C for system-only and 45 °C for whole-house. PV is credited in operational carbon only and does not reduce the winter peak.

**Alternatives considered.**

Counting all embodied carbon upfront was rejected as the main case because it charges homes converting near 2050 for benefits outside the horizon; it is kept as a sensitivity. A fixed coefficient of performance was rejected because it ignores the effect of flow temperature.

**Consequences.**

Grid factors from the NESO Future Energy Scenarios and the Green Book grid average are sensitivities. Radiator upgrades have no embodied carbon value and are a stated limitation.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebooks 04a–04b — Capacity and rollout

### ADR-025 — Coincidence From the Electrification of Heat Trial

**Context.**

Homes do not reach their peaks at the same time, and the diversity of a group depends on its size and on the weather.

**Decision.**

The coincidence factor c(N), the group peak per home relative to the mean of each home's peak, is estimated from air-source homes in the Electrification of Heat trial on the ten coldest days with at least 100 complete homes, matching the definition of the design-day profiles so that diversity is not counted twice. Random groups are drawn for each group size on each day and the daily medians are averaged. High and low cases combine group composition and day-to-day variation. The factor applies to space heating; scheduled off-peak hot-water reheat is added undiversified. RHPP data are a check.

**Alternatives considered.**

A fixed after-diversity maximum demand per home was used only as a first look, because it ignores archetype and group size.

**Consequences.**

Results beyond the largest observed group size use a fitted decay and are flagged. Whether the trial homes represent the study stock is a stated limitation.

---

### ADR-026 — Existing Demand Shape From Primary Transformer Flows

**Context.**

New heat-pump load adds to existing demand, which peaks in the evening, while heat-pump load peaks in the early morning.

**Decision.**

Spare capacity at each hour is the deployable headroom plus the margin by which existing demand at that hour falls below its maximum, with existing demand shaped by NGED primary transformer flows on the coldest days of the published winter. Primaries without their own published flows, including all Northern Powergrid primaries, use the mean shape of the published primaries. Comparing the daily maximum per home with headroom, and the evening window alone, are sensitivities.

**Alternatives considered.**

Assuming that new and existing peaks coincide was rejected as the main case because it would understate capacity; it is kept as the conservative bound.

**Consequences.**

The published winter includes a lockdown period with unusual daytime demand, and Northern Powergrid shapes are borrowed; both are stated limitations, and the effect of substituting the mean shape is tested at primaries with their own flows.

---

### ADR-027 — Homes Within Headroom by Monte Carlo

**Context.**

The number of homes a primary can accept depends on which homes connect.

**Decision.**

For each primary, gas-heated homes are added in random order, with floor areas drawn from the primary's EPC records of each archetype, until the diversified load at any hour exceeds the spare capacity; the median over random orders is the main estimate. The proportional mix is the deterministic reference, and smallest-peak-first and largest-peak-first are ordering sensitivities. The binding year is the year in which the rollout first exceeds that number.

**Alternatives considered.**

A single deterministic mix was rejected as the main case because it hides the dependence on connection order.

**Consequences.**

The estimate assumes homes connect in random order and does not represent clustering of connections within a supply area. Stress tests reduce headroom by 10–30%.

---

### ADR-028 — Rollout Allowance and Reinforcement Dates as Scenarios

**Context.**

Neither local installation capacity nor reinforcement dates are published.

**Decision.**

Annual installations follow the CCC Balanced Pathway for existing homes, scaled by the city's share of UK dwellings and capped at the gas-heated pool; 50%, 75% and 125% of the pathway and a lower Warm Homes Plan rate are sensitivities. A primary is reinforced a fixed number of years after its binding year (central case five years; 0–10 years tested); after reinforcement its headroom limit is removed. Existing heat pumps add no starting load, as they are within measured demand and outside the gas-heated pool.

**Alternatives considered.**

Operators' planned works were not used as reinforcement dates (`ADR-017`). Holding the Warm Homes Plan rate after 2030 was rejected as the main case because it converts only a small share of the pool by 2050.

**Consequences.**

Reinforcement dates are assumptions, not investment plans; results are reported across delays.

---

### ADR-029 — Rollout Scenarios Ra–Rd

**Context.**

The carbon cost of network constraints needs a comparison that holds the order of conversion fixed.

**Decision.**

Four scenarios convert homes in proportional mix as uptake arrives: Ra without a headroom limit; Rb holding connections at the homes within headroom from the binding year until reinforcement, when the deferred homes connect; Rc whole-house for all homes; and Rd system-only first, switching to whole-house from a primary-specific year chosen for the lowest emissions. The carbon cost of waiting is Rb − Ra. Waiting homes keep their gas boilers.

**Alternatives considered.**

Comparing only optimised schedules was rejected because the effect of the network would be mixed with the effect of reordering (`ADR-043`).

**Consequences.**

Rb is not bound by the annual installation allowance once reinforcement arrives, so it is not directly comparable with the sequencing strategies, which are (`ADR-031`).

---

### ADR-030 — Carbon Value of Added Headroom With Fixed Reinforcement Dates

**Context.**

Added headroom can postpone a reinforcement trigger and move waiting into high-uptake years, which would make added capacity appear to raise emissions.

**Decision.**

The value of added headroom (100 kW to 1 MW, or 1–10% of firm capacity) is estimated with the reinforcement date fixed at its value without the addition.

**Alternatives considered.**

Letting the added headroom move the reinforcement date was rejected because the result would measure the trigger rule, not the capacity.

**Consequences.**

Values are tCO₂e avoided per kilowatt over 2025–2050 at each delay; no cost data are used, so the ranking of primaries is a carbon indicator only.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebook 05 — Sequencing

### ADR-031 — Linear Programme Over Primaries, Archetypes, Pathways and Years

**Context.**

Sequencing must respect each primary's hourly capacity, each city's annual allowance and the housing pool together.

**Decision.**

The decision variable is the number of homes of each archetype at each primary converting in each year by each pathway. Constraints are the pool, each city's annual allowance and, for every primary that runs out of headroom, the design-day load at every hour against spare capacity until reinforcement. The hourly load per home is scaled so that the proportional system-only mix reaches exactly the non-linear capacity of `ADR-027`. O1 minimises whole-life emissions; O2 minimises benchmark overshoot and then whole-life emissions; O3 limits overshoot between the two. The programme is solved with HiGHS, rounded to whole homes with every constraint rechecked, and reported with shadow prices.

**Alternatives considered.**

A household-level integer programme was rejected because the stock is represented by cells, not individual homes.

**Consequences.**

The result is an aggregate planning schedule. "+" marks runs in which whole-house retrofit is allowed; without it the run is system-only.

---

### ADR-032 — Every Schedule Within the Non-Linear Capacity

**Context.**

The linear constraint is exact for the proportional mix only; an optimiser can choose mixes that it overstates.

**Decision.**

Every schedule is checked against the non-linear capacity of its own archetype mix at each constrained primary-year. Where a primary exceeds it by more than 1%, its linear limit is tightened by the largest exceedance and the schedule re-solved until no primary-year exceeds 1%. Mix checks report the error of the linear constraint for whole-house, 50/50, higher-peak and lower-peak mixes against thresholds set before the run.

**Alternatives considered.**

Reporting a single agreement percentage was rejected because the error depends on the mix the schedule chooses.

**Consequences.**

The flat-parity scenarios O5+ are not re-checked and are reported as conditional comparisons.

---

### ADR-033 — Proportional Order (S0) as the Reference

**Context.**

Distributional effects need a reference that treats every group within a primary alike.

**Decision.**

S0 converts each primary's homes in proportion to its archetype mix as uptake arrives, within headroom and the annual allowance. Simple rules prioritise the highest gas emitters (S1), the largest carbon saving per kilowatt of peak (S2), primaries with the most spare capacity (S3) and the smallest peaks (S4). Every strategy shares the same stock, allowance, headroom, reinforcement dates, accounting and horizon.

**Alternatives considered.**

Observed market-led uptake was not used as the reference because it is not available at primary level and would combine allocation with household choice.

**Consequences.**

S0 is socially neutral within each primary by construction, so any distributional pattern under S0 arises through the network. S0 is an analytical reference, not a forecast.

---

### ADR-034 — Waiting Times by Matching Desired and Actual Conversions

**Context.**

Waiting needs a definition that is the same across strategies.

**Decision.**

Each primary's desired uptake follows the rollout trajectory, the same under every strategy. Within each primary–archetype cell, homes are matched in order between desired uptake and the schedule's conversions; the wait is the difference in years. Medians and 90th percentiles are reported among converted homes, with homes still waiting and the share still on gas in 2050 reported separately.

**Alternatives considered.**

Giving homes still on gas in 2050 an assumed wait was rejected because the value would be arbitrary.

**Consequences.**

Mean waits exclude homes never converted by 2050, so they are read alongside the share still on gas.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebooks 06 and 06b — Who and where remains on gas

### ADR-035 — Groups From Cells and Area Shares

**Context.**

Household-level deprivation, fuel poverty and tenure are not available for the modelled homes.

**Decision.**

Groups are flats and houses (archetype); national IMD 2025 quintile from the decile of the home's LSOA; estimated fuel-poor households from the LSOA share in fuel poverty; and tenure from Census 2021 output-area shares. Every home in a primary–archetype cell is treated alike, so a group's outcome is the mixture of its cells' outcomes weighted by the group's share of each cell.

**Alternatives considered.**

Combining tenure with dwelling type was not possible because the joint distribution is not published at output-area level; EPC tenure by property type is reported as a check.

**Consequences.**

Group differences arise only from location and archetype. Results are area-level and subject to ecological inference.

---

### ADR-036 — Concentration Index of Homes Still on Gas

**Context.**

Group ratios show who is left on gas but not how the share is distributed across the deprivation ranking.

**Decision.**

The concentration index of the LSOA share still on gas, ranked by IMD score and weighted by homes, is reported with group ratios to the city. The report year is 2035; 2030–2050 are given in the supplementary tables.

**Alternatives considered.**

Normalised or corrected indices were not adopted as the main measure; the index's attainable range depends on the mean, so runs are compared where the city-wide share still on gas is the same.

**Consequences.**

A negative index means homes still on gas are concentrated in more deprived LSOAs.

---

### ADR-037 — Parity Floors and the Parity-Constrained Allocation (O4+)

**Context.**

The carbon cost of distributional targets needs to be measured under the same constraints as O1+.

**Decision.**

O1+ is re-solved with a parity constraint for one group at a time (flats, most deprived 20%, fuel-poor, social rented), then all together: in every year, the group's share converted so far is at least a floor (0.9 or 1.0) times the city's share. O4+, the parity-constrained allocation, applies a floor of 1.0 to the most deprived 20% and fuel-poor households together. The cost is the increase in whole-life emissions over O1+.

**Alternatives considered.**

Weighting the objective by deprivation was rejected because the weight would be arbitrary; floors are interpretable as programme rules.

**Consequences.**

The cost is a carbon difference, not a financial or delivery cost (`ADR-006`).

---

### ADR-038 — LSOA Maps of the Urban Part Only

**Context.**

Large rural LSOAs dominate area maps, and a grey rural fill can be mistaken for a data class.

**Decision.**

LSOAs are dissolved from 2021 output areas. An LSOA is mapped if at least 50% of its dwellings are in study-area primaries and belongs to the city with most of them. Only the urban part of each LSOA is coloured; rural parts are white with light grey dots. Classes are quintiles of both cities pooled (tertiles for bivariate maps), so colours compare across cities. Both cities are drawn at one scale.

**Alternatives considered.**

Separate class breaks for each city were rejected because the same colour would mean different values.

**Consequences.**

Place and council names sit in the margins with leader lines so that no label covers a result.

---

### ADR-039 — Dense-and-Late LSOAs as a Screening Indicator

**Context.**

Places where heat-demand density is high and conversion is late are candidates for comparing heat networks with individual heat pumps.

**Decision.**

Heat-demand density is domestic gas per km² of the LSOA's urban land. An LSOA is dense and late if it is in the top density quintile of both cities pooled and in the top third for the share still on gas in 2045 under O1+. Contiguous dense-and-late LSOAs are grouped into zones in 08.

**Alternatives considered.**

Applying a heat-network viability threshold was rejected because viability depends on costs and building-level conditions not modelled here.

**Consequences.**

The indicator identifies places for further assessment; it does not establish that communal or networked heating is feasible.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebook 07 — Screening of uncertain inputs

### ADR-040 — Morris Screening of Ten Inputs

**Context.**

The model has many uncertain inputs; a full variance-based analysis would need far more runs than the sequencing allows.

**Decision.**

Morris elementary effects are computed for ten inputs (coincidence, headroom, space-heating demand, heat-pump efficiency, grid factor, embodied carbon, PV credit, refrigerant leakage, rollout pace and reinforcement delay) on seven outputs, with four levels and 20 trajectories selected for spread from 200 candidates, giving 220 runs per city. Influence is μ* normalised by the largest μ* for each output, with bootstrap intervals over trajectories; inputs below 0.01 are reported as negligible, and rankings are marked as resolved only where intervals do not overlap. Inputs defined by alternatives are encoded on a 0–1 scale. Runs are saved as they finish and resumed.

**Alternatives considered.**

Sobol indices were rejected because of the number of runs required.

**Consequences.**

The screening ranks influence within the specified ranges; it is not a probabilistic uncertainty analysis. The base run must reproduce the main results.

---

### ADR-041 — Input Ranges From Evidence

**Context.**

Screening results depend on the ranges given to each input.

**Decision.**

Ranges are taken from evidence where it exists: demand ±10% from the calibration holdout error; heat-pump efficiency from the interquartile range of in-situ performance of metered installations, with RHPP and Electrification of Heat shown alongside; coincidence from its low and high cases; headroom from deployable to primary-level; rollout pace from 0.5 to 1.25 times the CCC pathway; and reinforcement delay from 0 to 10 years.

**Alternatives considered.**

Uniform ±20% ranges for every input were rejected because they would rank inputs by an arbitrary choice.

**Consequences.**

The grid range includes only scenarios of lower intensity than the main factor, and embodied carbon varies as one factor; neither changes the rankings, and both are stated.

---

### ADR-042 — Cold-Spell Case Outside the Screening

**Context.**

Inputs that are each plausible can combine into a severe case that screening does not show.

**Decision.**

A cold-spell case combines high coincidence, demand +10% and the lowest efficiency, with deployable headroom, and is reported against the base run.

**Alternatives considered.**

Including the combination in the screening was rejected because Morris designs vary one input at a time.

**Consequences.**

The case shows how far the outputs move under joint stress, without a probability attached.

[Back to top](#️-lantern--architecture-decision-records)

---

# 📍 Notebook 08 — Spatial analysis and counterfactual decomposition

### ADR-043 — Network, Allocation and Interaction Components

**Context.**

Differences between strategies combine the effect of the network with the effect of the order of conversion.

**Decision.**

S0, O1, O1+ and O4+ are each run with deployable headroom and with no headroom limit, with the allowance and pool unchanged. Writing Y(o, n) for an outcome under ordering o and network condition n: network = Y(S0, H) − Y(S0, N); allocation = Y(O, N) − Y(S0, N); interaction = Y(O, H) − Y(O, N) − Y(S0, H) + Y(S0, N). The pathway effect of whole-house retrofit is Y(O1+, n) − Y(O1, n) under each condition. Outcomes are whole-life emissions, the concentration index, Moran's I and bivariate Moran's I.

**Alternatives considered.**

Comparing optimised schedules with S0 under headroom only was rejected because order and network would be confounded.

**Consequences.**

The components sum to the total by construction. For Moran's I they are differences between runs and the interaction also absorbs the non-additivity of the index. The decomposition is a model-based counterfactual, not causal identification, and the no-limit case is an analytical reference, not a feasible network state.

---

### ADR-044 — Spatial Weights and Inference

**Context.**

Clustering statistics depend on the weights matrix, and many runs are tested.

**Decision.**

Queen contiguity of whole LSOA polygons within each city, row-standardised, is the main weights matrix; six-nearest-neighbour weights on LSOA centroids are the alternative. Global Moran's I uses 999 random permutations; local Moran's I uses 999 conditional permutations with a Benjamini–Hochberg false discovery rate of 5%; bivariate Moran's I relates each LSOA's IMD score to the share still on gas in neighbouring LSOAs. Global p-values are Holm-adjusted over all runs of each city. The implementation was checked against PySAL on a test grid.

**Alternatives considered.**

Unadjusted p-values were rejected because several runs per city are tested together.

**Consequences.**

The smallest attainable adjusted global p-value is set by the number of permutations and runs. Results that change under the alternative weights are reported.

---

### ADR-045 — Headroom and Deprivation by Primary

**Context.**

Whether network capacity itself is distributed unequally is a separate question from who is left on gas.

**Decision.**

The share of gas-heated homes within deployable headroom is correlated with the homes-weighted mean IMD score of each primary (Spearman, 9,999 permutations, per city), and homes in primaries that bind before 2050 are compared with the rest.

**Alternatives considered.**

LSOA-level correlation was rejected because headroom is defined per primary.

**Consequences.**

The test has limited power with the number of primaries in each city, and is stated as such.

---

### ADR-046 — Houses-Only Check

**Context.**

Flats use a scaled profile (`ADR-022`), which could drive the distributional results.

**Decision.**

The network-or-ordering measures are repeated with flat cells left out.

**Alternatives considered.**

A flat-specific profile was not available.

**Consequences.**

The check shows whether the deprivation gradient depends on the flat proxy.

---

### ADR-047 — Flat Parity (O5+) and Dense-and-Late Zones

**Context.**

Flats are deferred by the carbon objective, and the cost of bringing them forward needs to be stated with its limits.

**Decision.**

O5+ adds a parity floor for flats (0.5, 0.75 and 1.0) to O4+, with deployable headroom. Dense-and-late LSOAs (`ADR-039`) are grouped into contiguous zones, with their deprivation, flats, social renting, fuel poverty, homes in binding primaries and overlap with high–high clusters.

**Alternatives considered.**

Including O5+ among the main strategies was rejected because it is not re-checked against non-linear capacity (`ADR-032`).

**Consequences.**

O5+ results are conditional comparisons under the scaled flat profile. The zones describe where place-based assessment could start.

[Back to top](#️-lantern--architecture-decision-records)
