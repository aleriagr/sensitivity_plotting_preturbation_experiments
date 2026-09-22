# Sensitivity mapping from perturbation screens

A small Jupyter notebook that turns a one-variable-at-a-time perturbation screen into a radar
(spider) plot of each response's **% change relative to a reference condition**. It's a pre-BO
robustness check: before committing to a Bayesian optimization campaign, see which conditions the
reaction is fragile to (large swings → tight control needed) versus robust to (small swings → safe
to leave loose), and whether a perturbation that hurts the main response also helps or hurts side
products.

## What it does

- Reads a perturbation screen (one row per condition, one column per response).
- Computes, for each chosen response, the `% sensitivity` of every condition against the
  `Reference` row (plus the raw delta and its absolute magnitude).
- Ranks conditions by absolute sensitivity (most disruptive first).
- Draws a Plotly radar chart with concentric % rings and one polygon per response, and optionally
  writes it to HTML and PNG.

## Repository contents

```
Sensitivity_Mapping_1.0/
├── sensitivity_mapping_plotly.ipynb   # the notebook (run this)
├── sensitivity_screen.xlsx            # example perturbation screen (multiple sheets)
├── README.md
├── requirements.txt                   # pip dependencies
├── environment.yml                    # conda environment
├── LICENSE                            # MIT (fill in the copyright holder)
└── .gitignore
```

## Installation

**conda (recommended):**

```bash
conda env create -f environment.yml
conda activate sensitivity-mapping
jupyter lab
```

**pip / venv:**

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

**`kaleido` is required for PNG export.** The notebook writes both an HTML and a PNG by default; the
PNG step (`fig.write_image`) needs `kaleido`. If you don't want the PNG, set `save_png = False` in the
User Inputs cell and `kaleido` becomes unnecessary.

## Input format

An Excel workbook with, on the sheet you select:

- a **`Condition`** column (one row per condition), including one row named by
  `reference_condition` (default `"Reference"`);
- one **numeric column per response** you want to plot (defaults: `Yield F`, `Yield byproducts`).

`% sensitivity` for a condition = `(response − reference_response) / reference_response × 100`, so
the reference row is the 0% baseline and is dropped before plotting.

The bundled `sensitivity_screen.xlsx` has several sheets; the notebook uses **`sheet_name = 1`**
(the second sheet, 0-indexed — `Sheet1_C2`, with columns `Condition`, `Yield F`,
`Yield byproducts`). Point `sheet_name` at a different index or a sheet name to use another layout.

Optional: if the sheet has replicate columns `Yield_1` and `Yield_2`, the notebook averages them
into `Yield` and records the spread as `Yield_std`. This is a no-op if those columns aren't present.

## Usage

Open the notebook and edit the **User Inputs** cell:

- `excel_file`, `sheet_name` - the workbook and which sheet to read.
- `response_columns`, `response_labels`, `response_colors` - which response column(s) to plot, their
  legend names, and line colors. A commented block shows how to plot a single response instead of two.
- `reference_condition` - the row every other row is measured against (default `"Reference"`).
- `ring_levels`, `ring_colors` - the concentric background rings (the % scale) and their fills,
  largest (−100%) first.
- `save_html`, `save_png`, `output_html`, `output_png` - whether to write the figure to disk and
  under what filenames.

Then run all cells. The radar renders inline and, if enabled, is written to `sensitivity_radar.html`
and `sensitivity_radar.png`.

## Notes

- Plotly's polar axis can't take negative radii, so the chart maps % to radius with `to_r`: 0% sits
  on a middle ring and −100% at the center. Ring labels show the true % values.
- Generated `sensitivity_radar.*` (and `radar.*`) files are git-ignored - they regenerate on run.

## License

Released under the MIT License - see [LICENSE](LICENSE). Fill in the copyright-holder line in that
file (year and name/institution) before submitting.
