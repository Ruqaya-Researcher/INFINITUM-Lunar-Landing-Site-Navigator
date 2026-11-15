# INFINITUM
Lunar Landing Site Navigator

A terrain-aware analysis and comparison tool for lunar landing-site conditions, solar-energy availability, and Earth visibility.**

Developed for the **NASA International Space Apps Challenge 2026**  
Challenge: **Lunar Landing Site Navigator**

---

## 1. Project Overview

INFINITUM is a computational tool designed to help evaluate environmental conditions at candidate lunar landing locations.

Operating on selected Regions of Interest (ROIs) within **Site 01 – Connecting Ridge**, the project combines lunar terrain data with astronomical position calculations to investigate how local topography affects:

- Solar illumination and potential solar-energy generation.
- Direct geometric visibility of Earth.
- Terrain obstruction of the Sun and Earth.
- Duration and continuity of illumination and visibility windows.
- Relative performance of selected candidate locations.

Rather than treating solar illumination and Earth visibility as simple elevation-angle problems, INFINITUM incorporates terrain-derived horizon profiles to estimate whether the local lunar landscape obstructs the observed celestial bodies.

The project is implemented primarily in a **Google Colab notebook**, with a **Gradio-based Mission Control interface** for user interaction and result visualization.

## 2. What INFINITUM Does

### A. Solar Position and Terrain-Aware Illumination

For each selected ROI, the system calculates the Sun's:

- Azimuth.
- Elevation.
- Angular diameter.
- Distance from the Moon.

These values are evaluated against the local terrain horizon.

The system estimates:

- Solar-disk visible fraction.
- Whether the solar disk is fully visible, partially visible, or blocked.
- Ideal solar irradiance and power under the selected model assumptions.
- Terrain-adjusted solar irradiance and power.
- Accumulated solar energy over the simulation period.
- Illumination duration and interruption periods.

This distinction is important: a location may have a positive theoretical solar-power estimate while receiving no direct sunlight because a nearby terrain feature blocks the Sun.

### B. Earth Geometric Visibility

INFINITUM also evaluates Earth's position relative to each ROI and its local terrain horizon.

The analysis includes:

- Earth azimuth and elevation.
- Earth angular diameter.
- Terrain obstruction.
- Estimated visible fraction of Earth's apparent disk.
- Earth visibility duration.
- Continuous visibility windows and interruptions.

**Important:** Earth visibility in the current implementation is a geometric line-of-sight estimate. It does not establish that a usable communication link is available.

No complete radio-frequency link budget, antenna pointing model, transmitter power analysis, or communication-system performance validation is included.

### C. Terrain Horizon Analysis

The project uses lunar elevation data to derive local terrain-horizon profiles around the selected ROIs.

The generated horizon dataset contains angular terrain-elevation information sampled across azimuth directions.

The current horizon dataset uses **360 azimuth directions per ROI**, at one-degree angular intervals.

These profiles are used to evaluate terrain obstruction of the Sun and Earth.

Terrain coverage is limited by the available DEM extent. The analysis therefore uses the actual available terrain coverage rather than assuming that terrain data extends indefinitely or to an unsupported fixed distance.

### D. Solar-Energy Estimation

The solar model distinguishes between:

1. **Ideal solar power:** a theoretical estimate without terrain obstruction.
2. **Terrain-adjusted solar power:** an estimate accounting for the visible fraction of the solar disk.

The model uses the following stated baseline assumptions:

| Parameter | Value |
|---|---:|
| Solar constant at reference distance | 1361 W/m² |
| Reference distance | 1 AU |
| Assumed panel area | 1 m² |
| Assumed panel efficiency | 25% |

The model also accounts for solar distance and a panel-incidence factor.

These are configurable modeling assumptions, not specifications of a finalized lunar power system.

The resulting energy estimates should therefore be interpreted as **model-based estimates**, not guaranteed electrical output from a real spacecraft or surface installation.

### E. Candidate-Site Comparison

INFINITUM includes a weighted comparison framework for selected ROIs.

The current comparison weights are:

| Criterion | Weight |
|---|---:|
| Solar performance | 30% |
| Earth visibility | 30% |
| Terrain characteristics | 20% |
| Visibility and illumination continuity | 20% |

The comparison framework considers performance metrics derived from the simulation, including illumination, solar energy, Earth visibility, terrain horizons, and interruption durations.

Scores are normalized relative to the locations included in the comparison.

**Consequently, the ranking is comparative and dependent on the selected candidates and model assumptions. It is not an absolute scientific safety score or a certified landing-site suitability assessment.**

## 3. Demonstrated Results and Validation Checks

