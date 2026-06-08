# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## Overview

This is a **GDP dashboard** — a single-page [Streamlit](https://streamlit.io/)
web app that visualizes World Bank GDP data for selected countries and years.
It started life as the official Streamlit "gdp-dashboard" template, so the
codebase is intentionally small and beginner-friendly.

The deployed demo lives at https://gdp-dashboard-template.streamlit.app/.

## Tech stack

- **Python** (developed/tested against 3.11 — see `.devcontainer/devcontainer.json`)
- **Streamlit** — UI framework and dev server
- **pandas** — CSV loading and data reshaping

Dependencies are unpinned in `requirements.txt` (just `streamlit` and `pandas`).

## Repository layout

```
.
├── streamlit_app.py          # The entire application (single file)
├── data/
│   └── gdp_data.csv          # World Bank GDP data, wide format (one column per year, 1960–2022)
├── requirements.txt          # Python dependencies (streamlit, pandas)
├── README.md                 # User-facing run instructions
├── LICENSE                   # Apache 2.0
├── .devcontainer/
│   └── devcontainer.json     # Codespaces / dev container config; auto-runs the app on port 8501
└── .github/
    └── CODEOWNERS            # @streamlit/community-cloud owns everything
```

There is **no** test suite, linter config, CI workflow, or build step. The app
runs directly from source.

## Running the app

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Launch the dev server (opens http://localhost:8501)
streamlit run streamlit_app.py
```

In a dev container / Codespace this happens automatically: `updateContentCommand`
installs requirements and `postAttachCommand` starts `streamlit run` on port 8501.

There is no separate "build" or "production" command — Streamlit Community Cloud
runs the same `streamlit run streamlit_app.py` entry point.

## How the app works

Everything is in `streamlit_app.py`, executed top-to-bottom on every interaction
(Streamlit's re-run model). The flow:

1. **`st.set_page_config(...)`** — sets the browser tab title/icon. Must be the
   first Streamlit call.
2. **`get_gdp_data()`** — decorated with `@st.cache_data` so the CSV is only read
   and reshaped once. It:
   - Reads `data/gdp_data.csv` (path resolved relative to the script via
     `Path(__file__).parent`).
   - Uses `pandas.melt` to pivot the wide year-columns (`"1960"`…`"2022"`) into
     tidy long format with `Year` and `GDP` columns, keyed by `Country Code`.
   - Converts `Year` from string to numeric.
   - Hard-codes `MIN_YEAR = 1960` and `MAX_YEAR = 2022` — these must match the
     columns present in the CSV.
3. **Page rendering** — a year-range `st.slider`, a country `st.multiselect`
   (defaults: `DEU, FRA, GBR, BRA, MEX, JPN`), an `st.line_chart` of GDP over
   time, and a row of `st.metric` cards showing each country's GDP and growth
   multiple for the selected end year.

Note the use of bare triple-quoted strings (e.g. `'''# :earth_americas: ...'''`)
and empty `''` lines as a shorthand for `st.markdown` / vertical spacing — this
is idiomatic Streamlit "magic" in this template, not dead code.

## Conventions & guidance for changes

- **Keep it single-file.** The whole app is `streamlit_app.py`. Only split it
  out if a change genuinely warrants it; otherwise match the existing inline,
  comment-heavy style.
- **Data assumptions.** GDP values are in raw dollars; the metric cards divide by
  1e9 to display billions (`B`). The data is sparse — many country/year cells are
  `NaN`, so guard with `math.isnan(...)` as the existing growth calculation does.
- **Changing the year range** requires updating both the CSV columns and the
  `MIN_YEAR` / `MAX_YEAR` constants in `get_gdp_data()`.
- **Caching.** Anything that loads or transforms data should stay behind
  `@st.cache_data` (add a `ttl=` if the source ever becomes a live endpoint, as
  the docstring notes).
- **The data is wide on disk, long in memory.** Don't assume the CSV is already
  tidy; the `melt` step is what makes it chart-friendly.

## Git workflow

- Default branch is `main`.
- Commit with clear, descriptive messages.
- Do **not** open a pull request unless explicitly asked.
