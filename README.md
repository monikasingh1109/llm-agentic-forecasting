# Agentic Forecasting Analyst

An evidence-first forecasting project built around Walmart weekly store sales. It combines exploratory data analysis, time-series and machine-learning baselines, model selection, forecast quality checks, and auditable analyst explanations.

> **Core principle:** the LLM may suggest candidates or help explain results, but executable code evaluates models, validates claims, and determines the winner. Decisions are based on backtesting evidence—not on the LLM's opinion.

## Project overview

The project is represented by two notebooks:

| Notebook | Purpose |
|---|---|
| `Walmart_Simple_EDA_ML_Agent.ipynb` | Compact, presentation-friendly workflow: EDA, forecasting baselines, rolling-origin validation, and a model-selector agent. |
| `Walmart Agentic Forecasting.ipynb` | Extended experiment: per-store series profiling, model selection versus fixed policies, reasoning-trace audits, fault injection, and an analyst memo. |

## What it does

### 1. Exploratory data analysis

The notebooks investigate:

- Data quality, missing values, duplicate store-week records, and weekly date gaps.
- Weekly sales distribution and skew.
- Yearly seasonality and lag-52 autocorrelation.
- Holiday effects, including a pre-Christmas period that may not be captured by the source holiday flag.
- Differences in average sales between stores.
- Within-store relationships between sales and temperature, fuel price, CPI, and unemployment.

EDA findings inform the modelling choices rather than being treated as a separate, purely descriptive step.

### 2. Forecasting models

The simple notebook evaluates a small menu of forecasting approaches:

- **Seasonal Naive** — uses sales from the same week a year earlier.
- **ETS** — exponential smoothing with trend and yearly seasonality, fitted per store.
- **LightGBM** — a pooled gradient-boosting model across stores.
- **Hybrid** — an average of ETS and LightGBM when both are eligible.

The forecast horizon is **12 weeks**. The LightGBM formulation predicts the log ratio of current sales to sales 52 weeks earlier, then converts that prediction back to sales units. Features include calendar and holiday indicators plus lagged and rolling sales ratios.

### 3. Evidence-based model selection

The simple workflow profiles the dataset, shortlists eligible models using rules by default, runs a model tournament, and chooses the model with the lowest validation weighted mean absolute error (WMAE).

The extended notebook makes the comparison more rigorous by selecting a model per store using earlier folds and evaluating it on a later fold. It compares the agent with fixed model policies, a best-single-model policy, and an oracle reference (best model in hindsight, for diagnostic comparison only).

### 4. Leakage checks and validation

- Rolling-origin backtesting trains on earlier observations and evaluates the following 12 weeks.
- The simple notebook uses four rolling folds; folds 1–3 select the model and fold 4 is held out as a final check.
- A feature leakage assertion checks that forecast-week features do not change when future sales are hidden.
- The extended notebook uses past folds to select models and later folds to evaluate those choices.

### 5. Auditable explanations and QA experiments

The extended notebook also includes:

- **Profile agent:** calculates series characteristics such as frequency, missingness, stationarity, seasonality, outliers, and holiday effect.
- **Candidate proposal and selection:** proposes models from an allowed menu and documents why candidates qualify.
- **Reasoning-trace audit:** recomputes profile values and scores, verifies selection logic, and checks generated explanation text against structured claims.
- **Fault injection:** deliberately corrupts claims to measure whether the audit catches the injected faults.
- **Analyst memo:** creates a summary from computed facts; optional LLM-written text is checked against the facts dictionary.
- **Planner test / feature discovery / QA guard:** the extended notebook describes additional experiments that compare agent-assisted suggestions or checks with traditional approaches.

These checks establish whether a trace is consistent with the supplied data and rules; they do **not** prove that the modelling rules themselves are optimal.

## Repository layout

A suggested repository layout is:

```text
.
├── README.md
├── Walmart_Simple_EDA_ML_Agent.ipynb
├── Walmart Agentic Forecasting.ipynb
├── Walmart.csv                         # add locally; do not commit private/licensed data
└── ml_outputs/
    ├── cv_predictions.csv
    ├── forecast_next_12_weeks.csv
    ├── leaderboard_validation.csv
    └── holdout_fold4.csv
```

The notebooks create the `ml_outputs/` directory and write CSV outputs there. The exact files produced depend on which notebook and cells you run.

## Dataset

The notebooks are written for a Walmart store-level weekly sales dataset with fields such as:

- `Store`
- `Date`
- `Weekly_Sales`
- `Holiday_Flag`
- `Temperature`
- `Fuel_Price`
- `CPI`
- `Unemployment`

The code currently expects a local CSV file. In the **simple notebook**, update the hard-coded Windows file path to the path where your dataset is stored. In the **extended notebook**, check the data-loading cell and make sure it points to the correct CSV. Keep the dataset in the project folder or configure a path appropriate to your environment.

Do not publish data unless you have the right to redistribute it. If the dataset is not included, users must obtain it separately and place it at the configured path.

