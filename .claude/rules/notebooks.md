---
paths:
  - "**/*.py"
  - "**/*.ipynb"
---

# Notebook rules & gotchas

## Rules

- Edit and run notebooks only through the Jupyter MCP tools. Never
  write the `.ipynb` file directly: it can be open in JupyterLab as a
  live collaborative document, and a direct write is lost or overwrites
  unsaved edits.
- Git strips notebook outputs on commit (nbstripout filter, see
  `.gitattributes`). The file on disk keeps its outputs; only the
  committed copy is clean. To commit the outputs of a showcase notebook
  for GitHub, set `"keep_output": true` in its notebook metadata, render
  Plotly figures as static PNG too (GitHub does not run JavaScript), and
  commit only after a clean Restart & Run All. Set this flag only when
  the user asks for it.
- A notebook must survive a clean Restart & Run All, top to bottom.
  Verify with `python -m nbconvert --to notebook --execute --inplace
  <nb>` on the `geoenv` kernel, or with a kernel restart and a run of
  all cells through the Jupyter MCP tools. Out-of-order execution counts
  are a sign that the notebook depends on hidden state.
- Keep scratch exploration out of the main narrative. Remove a scratch
  cell when it has done its job, or move it below a `## Scratch`
  markdown heading at the end. Do not commit a cell that raises an
  error unless the error is the point of the cell, and then say so in a
  comment.
- Each code cell that defines core logic ends with a short smoke test
  that prints a known value (for example, Almaty → JFK ≈ 10,300 km).
  Keep these when you refactor: they are the regression check.
- When logic moves to `src/` (see "Extending the repo" in `CLAUDE.md`),
  the notebook only contains calls, inspection, plots and file writes.
  New logic goes into the module first.
- Results are deterministic: the same nodes and obstacles give the same
  graph and the same route. After a pure refactor, the printed edge
  list, route and distances must not change. Treat any change as a
  behavioural change to explain or fix.

## Gotchas

- `airports.csv` is indexed by `ICAO` and stores `GeoPointLat` before
  `GeoPointLong`. Swap to `(lon, lat)` when you convert.
- `*.csv` is gitignored. `data/airports.csv` is tracked only because it
  was added before that rule. A new CSV needs `git add -f`, or a
  negation in `.gitignore`.
- `gdf.loc[i]` returns a row (a Series), not a GeoDataFrame, so
  `.plot()` fails on it. Use `gdf.loc[[i]]`.
- geoBoundaries ADM0 geometries are MultiPolygons with many parts
  (islands, exclaves). Simplify with an explicit tolerance and check
  that small parts are not lost or made invalid before you use them as
  obstacles.
- Visibility-edge construction is O(n² · e) for n nodes and e obstacle
  edges. It is fine for a triangle and too slow for raw country
  borders. Simplify the borders first; for many obstacles, use an
  STRtree pre-filter (see `.old/graph_path_optim.ipynb`).
