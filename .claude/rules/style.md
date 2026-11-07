---
paths:
  - "**/*.py"
  - "**/*.ipynb"
  - "**/.gitignore"
  - "**/.gitattributes"
---

# Style guide

Apply these rules to new and changed code. In notebooks they apply to
code cells. Do not reformat existing cells in bulk only for style: that
makes noise in the notebook diff. Bring a cell into line when you change
it for another reason.

- **Imports:** three groups separated by blank lines — standard
  library, third-party, local. Within each group: bare `import`
  statements first, then `from … import` statements. Within each of
  those two sub-groups, sort by ascending length of the module name:

  ```python
  import os
  import json

  import numpy as np
  import pandas as pd
  import networkx as nx
  import geopandas as gpd
  import plotly.graph_objects as go
  from shapely.geometry import Polygon

  from src.sphere import to_unit
  ```

- **Constants:** four tiers separated by blank lines, sorted by
  ascending length of the constant name within each tier:

  1. Environment variables (`os.environ.*`)
  2. Scalar constants — plain strings, URLs, numbers
  3. Path constants — `Path(...)` literals
  4. Derived constants — anything that references a constant above

  ```python
  EPS = 1e-9
  R_EARTH = 6371.0088  # mean Earth radius, km

  DATA_DIR = Path('data')

  AIRPORTS_CSV = DATA_DIR / 'airports.csv'
  ```

- **`.gitignore` ordering:** three tiers separated by blank lines,
  sorted by ascending pattern length within each tier; a negation
  (`!…`) sits directly under the pattern it carves out. Rely on git's
  match-at-any-depth semantics — don't add `**/` or bare-directory
  duplicates of a pattern that already matches everywhere.

  1. Dotfile / dot-directory patterns (leading `.`) and their negations
  2. File-glob patterns (`*.ext`)
  3. Directory patterns (trailing `/`)

- **`.gitattributes` ordering:** commented groups separated by blank
  lines — notebook filter, LFS by path, LFS by extension (geodata, then
  binaries) — sorted by ascending pattern length within each group.
  Every LFS line is `<pattern> filter=lfs diff=lfs merge=lfs -text`.

- **Quotes:** single quotes by default (`'value'`); double quotes only
  when the string itself contains a single quote.

- **Line length:** keep lines ≤ 79 characters (PEP 8).

- **Argument nesting:** keep a call or signature on one line when it is
  short and simple. Break it to **one argument per line** — hanging
  indentation, closing parenthesis on its own line aligned with the
  opening statement, and a trailing comma after the last argument —
  whenever any of these hold:

  1. it has **two or more keyword arguments** (for a `def`, two or more
     parameters with default values);
  2. **any argument is a compound expression** — an inline calculation,
     a function/method call, a comprehension, or a conditional
     expression (a bare name, literal, or simple keyword default such
     as `n=64` does not count; a keyword argument whose *value* is a
     call or calculation does);
  3. it does not fit on one line within the line-length limit.

  A call with a **single** keyword argument that fits on one line and
  has no compound argument stays inline
  (`nx.path_weight(G, path, weight='weight')`) — the one exemption.
  All-positional calls likewise stay inline unless (2) or (3) forces a
  break. **Once nested, every argument goes on its own line** —
  positional ones included, never two to a line. When a compound
  argument would itself wrap across several lines, extract it to a
  named local instead of nesting deeply.

  ```python
  path = nx.astar_path(                         # two kwargs → nest
      G,
      'UAAA',
      'KJFK',
      heuristic=heuristic,
      weight='weight',
  )

  lam, phi = np.radians(lon), np.radians(lat)
  xyz = np.array(                               # compound args → nest
      [
          np.cos(phi) * np.cos(lam),
          np.cos(phi) * np.sin(lam),
          np.sin(phi),
      ],
  )
  ```

- **String interpolation:** prefer f-strings; fall back to `.format()`
  or `%` only when f-strings are not applicable.

- **Text encoding:** set UTF-8 explicitly wherever the API exposes an
  encoding option — `open()`, `read_text()`/`write_text()`, pandas
  `read_csv`/`to_csv`, and so on. Binary mode takes no `encoding`;
  BOM-prefixed inputs need `encoding='utf-8-sig'` on read; a source in
  another encoding (for example `cp1251`) is read in that encoding and
  written back as UTF-8.

- **Blank lines around compound statements:** separate a `for`,
  `while`, `if`/`elif`/`else` block from surrounding statements with a
  blank line before its header and after its last line — except when
  the block is the first statement in its enclosing block (no leading
  blank line needed) or the last (no trailing one needed):

  ```python
  G = nx.Graph()
  m = len(ring_names)

  for i in range(m):
      u, v = ring_names[i], ring_names[(i + 1) % m]
      G.add_edge(u, v, weight=gcd_km(nodes[u], nodes[v]))

  names = list(nodes)
  ```

- **Progress bars:** a notebook concern. Library code (future `src/`)
  never imports `tqdm`. A long loop that is worth watching takes an
  injected `progress` factory that defaults to `iter` and wraps its loop
  with it (`for x in progress(items):`). In notebooks, use
  `from tqdm.auto import tqdm` for long loops only — for example the
  O(n²) visibility-edge build over real borders — not for short loops
  or comprehensions.
