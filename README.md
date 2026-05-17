# BiasMix-Finance: Post-Generation KYC Guardrails for LLM Portfolio Advice — Supplement

This supplement contains the code notebooks and data artifacts needed to reproduce the **tables/figures reported in the paper** and (optionally) re-run the end-to-end pipeline including **LLM generation**.

There are **two reproducibility modes**:

1. **Exact reproduction (recommended for reviewers)**: Reproduce all reported tables/figures **exactly** using the provided `results.jsonl` (no LLM calls).
2. **End-to-end rerun (optional)**: Re-run scenario prompting + parsing + projection by calling an LLM (requires API keys). Results may differ due to LLM nondeterminism.

---

## Contents

- `notebooks/BiasMix_Finance_Post_Generation_KYC_Guardrails.ipynb` — main reproducibility notebook
- `notebooks/eda_universe_diagnostics.ipynb` — optional EDA notebook (universe/Σ diagnostics)
- `data/scenarios_full.jsonl` — scenario definitions
- `data/assets.csv` — ETF universe metadata
- `data/sigma.npy` — covariance matrix Σ
- `outputs/results.jsonl` — **canonical run outputs used for paper evaluation** (first-pass + final weights, parse failures, metrics)

> **Note:** Exact reproduction of paper numbers requires `outputs/results.jsonl`. End-to-end rerun requires LLM access and will not necessarily match the paper’s numbers exactly.

---

## Quickstart (Google Colab)

1. Open `notebooks/BiasMix_Finance_Post_Generation_KYC_Guardrails.ipynb` in Colab.
2. Upload the required files into the Colab runtime (or mount Google Drive and update paths):
   - `data/scenarios_full.jsonl`
   - `outputs/results.jsonl` *(for exact reproduction)*
   - `data/assets.csv`, `data/sigma.npy` *(needed for projection / ablations / some plots)*

In the notebook, the default expectation is that these files appear under `/content/` (you can edit the “Load Data” cell to change paths).

---

# A) Exact reproduction (NO LLM calls) — recommended for reviewers

This path reproduces the **paper’s reported metrics/tables/figures exactly**, using the provided `outputs/results.jsonl`.

### Run “Load Data”
Run the notebook section:

- **Load Data** (Cells **2–4**)

This sets up paths and loads scenario metadata and artifacts.

### A2 — Run evaluation and statistics
Run the notebook section:

- **Statistical Framework** (Cells **38–55**)

This will:
- load `results.jsonl`
- compute violation rates and summaries
- compute **Wilson 95% CIs** for binomial rates
- compute distance metrics (e.g., correction distance **D**) with **bootstrap 95% CIs**
- run paired **Wilcoxon** tests and apply **BH-FDR**
- write paper tables as CSV into:
  - `/content/tables/`

**Generated CSVs (typical):**
- `tables/violation_overall.csv`
- `tables/violations_summary.csv`
- `tables/distance_D_mean_CI.csv`
- `tables/delta_sigma_CI.csv`
- `tables/delta_waer_CI.csv`
- `tables/delta_hhi_CI.csv`
- `tables/wilcoxon_model_pairs_fdr.csv`

**Note about Cell 41:** If the cell is written as `pip install ...` without `!`, Colab may error. Change it to `!pip install ...` and re-run that cell.

### A3 — Generate paper figures (PDF)
Run:

- **Cell 60**

This writes the main paper plots (PDF) and may download them via `google.colab.files.download(...)`:
- `violation_rate.pdf`
- `correction_distance.pdf`
- `violation_severity.pdf`

If you do not want downloads, comment out the `files.download(...)` lines.

### A4 — Optional paper-support analyses
These are not required for the main tables/figures, but are included for completeness.

- **Stage breakdown summaries** (Cell **61**)
- **Sector-level before/after plots** (Cells **62–63**)  
  Outputs in `/content/figs/` (requires sector sums in `results.jsonl`)
- **Solver ablation table on test split** (Cells **64–66**)  
  Writes `/content/ablation_solver_table.csv` (Cell **65** installs solver deps: `!pip -q install cvxpy osqp`)

---

# B) End-to-end rerun (WITH LLM calls) — optional

This reruns prompting + parsing + projection to produce a new `results.jsonl`.

**LLM nondeterminism:** Exact numbers may differ from the paper, even with the same prompts/config.

## B1 — Provide API keys (Colab Secrets)
The notebook reads API keys from **Colab Secrets** (`google.colab.userdata.get(...)`).

In Colab:
- Click the **key icon (Secrets)** in the left sidebar
- Add secrets as needed:

### Gemini (primary)
- `GOOGLE_API_KEY`

### OpenAI (secondary closed)
- `OPENAI_API_KEY`

### OpenAI-compatible provider (secondary open; e.g., hosted Llama)
- `OPENAI_COMPAT_API_KEY`
- `OPENAI_COMPAT_BASE_URL`

Reviewers can also modify the model/provider configuration in:
- **Model configuration** (Cell **22**) — edit the `MODELS` dict to use alternative providers/models.

## B2 — Run pipeline to generate a fresh results file
Run these sections in order:

1. **Load Data** (Cells **2–4**)
2. **Helper Functions** (Cell **6**)
3. **Validator** (Cell **8**)
4. **Nearest-feasible projector** (Cell **11**)
5. *(Optional)* **Smoke test** (Cell **14**)
6. **LLM Interface & Runner** (Cells **16–35**)

### Full grid run (expensive)
- **Cell 35** runs the full grid:
  - `batch_run_grid(split="train")`
  - `batch_run_grid(split="dev")`
  - `batch_run_grid(split="test")`

This sweeps:
- `model_keys=("primary","secondary_closed","secondary_open")`
- `modes=("direct","critique","sc")`

## B3 — Evaluate the new run
After generating `results.jsonl`, run the same evaluation cells as in Mode A:

- **Statistical Framework** (Cells **38–55**) → tables to `/content/tables/`
- **Cell 60** → figures as PDFs

---

## Output locations

- Tables (CSV): `/content/tables/`
- Main figures (PDF): written in the notebook working directory (typically `/content/`)
- Sector plots (optional): `/content/figs/`
- Ablation table (optional): `/content/ablation_solver_table.csv`

---

## Reproducibility statement

- **Deterministic** given `results.jsonl`: parsing → constraint checks → projection verification → metrics → CIs/tests → plots.
- **Stochastic** when re-running LLM calls: model sampling and provider/version changes can affect outputs.

To reproduce the paper’s reported numbers exactly, use **Mode A** with the provided `outputs/results.jsonl`.

---

## Contact / Notes

If any cell numbers shift due to notebook edits, use the **section headings** referenced above:
- “Load Data”
- “Statistical Framework”
- “LLM Interface & Runner”
- “Solver ablation table on test split”
