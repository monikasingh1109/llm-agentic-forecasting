LLM Agentic Forecasting: Walmart Weekly
Sales
Does an LLM-based multi-agent system forecast retail sales better than traditional
methods, or at least explain them better? This project tests that question on the Walmart
45-store weekly sales dataset.
Status: 🚧 In progress. Results below are placeholders and will be filled in as experiments
are completed.
Table of Contents
Objective
Hypotheses
Dataset
Approach
Multi-Agent Architecture
Evaluation Methodology
Results
Project Structure
Getting Started
Roadmap
Limitations
License
Objective
Compare traditional forecasting methods against LLM-based approaches, including a
three-agent pipeline (Data Agent, Forecast Agent, Insight Agent), for weekly sales
forecasting, and measure whether the LLM approach adds measurable value in accuracy,
explainability, or both.
Hypotheses
Defined before running experiments to keep the evaluation honest:
#
Hypothesis
Supported if...
H1 LLM-only forecasting is competitive with
traditional models
H2A hybrid (statistical forecast + LLM agent
contextual adjustment) beats either alone
H3The Insight Agent produces more useful,
grounded explanations than a standard report
Negative results will be reported too.
Dataset
Error is comparable to baselines with
no model training
Lower error than both, especially on
holiday or anomalous weeks
Higher rubric scores, and all cited
numbers trace back to the data
Walmart Sales Dataset of 45 Stores (weekly sales, Feb 2010 to Oct 2012).
Property
Rows
Stores
Value
6,435
45
Weeks per store143 (5 Feb 2010 to 26 Oct 2012)
Missing values None
Date format
Column
Store
Date
dd-mm-yyyy
Store ID (1 to 45)
Week start date
Weekly_Sales 
Target variable
Description
Holiday_Flag 
1 if the week contains a major holiday (10 such weeks in the data)
Temperature 
Regional temperature (°F)
Fuel_Price
CPI
Regional fuel price
Consumer Price Index
Unemployment 
Regional unemployment rate
Place the CSV at 
data/raw/walmart-sales-dataset-of-45stores.csv 
.
Approach
Traditional arm (baselines)
1. Seasonal naive (same week last year)
2. Holt-Winters / ETS
3. SARIMAX (with exogenous variables)
4. LightGBM with lag features
LLM arm
1. LLM-only: recent history is passed as text and the model forecasts the next 12 weeks
2. Multi-agent pipeline (below)
3. 
(Optional)
 Time-series foundation model (e.g. Chronos) as a middle ground
Multi-Agent Architecture
Raw CSV
Data Agent
quality report + features
Data tools
Forecast Agent
forecast + intervals
Statistical model tools
Agent
Responsibility
Insight Agent
Insights &
recommendations
Data Agent Loads and validates data, detects outliers and holiday effects, computes
summaries and features
Forecast
Agent
Runs statistical models via tools, then decides whether and how to adjust
forecasts using context
Insight Agent Explains drivers, risks, and recommended actions, citing the underlying
numbers
Design principle: the LLM never does arithmetic. All calculations are done by Python tools;
the LLM decides which tools to call and interprets the results.
Evaluation Methodology
Rolling-origin backtesting: 12-week-ahead forecasts from multiple cut-off dates (no single
train/test split)
Metrics: MAE, RMSE, sMAPE, MASE
Slices: all weeks / holiday weeks / non-holiday weeks / by store size
Cost metrics: tokens, latency, and API cost per forecast
Significance: paired comparison across stores (Wilcoxon signed-rank test)
Insight quality: rubric-based scoring plus a grounding check (every number cited must
match the data)
Results
To be completed.
Method
Seasonal naive
ETS
SARIMAX
LightGBM
LLM-only
MAERMSEsMAPEMASEHoliday-week MAECost / forecast----
Multi-agent (hybrid)------
Project Structure------------------------
Getting Started
Prerequisites
Python 3.10+
An Anthropic API key (for the LLM arm)
Installation
llm-agentic-forecasting/ ├──
 data/ │
   
├──
 raw/                  # original CSV │
   
└──
 processed/ ├──
 notebooks/ │
   
├──
 01_eda.ipynb │
   
└──
 02_baseline_results.ipynb ├──
 src/ │
   
├──
 data/                 # loader, rolling-origin splits │
   
├──
 baselines/            # naive, ETS, SARIMAX, LightGBM │
   
├──
 agents/               # data_agent, forecast_agent, insight_agent │
   
├──
 tools/                # Python functions the agents call │
   
├──
 evaluation/           # metrics │
   
└──
 run_experiment.py ├──
 prompts/                  # versioned agent prompts ├──
 results/                  # forecasts, metrics, plots ├──
 docs/ │
   
└──
 experiment_design.md ├──
 tests/ ├──
 .env.example ├──
 requirements.txt └──
 README.md
bash
gitclonehttps://github.com/YOUR-USERNAME/llm-agentic-forecasting.git
cdllm-agentic-forecasting
python-mvenvvenv
sourcevenv/bin/activate # Windows: venv\Scripts\activate
pipinstall-rrequirements.txt
Configuration
⚠ Never commit 
.env
. It is listed in 
.gitignore
.
Run
(Commands will be finalized as the code is written.)
Roadmap
 EDA: seasonality, holiday effects, store comparison
 Evaluation harness (rolling-origin folds + metrics)
 Traditional baselines
 LLM-only forecaster (small subset of stores)
 Data Agent
 Forecast Agent (hybrid)
 Insight Agent
 Full 45-store run and significance testing
 Final write-up with charts
Limitations
Short history: under 3 years of data, so only about 2 full yearly cycles are available to learn
seasonality.
Holiday coverage: Thanksgiving and Christmas appear only in 2010 and 2011, so they
cannot be used as a held-out test.
bash
cp.env.example.env
# then edit .env and add your key:
# ANTHROPIC_API_KEY=your-key-here
bash
# Explore the data
jupyternotebooknotebooks/01_eda.ipynb
# Run baselines (example)
python-msrc.run_experiment--methodseasonal_naive--stores12345
# Run the multi-agent pipeline on a small subset first
python-msrc.run_experiment--methodmulti_agent--stores12345
Possible training-data leakage: LLMs may have encountered this public dataset.
Mitigation: store IDs and dates are anonymized/shifted in LLM prompts.
Cost: LLM evaluation across 45 stores and many folds is expensive, so early experiments
use a store subset.
License
MIT. See LICENSE.
Acknowledgements
Dataset: Walmart Store Sales (publicly available on Kaggle).