The project has undergone computational checks using the available lunar terrain data and astronomical calculations.

The following are examples of observed results from a diagnostic run covering 289 samples at five-minute intervals.

### Solar-obstruction diagnostic: ROI_1 and ROI_2

| Metric | ROI_1 | ROI_2 |
|---|---:|---:|
| Samples analyzed | 289 | 289 |
| Solar-disk visible fraction | 0 throughout | 1 throughout |
| Terrain horizon range | 3.007° to 3.589° | -0.375° to 0.210° |
| Maximum ideal power | 8.384 W | 8.410 W |
| Maximum terrain-adjusted power | 0 W | 8.410 W |

These results are consistent with the calculated geometry:

- **ROI_1:** the upper edge of the solar disk remains below the terrain horizon throughout the examined samples. The solar disk is therefore fully obstructed in this run.
- **ROI_2:** the lower edge of the solar disk remains above the terrain horizon throughout the examined samples. The solar disk is therefore fully visible in this run.

This diagnostic demonstrates why terrain obstruction must be considered separately from ideal solar irradiance.

It also confirms that the model can produce different terrain-adjusted solar-power estimates for locations experiencing nearly identical solar elevations but different local terrain horizons.

**These results apply to the examined simulation period and ROIs only. They should not be generalized to all lunar dates or all candidate locations.**

### Additional computational checks

The project also includes checks for:

- Consistency of selected ROI identifiers across analysis datasets.
- Availability of 360 terrain-horizon directions per ROI.
- Unique azimuth-direction coverage.
- Consistency of solar and Earth time-series records.
- Simulation sample counts and time-step intervals.
- Coordinate-conversion round-trip numerical consistency.

A small coordinate-conversion round-trip numerical error was observed in testing (approximately \(1.9 \times 10^{-9}\) m for the tested conversion). This indicates numerical consistency of that conversion test; it does not independently establish the correctness of the underlying geographic reference frame.

## 4. Data Sources and Computational Dependencies

### Astronomical calculations

The project retrieves astronomical ephemeris information through the **NASA/JPL Horizons System**.