## Setup

Python 3.10 or newer is a reasonable starting point; use the Python version supported by your installed dependencies.

Create and activate a virtual environment (optional but recommended):

```bash
python -m venv .venv
```

Activate it:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the libraries used in the notebooks:

```bash
pip install jupyter numpy pandas matplotlib scipy statsmodels lightgbm
```

For the optional Gemini integration, install the Google Gen AI SDK:

```bash
pip install google-genai
```

Launch Jupyter:

```bash
jupyter notebook
```

Open either notebook and run the cells in order. Some cells use notebook-specific commands such as `%pip`; run them in Jupyter/IPython.

## Optional LLM integration

The rule-based workflow works without an API key. If you want to enable the optional Gemini-powered candidate proposal or memo generation, set `GEMINI_API_KEY` in your environment before running the relevant cells.

```bash
# macOS / Linux
export GEMINI_API_KEY="your_api_key"

# Windows PowerShell
$env:GEMINI_API_KEY="your_api_key"
```

Never commit API keys to GitHub. Prefer environment variables or a local `.env` file excluded by `.gitignore`. The extended notebook includes a cell that prompts for a key; avoid printing or saving the key in notebook output.

The LLM's output is restricted to an allowed model menu and validated by code. It does not directly choose the winning model.

## Run the workflow

1. Download or otherwise obtain the dataset and configure the CSV path in the notebook.
2. Install the dependencies.
3. Open `Walmart_Simple_EDA_ML_Agent.ipynb` for the compact EDA → ML → model-selection workflow, or open `Walmart Agentic Forecasting.ipynb` for the extended experiments.
4. Run cells from top to bottom.
5. Review the validation leaderboard and held-out fold before interpreting the final forecast.
6. Inspect the generated CSV files and analyst memo.

## Generated outputs

The simple workflow writes files under `ml_outputs/`:

| File | Contents |
|---|---|
| `cv_predictions.csv` | Store-week predictions and errors from cross-validation. |
| `forecast_next_12_weeks.csv` | Forecasts for each store across the next 12 weeks, including an empirical 80% range. |
| `leaderboard_validation.csv` | Model comparison on the validation folds. |
| `holdout_fold4.csv` | Model comparison on the held-out fourth fold. |

The extended notebook also produces experiment results and a memo as defined by its output cells. Check the notebook's final cells for the exact filenames used in your run.

## Evaluation and interpretation

- **WMAE:** weighted mean absolute error; lower is better. Holiday observations receive extra weight in the simple model's LightGBM training and evaluation logic.
- **Rolling-origin validation:** tests whether a model or selection policy generalizes to later periods.
- **Diebold–Mariano-style comparison:** the simple notebook includes a statistical comparison of forecast errors.
- **Store-level results:** the extended notebook evaluates whether per-store model selection improves on fixed policies.

Do not assume that the agent wins. The extended notebook explicitly emphasizes reporting the measured result honestly: a per-store selector can perform worse than a strong single policy when the validation window is short and model rankings are noisy. Re-run the notebooks and report the results generated by your own dataset and environment; do not treat example results in markdown as guaranteed outcomes.

## Limitations

- The project is currently notebook-oriented rather than a packaged command-line application or deployed forecasting service.
- The workflow expects a Walmart-style weekly dataset with the columns described above.
- Model eligibility and selection depend on hand-designed rules and the available validation folds.
- Short evaluation windows can lead to noisy rankings and overfitting in model selection.
- Synthetic fault-injection tests cover specified error types; they do not cover every possible failure or prove that a decision is substantively correct.
- Forecast intervals are derived from historical backtest ratios and should be interpreted as empirical ranges, not guaranteed coverage.
- External factors such as weather and macroeconomic variables may be unavailable for the full forecast horizon and are not necessarily used as predictive features.

## Future improvements

- Test on additional, more heterogeneous time-series datasets.
- Add longer validation histories and multiple forecast horizons.
- Compare against additional approaches such as ARIMA, Prophet, TBATS, and AutoML/meta-learning selectors.
- Evaluate feature discovery and ensemble weighting using pre-defined validation protocols.
- Expand the reasoning audit to more classes of plausible but incorrect claims.
- Add automated tests, configuration files, reusable Python modules, and a reproducible pipeline outside notebooks.

## Reproducibility checklist

- [ ] Configure the dataset path and verify the input columns.
- [ ] Record Python and package versions.
- [ ] Keep API keys and private data out of version control.
- [ ] Run all notebook cells from a clean kernel.
- [ ] Review leakage assertions and validation/holdout metrics.
- [ ] Report the actual outputs from your run, including cases where the agent does not improve accuracy.

## License and data attribution

Add a license for your own code before publishing, and follow the license and attribution requirements of the dataset source. This README does not grant rights to redistribute the Walmart dataset.

---

**Project theme:** Agentic Forecasting Analyst — trustworthy forecasting through measurable model selection, leakage-aware validation, and auditable explanations.
