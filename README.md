# Protecting Points, Preserving Patterns

Analysis notebooks for a master's thesis assessing utility restrictions of data protection
strategies (geographic masking, statistical masking, synthetic data generation, spatial
aggregation, and a software-agent approach) applied to geospatial health data, evaluated
through descriptive mapping, kernel density estimation, and global/local spatial
autocorrelation (Moran's I).

## Data

**This repository does not include any data.** 

To run these notebooks yourself, supply your own data under a local `data/` folder, following this layout:

```
data/
  original/2008_eov_illness_stand.shp        # original point dataset (input)
  population_grid/
    GHS_Hungary_pop.shp                      # population grid (input, statMask.ipynb)
    GHS_Hungary_pop_city-clipped2.shp        # population grid, city extent (input, most notebooks)
    GHS_Hungary_pop_centroids.shp            # population grid centroids (input, MaskMyPy.ipynb)
  donut_masking/                             # created by MaskMyPy.ipynb
  area_aggregation/                          # created by AreaAggregation.ipynb
  statistical_masking/                       # created by statMask.ipynb
  synthetic/                                 # created by SyntheticDataSDV.ipynb
```

Every notebook resolves this folder from a `DATA_DIR` variable at the top of its "Paths" cell
(default `"data"`, relative to wherever the notebook is run from). Override it without editing
any notebook by setting the `DATA_DIR` environment variable, e.g.:

```
DATA_DIR=/path/to/your/data jupyter execute AreaAggregation.ipynb
```

## Pipeline

The notebooks are independent scripts, not a single pipeline that runs end-to-end — but later
ones consume the shapefiles produced by earlier ones. Run them in this order:

1. **`MaskMyPy.ipynb`** — donut (random perturbation) masking of the original points.
2. **`AreaAggregation.ipynb`** — point-to-polygon aggregation onto the population grid.
3. **`statMask.ipynb`** — statistical masking (k-anonymity, quasi-identifier generalisation,
   within-cell perturbation).
4. **`SyntheticDataSDV.ipynb`** — synthetic data generation with SDV's Gaussian Copula
   synthesizer.
5. **`distribution_comparison_case_counts.ipynb`** — quantitative check of case counts per
   grid cell across all datasets, before the spatial analyses.
6. **`Descriptive-Mapping.ipynb`** — choropleth case counts per grid cell, for QGIS mapping.
7. **`Locally_Weighted_Average_Smoothing.ipynb`** — locally averaged (spatially smoothed)
   disease maps.
8. **`global_local_Morans_I.ipynb`** — global Moran's I and Local Moran's I (LISA) cluster
   classification.
9. **`Kernel-density.ipynb`** — kernel density estimation of case intensity.

Steps 1–9 run in a plain local Python environment. The fifth protection strategy assessed in
the thesis — a **software-agent approach** built on PySyft, where a restricted researcher only
ever receives disclosure-controlled results, never raw coordinates — is a separate,
Docker-based demo kept in its own subfolder: see
[`software-agent/README.md`](software-agent/README.md).

The Spatial Point Pattern Test (Andresen's test) used elsewhere in the thesis is run with an
external Java tool ([`spatialtest`](https://github.com/wsmalleson/spatialtest)), not included
here.

`archive/statMask-drop.ipynb` is an earlier statistical-masking approach (Gaussian coordinate
noise, no quasi-identifier generalisation) that was superseded by `statMask.ipynb` and is *not*
used for any result in the thesis. It is kept only for transparency about the method's history.

## Privacy notes for anyone re-running these notebooks

- Steps 6–9 apply a shared publication rule (`K_MIN` cases and `PMIN` population per grid cell,
  5 and 50 by default) before exporting any shapefile, applied identically to the original and
  every masked dataset, so that no resulting map can single out a sparsely populated,
  potentially identifiable cell.
- Do not add `print(...)`/`.head()`/`display(...)` calls on `odp`, `train_df`, or any
  intermediate DataFrame that still carries the original coordinates — see the note at the top
  of `SyntheticDataSDV.ipynb`. Cell outputs are stripped from every notebook in this repository
  before committing; keep it that way (e.g. `nbstripout` as a pre-commit hook) so a re-run never
  leaves real coordinates sitting in a committed `.ipynb` file.

## Setup

```
pip install -r requirements.txt
```

## References

The methods implemented here follow published work, cited in full in the accompanying thesis.
The core references per method:

- **Donut masking** (`MaskMyPy.ipynb`): Swanlund, D., & Schuurman, N. (2025). [MaskMyPy: Python
  tools for performing and analyzing geographic masks](https://doi.org/10.1186/s12942-025-00399-6).
  *International Journal of Health Geographics*, 24(1), 12.
- **Area aggregation** (`AreaAggregation.ipynb`): Armstrong, M. P., Rushton, G., & Zimmerman, D. L.
  (1999). [Geographically masking health data to preserve confidentiality](https://doi.org/10.1002/%28SICI%291097-0258%2819990315%2918:5%3C497::AID-SIM45%3E3.0.CO;2-%23).
  *Statistics in Medicine*, 18(5).
- **Statistical masking** (`statMask.ipynb`): Sweeney, L. (2002). [k-Anonymity: A model for
  protecting privacy](https://doi.org/10.1142/S0218488502001648). *International Journal of
  Uncertainty, Fuzziness and Knowledge-Based Systems*, 10(5), 557–570.
- **Synthetic data generation** (`SyntheticDataSDV.ipynb`): DataCebo, Inc. (2026). [Synthesize a
  Table (Gaussian Copula)](https://docs.sdv.dev/sdv/modeling/single-table-synthesizers/gaussiancopulasynthesizer).
  SDV documentation.
- **Smoothing, global/local Moran's I, kernel density**
  (`Locally_Weighted_Average_Smoothing.ipynb`, `global_local_Morans_I.ipynb`,
  `Kernel-density.ipynb`): Waller, L. A., & Gotway, C. A. (2004). *Applied Spatial Statistics for
  Public Health Data*. John Wiley & Sons. Implemented via PySAL/esda — Rey, S. J., & Anselin, L.
  (2010). [PySAL: A Python library of spatial analytical methods](https://doi.org/10.1007/978-3-642-03647-7_11).
  In *Handbook of Applied Spatial Analysis* (pp. 175–193). Springer.
- **Software-agent approach** (`software-agent/`): OpenMined. (2024). [PySyft](https://github.com/OpenMined/PySyft)
  (Version 0.9.5) [Computer software].
