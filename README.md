# AI-Powered Intrusion Detection System using Random Forest and CIC-IDS2017

A proof-of-concept machine-learning Network Intrusion Detection System (NIDS)
built for the *Information Security* course assignment (CLO 4). The project
follows a real-world scenario: **SecureNet Corp.** wants to know whether ML
can augment its existing signature-based NIDS by automatically classifying
network traffic as **normal** or **malicious**.

## Objective

Design, train, and evaluate a Random Forest classifier that flags malicious
network flows (DDoS, PortScan, Brute-Force, Web Attacks, Bot, Infiltration)
in traffic modeled on the CIC-IDS2017 dataset, and interpret the results
from a security-operations point of view (false positives vs. false
negatives, deployment posture, future improvements).

## Repository Structure

```
CLO4-IDS-ML-Solution/
├── README.md
├── requirements.txt              Python dependencies for the notebook
├── notebook/
│   └── AI_Powered_IDS_RandomForest_CICIDS2017.ipynb   Full, executed pipeline
├── data/
│   └── network_traffic_raw.csv   Sample CIC-IDS2017-style dataset
├── figures/                      Charts used in the notebook and report
├── report/
│   └── Report_....docx           MS Word project report (6 pages)
├── dashboard/
│   ├── app.py                    Streamlit interactive dashboard
│   ├── requirements.txt
│   └── html/
│       ├── ids_dashboard.html    Offline dashboard, zero install required
│       ├── model_data.json       Exported model + data (source of truth)
│       ├── index_template.html   Page markup/CSS/JS (edit this, not the html above)
│       └── build_dashboard.py    Rebuilds ids_dashboard.html from the two files above
└── src/
    ├── generate_data.py          Regenerates data/network_traffic_raw.csv
    ├── export_for_html.py        Regenerates dashboard/html/model_data.json
    └── create_report.js          Regenerates the Word report from figures/
```

Everything under `src/` and `dashboard/html/build_dashboard.py` +
`index_template.html` is source: the notebook, dataset, report, and
`ids_dashboard.html` are the actual graded deliverables and already
reflect their output.

## Dataset Setup Instructions

**CIC-IDS2017** (Canadian Institute for Cybersecurity, University of New
Brunswick) is a labeled network-traffic dataset combining benign traffic
with common modern attacks (DDoS, PortScan, Brute-Force, Web Attacks, Bot,
Infiltration), exported as CICFlowMeter bidirectional flow records. It was
chosen because it reflects realistic, contemporary attack traffic relevant
to a web-facing organization, unlike older datasets such as KDD'99/NSL-KDD.

**Option A, bundled sample data (default, no download needed):**
`data/network_traffic_raw.csv` is a synthetic dataset generated to match
CIC-IDS2017's schema, class taxonomy, and known data-quality quirks
(`inf` values when `Flow Duration = 0`, missing sensor fields, heavy class
imbalance toward benign traffic). It lets everything in this repo run
end-to-end out of the box. Regenerate it any time with:

```bash
python3 src/generate_data.py   # run from the repo root
```

**Option B, using the real CIC-IDS2017 dataset:**
1. Download the CSV files from the official source:
   https://www.unb.ca/cic/datasets/ids-2017.html
2. Concatenate the daily CSVs (or use a single day, e.g.
   `Wednesday-workingHours.pcap_ISCX.csv`) into one file.
3. Strip leading/trailing whitespace from column headers (the notebook
   already does this with `df.columns = [c.strip() for c in df.columns]`).
4. Save the result as `data/network_traffic_raw.csv`, keeping the `Label`
   column with class names (e.g. `BENIGN`, `DDoS`, `PortScan`, ...).
5. Re-run the notebook top to bottom; no code changes are required.

## How to Run the Code

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd CLO4-IDS-ML-Solution

# 2. Create an environment and install dependencies
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# 3. Launch Jupyter and run the notebook top to bottom
jupyter notebook notebook/AI_Powered_IDS_RandomForest_CICIDS2017.ipynb
```

Or run non-interactively end-to-end:

```bash
cd notebook
jupyter nbconvert --to notebook --execute --inplace \
    AI_Powered_IDS_RandomForest_CICIDS2017.ipynb