- [JPL Horizons System](https://ssd.jpl.nasa.gov/horizons/)
- [JPL Horizons API Documentation](https://ssd-api.jpl.nasa.gov/doc/horizons.html)

The analysis uses the Horizons API to obtain the apparent geometric positions and relevant angular information for the Sun and Earth.

The configured reference center and coordinate settings are part of the computational implementation and should be considered when reproducing results.

### Lunar terrain and ROI data

The terrain analysis uses the supplied lunar elevation model and the Site 01 ROI dataset.

Principal processed inputs include:

- Lunar DEM: `Site01_final_adj_5mpp_surf.tif`
- Site ROI geometry: `Site01_ROIs` dataset
- Derived terrain-horizon dataset: `INFINITUM_Geographic_Terrain_Horizons.csv`
- ROI coordinate reference information and derived coordinate tables.

The processed files should be included in the repository or provided through an accessible data source if they are required to reproduce the analysis.

**Data provenance and coordinate-frame confirmation remain important limitations:** the precise compatibility of the original ROI geometry with the DEM reference frame and the intended lunar body-fixed frame has not been independently confirmed in this project.

No unverified coordinate-frame equivalence is claimed here.

## 5. Technical Implementation

| Component | Technology |
|---|---|
| Primary development environment | Google Colab |
| Programming language | Python |
| Numerical computation | NumPy |
| Data processing | Pandas |
| Astronomical ephemerides | NASA/JPL Horizons API |
| Terrain processing | Raster-based elevation analysis |
| Interactive interface | Gradio |
| Visualization | Python plotting and interactive interface components |

The notebook contains the computational workflow, analysis functions, result generation, and user-facing interface.

The project is designed around a notebook-based workflow rather than requiring a separate application server to reproduce its core analysis.

## 6. How to Run

1. Open the project notebook in Google Colab.
2. Ensure the required lunar terrain, ROI, and processed input datasets are available at the paths expected by the notebook.
3. Install or import the required Python dependencies.
4. Run the notebook cells in their intended order.
5. Configure the selected ROIs and simulation date/time or analysis interval.
6. Execute the astronomical and terrain-aware analysis.
7. Run the comparison workflow when multiple candidate ROIs are selected.
8. Launch the Gradio interface to inspect the results.

The notebook is the primary source of the project's implementation and configuration.

For reproducibility, the exact input datasets, dependency versions, analysis interval, and model assumptions should be retained alongside each reported result.

## 7. Scientific Assumptions and Limitations

INFINITUM is an analytical prototype, not a flight-qualified navigation or landing-safety system.

The following limitations must be considered when interpreting its outputs.

### 7.1 Coordinate-frame verification

The compatibility between the original ROI geometry, the terrain dataset, and the intended lunar body-fixed coordinate reference frame has not been independently confirmed in all respects.

Numerical coordinate-conversion consistency alone does not prove correct geospatial alignment.

### 7.2 Terrain-data limitations

Terrain-horizon estimates depend on:

- DEM resolution and accuracy.
- Available terrain coverage.
- ROI location and geometry.
- Terrain sampling and horizon-generation methods.
- Potential errors in the source elevation model.

Features outside the available DEM coverage cannot be evaluated by the current terrain model.

### 7.3 Solar-power model

The energy model uses simplified panel assumptions.

It does not fully simulate:

- A specified spacecraft or habitat power-system design.
- Battery charging and discharging.
- Storage losses and power-management constraints.
- Panel degradation, dust accumulation, or thermal behavior.
- Detailed panel tracking and deployment mechanisms.
- All possible surface and operational conditions.

The output is therefore an estimated solar-energy metric under stated assumptions.

### 7.4 Earth visibility is not communication availability

Geometric visibility does not guarantee a working communication link.

A complete communication assessment would require additional information, including antenna characteristics, radio frequencies, link budget, transmission power, receiver sensitivity, pointing constraints, and relay or ground-network availability.

### 7.5 Landing-site safety

The current implementation does not constitute a complete landing hazard assessment.

It does not independently certify:

- Surface slope and local roughness safety.
- Boulder and crater hazard avoidance.
- Subsurface properties.
- Landing dynamics and spacecraft stability.
- Thermal and radiation safety.
- Mission-specific operational constraints.

The comparison score must not be interpreted as a probability of successful landing or a formal safety certification.

### 7.6 Temporal and numerical resolution

Results depend on the chosen simulation interval, time-step size, angular sampling, and available input data.

Short-duration events or narrow visibility windows may require finer temporal and spatial resolution for more detailed investigation.

## 8. Current Development Status

The current prototype brings together the main computational components needed for a terrain-aware lunar environmental analysis:

- [x] Selected-ROI analysis workflow.
- [x] NASA/JPL Horizons integration.
- [x] Solar azimuth, elevation, angular diameter, and distance calculations.
- [x] Terrain-horizon generation from lunar elevation data.
- [x] Solar-disk visibility estimation.
- [x] Ideal and terrain-adjusted solar-power estimates.
- [x] Earth geometric visibility analysis.
- [x] Visibility and illumination time-series generation.
- [x] Weighted candidate-site comparison.
- [x] Interactive Gradio interface.
- [x] Computational consistency checks on selected datasets and simulation outputs.

### Not independently verified or not yet implemented

- [ ] Independent confirmation of all ROI/DEM lunar coordinate-frame relationships.
- [ ] Comprehensive validation against independent terrain-horizon reference measurements.
- [ ] Full validation of partial solar- and Earth-disk visibility across edge cases.
- [ ] Complete spacecraft-specific solar-array orientation and power-system modeling.
- [ ] Full radio-frequency communication link-budget analysis.
- [ ] Complete landing-hazard and safety assessment.
- [ ] Independent validation of all model outputs across multiple lunar dates and operating scenarios.

This distinction is intentional: implemented functionality, tested behavior, and future validation requirements are reported separately.

## 9. Future Development

Potential next steps include:

- Independent validation of lunar coordinate-frame alignment.
- Additional terrain-horizon verification against independent datasets.
- More extensive tests of partial celestial-disk obstruction.
- Configurable solar-array orientation and spacecraft power-system models.
- Communication link-budget and antenna-pointing analysis.
- Expanded landing-hazard metrics.
- Longer-duration simulations and automated visibility-window extraction.
- Further interface and visualization improvements.

## 10. Project Philosophy

INFINITUM is built around a central idea:

**A lunar landing location should not be evaluated from celestial geometry alone. Its surrounding terrain can fundamentally change the environmental conditions experienced at the surface.**

By combining astronomical ephemerides, terrain-derived horizons, solar-energy estimates, and Earth line-of-sight analysis, INFINITUM provides a unified framework for exploring these conditions across selected lunar candidate regions.

The project aims to make the analysis more accessible, interpretable, and useful while keeping its scientific assumptions and limitations explicit.

---

## Acknowledgments

Developed for the **NASA International Space Apps Challenge 2026**.

Astronomical ephemeris data are obtained through NASA/JPL Horizons.

Lunar terrain and ROI data are based on the datasets used in the Site 01 analysis workflow.

**INFINITUM — Exploring the conditions that shape the next steps on the Moon.**
