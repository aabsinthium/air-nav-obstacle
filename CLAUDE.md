# air-nav-obstacle — great-circle air routing around restricted airspace

Research notebooks for air routes that follow great circles and go
around closed or restricted regions. The current method is a spherical
visibility graph: the nodes are the route endpoints and the vertices of
the obstacle polygons, and an edge is a great-circle arc that does not
go through an obstacle. A\* with the direct great-circle distance as the
heuristic finds the shortest route on this graph.

Code conventions load automatically from `.claude/rules/` when you edit
Python or notebook files. New and changed prose for people (docs,
commit messages, code comments, markdown cells) follows ASD-STE100
Simplified Technical English, STE-flavored mode. Load the `ste-writing`
skill when one is available.

## Current focus

The spherical visibility graph and A\* work for one hand-made obstacle
(the Bishkek–Novosibirsk–Atyrau triangle) on the route UAAA → KJFK. Next:
use real country borders from `data/` as obstacles. Keep this line
current — it shifts often.

## Layout

```
air-nav-obstacle/
├── visibility_graph.ipynb   # active work: spherical geometry, visibility graph, A*, globe plots
├── data/                    # tracked inputs, all in Git LFS (see Large files and Git LFS)
│   ├── airports.csv         # ~6.4k airports: ICAO/IATA, city, country, GeoPointLat/GeoPointLong
│   └── geoBoundaries-<ISO3>-ADM<0|1>[_simplified].geojson   # RUS, KAZ, UKR, BLR borders
├── .claude/rules/           # path-scoped conventions (style, design vocabulary, notebooks + geometry)
└── .old/                    # gitignored archive of superseded notebooks (never tracked)
```

`.old/` holds the earlier approaches: HEALPix grids, dense-graph
pathfinding, cuGraph on CUDA, and STRtree polygon queries. Read them for
context. Do not edit them, and do not move them back into git.

## Environment

- Jupyter kernel `geoenv`: numpy, pandas, networkx, geopandas, shapely,
  plotly, matplotlib. The `.old/` notebooks also used `cugraph` and
  `healpy`. These are not in the current work.
- Edit and run notebooks only through the Jupyter MCP tools. The
  notebook can be open in JupyterLab as a live collaborative document
  (`.jupyter_ystore.db`), and a direct write to the `.ipynb` file is
  lost or overwrites unsaved edits.
- Git strips notebook outputs on commit with nbstripout (installed in
  `geoenv`; `filter.nbstripout.required` is set, so a commit fails if
  the filter is missing). The filter definition is in `.git/config`, not
  in git: after a fresh clone, run `nbstripout --install` from `geoenv`.
  See `.claude/rules/notebooks.md` for how to keep outputs in a
  showcase notebook.
- History was rewritten twice on 2026-11-07: to strip notebook outputs,
  then to move `data/**` into Git LFS. No local backup is kept. The old
  history (original SHAs, notebook outputs, no LFS) is still on GitHub
  under the read-only ref of PR #1, `refs/pull/1/head` (`635038c`). To
  recover it: `git fetch origin pull/1/head:refs/old-history`.

## Large files and Git LFS

Keep git history small and text-only. These go into Git LFS, never into
plain git:

- **Binaries**: arrays and tables (`.npy`, `.npz`, `.pkl`, `.h5`,
  `.parquet`, `.feather`), images, PDFs and archives.
- **Geofiles**: vector and raster geodata (`.geojson`, `.topojson`,
  `.gpkg`, `.fgb`, `.kml`/`.kmz`, shapefile `.shp`/`.shx`/`.dbf`,
  `.tif`/`.tiff`, `.nc`), wherever they are in the repo.
- **Large CSV/JSON**: any CSV or JSON file larger than about 1 MB. Put
  it under `data/`, where everything goes to LFS by path. Small CSV/JSON
  (configs, short lookup tables) stays in plain git, so it stays
  diffable.

How to apply it:

- `.gitattributes` holds the rules in three groups: the notebook filter,
  LFS by path (`data/**`), and LFS by extension. Before you commit a new
  binary or data format, check that it matches a rule
  (`git check-attr filter -- <path>` must say `lfs`). If it does not,
  add the extension or the path to `.gitattributes` in the same commit.
- Check with `git lfs ls-files` after the commit. A file that went into
  plain git by mistake needs `git lfs migrate import`, which rewrites
  history — so check before the commit, not after.
- Notebooks are not in LFS: nbstripout keeps them small. Do not save
  figures or data next to a notebook outside `data/` unless an LFS rule
  covers them.
- A fresh clone needs `git lfs install` once per machine (already done
  on this one). GitHub counts LFS storage and every LFS download
  against the account quota.
- GitHub limits: a warning at 50 MB per file and a block at 100 MB in
  plain git. LFS has a 2 GB per-file limit on the free plan.

## Geometry conventions

These hold across all code. A change to one of them is a design
decision, not a refactor.

- **Axis order is `(lon, lat)`**, as in GeoJSON. `airports.csv` stores
  `GeoPointLat` before `GeoPointLong`, so convert explicitly on load.
- **All geometry is on the unit sphere.** A point is a 3D unit vector
  (`to_unit`), an edge is a great-circle arc, and a distance is the
  central angle × `R_EARTH` (6371.0088 km, the mean radius). Do not mix
  in planar shapely predicates for routing decisions: they are wrong for
  long arcs and at the antimeridian.
- **`angle()` uses `atan2(|u × v|, u · v)`**, not `arccos(u · v)`. It
  stays accurate near 0 and π. Keep it.
- **Tolerance is `EPS = 1e-9`** (radians). Touching at a shared endpoint
  is not a crossing.
- **Polygon rings go clockwise** for the Plotly globe. On a sphere, d3
  fills the region to the left of the ring, so a counter-clockwise ring
  paints everything except the polygon.
- **The A\* heuristic must stay admissible.** The direct great-circle
  distance never overestimates, so A\* stays exact. Do not add a
  weighted or inflated heuristic without a written reason.

## Extending the repo

The project is at the prototype stage, so all logic is in the notebook
now. When a function becomes stable and a second notebook needs it,
move it into a repo-root `src/` package (for example
`src/sphere.py` for the vector geometry, `src/visibility.py` for the
graph) and import it from the notebooks. After that move, notebooks only
orchestrate: calls, inspection, plots and file writes. Do not create
`src/` for code that only one notebook uses.