```

## Summary of Results

Random Forest (binary classification: BENIGN vs. ATTACK) on a held-out 25%
test split:

| Metric | Score |
|---|---|
| Accuracy | 99.13% |
| Precision | 98.83% |
| Recall | 97.61% |
| F1 Score | 98.21% |
| ROC AUC | 98.72% |

The confusion matrix, ROC curve, and feature-importance chart are in the
notebook (Section 9) and in `figures/`, along with a security-focused
discussion of what the false-positive and false-negative counts mean
operationally (Section 10 of the notebook, and the report).

## Running the Local Streamlit Dashboard

`dashboard/app.py` mirrors the notebook pipeline in an interactive local
web app, with pages for Dashboard, Data Explorer, Model Training,
Visualizations, Predictions, Model History, and Settings.

```bash
# From the repository root
pip install -r dashboard/requirements.txt
streamlit run dashboard/app.py
```

This opens the dashboard at `http://localhost:8501` in your browser. It runs
entirely on your machine: data, trained models, and predictions all stay
local and are cleared when the browser session ends or you click
"New Session". From there you can:

- **Dashboard** -- see dataset and active-model summary metrics at a glance.
- **Data Explorer** -- filter and inspect the raw flow data, check for
  missing/infinite values.
- **Model Training** -- pick Random Forest, Decision Tree, or Logistic
  Regression, tune hyperparameters, and train on the fly.
- **Visualizations** -- confusion matrix, ROC curve, feature importance, and
  (once more than one model is trained) an accuracy comparison chart.
- **Predictions** -- classify a single flow via manual input, or upload a CSV
  for batch predictions with a downloadable results file.
- **Model History** -- review every model trained in the current session.
- **Settings** -- swap in your own CSV (e.g., the real CIC-IDS2017 data) or
  reset the session.

## Offline HTML Dashboard (No Install Required)

If you would rather not install Python packages at all,
`dashboard/html/ids_dashboard.html` is a single, self-contained HTML file
with the same Dashboard / Data Explorer / Model Performance /
Visualizations / Predictions layout. It requires no server, no pip install,
no Python, and no internet connection at all (Chart.js and PapaParse are
inlined directly into the file, not loaded from a CDN):

```bash
# Just open the file directly in a browser
open dashboard/html/ids_dashboard.html      # macOS
start dashboard\html\ids_dashboard.html     # Windows
xdg-open dashboard/html/ids_dashboard.html  # Linux
```

The actual trained 200-tree Random Forest is exported into the page as raw
decision-tree arrays, so predictions run for real in your browser's
JavaScript engine and match the Python model's output exactly (verified to
agree on 100% of test-set predictions). To rebuild it after changing the
model or the page itself:

```bash
python3 src/export_for_html.py           # regenerates dashboard/html/model_data.json
cd dashboard/html
npm install chart.js papaparse --no-save # only needed once
python3 build_dashboard.py               # regenerates ids_dashboard.html
```

This version does not support retraining with different hyperparameters
(use the notebook or the Streamlit dashboard for that) but covers dataset
exploration, full evaluation results, and live predictions.

## Methodology Overview

1. **Preprocessing** -- infinite values converted to `NaN` and median-imputed,
   `Protocol` label-encoded, all numeric features standardized with
   `StandardScaler`.
2. **EDA** -- class distribution, benign/attack split, protocol mix, flow
   duration by class, feature correlation heatmap.
3. **Model** -- `RandomForestClassifier` (`n_estimators=200`, `max_depth=18`,
   `class_weight="balanced"`), stratified 75/25 train/test split.
4. **Evaluation** -- Accuracy, Precision, Recall, F1, ROC-AUC, confusion
   matrix, feature importances, and a security-operations interpretation of
   the error rates.

## Tech Stack

`pandas` · `numpy` · `matplotlib` · `seaborn` · `scikit-learn` · `Jupyter` · `Streamlit` · `Chart.js` · `PapaParse`

## Author

Course: Information Security -- Assignment 1 (CLO 4)
