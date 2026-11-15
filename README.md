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

-Selected-ROI analysis workflow.
-NASA/JPL Horizons integration.
-Solar azimuth, elevation, angular diameter, and distance calculations.
- Terrain-horizon generation from lunar elevation data.
-  Solar-disk visibility estimation.
- Ideal and terrain-adjusted solar-power estimates.
-  Earth geometric visibility analysis.
-  Visibility and illumination time-series generation.
-  Weighted candidate-site comparison.
- Interactive Gradio interface.
- Computational consistency checks on selected datasets and simulation outputs.

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


## Data Sources, Tools, Acknowledgements & Verification

### 1. NASA Data and Scientific Resources

INFINITUM was developed using publicly available lunar topography and astronomical data resources.

**NASA Goddard Space Flight Center – Planetary Geodesy and Geophysics Archive (PGDA)**

- [High-Resolution LOLA Topography for Lunar South Pole Landing Sites](https://pgda.gsfc.nasa.gov/products/78)
- [South Pole Landing Site LOLA DEMs](https://pgda.gsfc.nasa.gov/data/LOLA_5mpp/)

The project used the Site 01 (Connecting Ridge) lunar terrain dataset, including:
- The 5 m/pixel Lunar Digital Elevation Model (LDEM).
- Site 01 Regions of Interest (ROI) shapefiles.
- Terrain elevation information used to derive directional terrain horizons.

The PGDA documentation identifies the DEM products as south-polar stereographic datasets using the MOON_ME reference frame associated with DE421.

**NASA/JPL Horizons System**

- [JPL Horizons](https://ssd.jpl.nasa.gov/horizons/)
- [Horizons API Documentation](https://ssd-api.jpl.nasa.gov/doc/horizons.html)

The Horizons API was used to retrieve time-dependent Sun and Earth observer geometry, including azimuth and elevation information, for the selected lunar locations.

The project used Horizons as an astronomical ephemeris source, while local terrain obstruction and visibility calculations were performed separately.

**NASA Lunar Reconnaissance Orbiter – LOLA**

The lunar elevation products originate from the Lunar Orbiter Laser Altimeter (LOLA) data and processing resources documented by NASA/GSFC PGDA.

Additional archive reference:

- [LOLA Planetary Data System Archive](https://imbrium.mit.edu/)

### 2. Coordinate Reference Frame and Geospatial Verification

Coordinate consistency was treated as an important scientific consideration throughout development.

The NASA/GSFC PGDA documentation specifies that the Site 01 DEM uses south-polar stereographic X/Y coordinates in meters and the MOON_ME reference frame associated with DE421.

However, the original ROI shapefile metadata did not provide sufficient independent confirmation of every coordinate-reference detail needed to establish a complete transformation match with the Horizons observer coordinates.

The project team therefore investigated the coordinate definitions, projection metadata, axis conventions, and potential orientation differences.

The investigation included:
- Inspecting the original shapefile and its projection metadata.
- Inspecting the GIS project and GeoPackage coordinate-system information.
- Reviewing the south-polar stereographic projection and lunar reference-frame definitions.
- Comparing projected coordinates with geographic ROI coordinates.
- Contacting Michael K. Barker, associated with the PGDA lunar topography products, to request clarification regarding the original Site 01 ROI coordinate reference frame and its relationship to DE421 / MOON_ME.

A clarification request was sent, but no response was received before the submission deadline.

**Verification status:** The DEM reference-frame information is documented by PGDA. However, a complete independent verification of the original ROI shapefile orientation and its exact transformation consistency with the Horizons coordinate configuration was not completed before submission.

This is documented as an outstanding verification limitation, not as evidence that the coordinates are incorrect.

### 3. Scientific Methods and Processing

The project implemented its own processing and analysis workflows using the supplied terrain and astronomical data.

These included:

- Directional terrain-horizon extraction using the lunar DEM.
- Terrain obstruction analysis for the Sun and Earth.
- Solar disk visibility calculations.
- Solar irradiance and estimated panel-power calculations.
- Terrain-adjusted solar energy estimation.
- Earth disk visibility and geometric communication-window analysis.
- Time-series generation and multi-site comparison.

The terrain-horizon workflow sampled 360 azimuth directions for each selected ROI.

Solar calculations incorporated the solar constant, Sun-Earth distance correction, panel incidence assumptions, and terrain visibility.

The estimated solar power model used a reference panel area of 1 m² and an assumed conversion efficiency of 25%. These are model assumptions rather than measurements of an actual deployed lunar solar panel.

Earth visibility was evaluated geometrically using the apparent Earth position, angular size, and terrain obstruction. It should not be interpreted as a complete radio-frequency communication link budget.

### 4. Software, Programming Languages and Development Tools

The project was developed primarily in Python using Google Colab.

The main tools and libraries included:

- Python – primary programming language.
- NumPy – numerical computations.
- Pandas – data processing and time-series analysis.
- Requests – HTTP requests to external scientific APIs.
- Matplotlib – scientific plotting and visualization, where used.
- Plotly – interactive scientific charts.
- Gradio – interactive mission-control interface.
- QGIS – geospatial inspection and coordinate-system investigation.
- GDAL – geospatial raster and terrain-data support.
- JPL Horizons API – astronomical ephemeris retrieval.

Official documentation:

- [Python](https://www.python.org/)
- [NumPy](https://numpy.org/)
- [Pandas](https://pandas.pydata.org/)
- [Requests](https://requests.readthedocs.io/)
- [Matplotlib](https://matplotlib.org/)
- [Plotly](https://plotly.com/python/)
- [Gradio](https://www.gradio.app/)
- [QGIS](https://qgis.org/)
- [GDAL](https://gdal.org/)

### 5. Presentation Images and Visual Assets

Five JPG image files were used in the project's presentation.

Their original source URLs and complete attribution metadata were not preserved in the available local filenames and project records.

Consequently, their original creators and exact source locations could not be independently established before submission.

They are disclosed here as presentation assets with incomplete source attribution rather than being incorrectly attributed to NASA or another organization.

The lunar terrain and scientific analysis data described above are separately attributed to their documented scientific sources.

### 6. AI-Assisted Development Disclosure

ChatGPT (OpenAI) was used as an AI-assisted development tool during the project.

Its assistance included:
- Brainstorming and refining the project concept.
- Research assistance and technical explanations.
- Python code development, debugging, and troubleshooting.
- Assistance with scientific documentation and interpretation.
- Editing and improving the English project description and presentation content.

The scientific data retrieval, project-specific calculations, terrain processing, and resulting analysis were implemented and executed within the project's computational workflow.

AI assistance is acknowledged as a development aid and does not constitute independent scientific validation of the resulting calculations.

### 7. Limitations and Verification Status

INFINITUM is an analytical prototype intended to support the evaluation of candidate lunar landing locations.

The following limitations should be considered:

1. Coordinate-reference consistency between the original Site 01 ROI shapefile and the Horizons observer-coordinate configuration requires further independent verification.

2. Terrain obstruction results depend on the available DEM coverage, resolution, and the selected horizon-sampling method.

3. Solar energy estimates depend on stated panel-area and efficiency assumptions and should not be interpreted as hardware performance measurements.

4. Earth visibility represents geometric visibility, not a fully validated telecommunications performance assessment.

5. The analysis does not constitute a certified landing safety assessment or replace mission-specific engineering validation.

6. The five presentation JPG assets have incomplete original-source attribution in the available records.

The team made a best-effort attempt to document the scientific datasets, software, external resources, and AI-assisted contributions available during the hackathon. However, this record is not claimed to be an exhaustive inventory of every asset or intermediate resource used during development.

Further coordinate verification, source recovery, uncertainty analysis, and engineering validation remain potential future improvements.

### 8. Scientific Reference

Barker, M. K., et al. (2021). *Improved LOLA Elevation Maps for South Pole Landing Sites: Error Estimates and Their Impact on Illumination Conditions*. Planetary and Space Science, 203, 105119.

https://doi.org/10.1016/j.pss.2020.105119

NASA/GSFC PGDA – Site 01: Connecting Ridge.

https://pgda.gsfc.nasa.gov/products/78

NASA/JPL – Horizons System.

https://ssd.jpl.nasa.gov/horizons/

---

**Documentation note:** This section reflects the sources, tools, methods, and verification status that could be identified from the project's development records. It distinguishes documented facts, model assumptions, unresolved questions, and incomplete attribution rather than presenting unverified details as established facts.
