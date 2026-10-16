# Software-Agent Approach (PySyft)

Docker-based demo of the fifth data-protection strategy assessed in the thesis: a
software-agent architecture built on [PySyft](https://github.com/OpenMined/PySyft) (v0.9.5),
where a restricted researcher never receives raw coordinates at all. Instead, the researcher
defines analysis functions against mock data, submits them as code requests, and only the
data owner's approved, sandboxed execution ever touches the real dataset — the researcher
receives just the (already disclosure-controlled) result.

This is a separate runtime from the rest of the repository (a Dockerised Jupyter Lab +
PySyft Datasite server, not a plain local Python environment), so it is kept in its own
subfolder with its own `Dockerfile` / `docker-compose.yaml` / `requirements.txt`.

## Data

**As with the rest of this repository, no data is included here.** Supply your own data
under a local `data/` folder next to `docker-compose.yaml` (already excluded by the
repository's root `.gitignore`, so it is never committed):

```
data/
  2008_eov_illness_stand.csv        # original point dataset (input, loaded by Owner_Final.ipynb)
  GHS_Hungary_pop_city-clipped2.shp # population grid, city extent (input)
  results/                          # created by Researcher_Final.ipynb (CSV / GeoPackage / Shapefile exports)
```

## Setup

```
docker compose up --build
```

This starts a Jupyter Lab at `http://localhost:8888` (no token/password — local dev only,
**not** safe to expose beyond your own machine) with two ports:

- `8888` — Jupyter Lab, for opening the notebooks below.
- `8093` — the PySyft "orchestra" Datasite server the notebooks launch and log into.

## Notebooks

Run in this order, in two separate Jupyter kernels (they represent two different parties):

1. **`notebooks/Owner_Final.ipynb`** — run as the *data owner*. Launches the local Datasite
   server, uploads the real point dataset and population grid as private assets (with mock
   versions the researcher can develop against), registers a restricted researcher account,
   and finally approves the researcher's code requests once they exist (step 12).
2. **`notebooks/Researcher_Final.ipynb`** — run as the *restricted researcher*. Logs in,
   confirms raw asset access is blocked (`SyftError: You have no access to the private data`),
   develops six analysis functions against mock data, submits them as code requests in one
   shared project, and — once the owner has approved them — executes each one for real and
   exports the results as CSV, GeoPackage, and ESRI Shapefile.

The six requested/approved functions: `visits_summary`, `create_published_grid_release`,
`create_smoothed_grid_release`, `global_morans_i_grid`, `local_morans_i_grid`,
`kernel_density_surface` — mirroring the descriptive, smoothing, and Moran's I / KDE analyses
run against the other four protection strategies elsewhere in this repository.

## Privacy notes

- Both notebooks apply the same shared publication rule as the rest of the repository:
  `K_MIN = 5` cases and `PMIN = 50` population per grid cell before any grid cell is released.
- The researcher never has `.data` access to the private assets — only `.mock` (synthetic,
  safe-to-inspect) versions during development, and the final, already-aggregated function
  output after owner approval.
- The credentials in the notebooks (`changethis`, `pw`, `researcher@example.com`) are placeholders for the
  local demo only — never reuse them anywhere else.
- Cell outputs are cleared from both notebooks before committing (same policy as the rest of
  this repository) — re-running them locally will repopulate real result tables under
  `data/results/`, which stay untracked via the root `.gitignore`.