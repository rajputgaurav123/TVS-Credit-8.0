# TVS Credit EPIC 8 — Analytics Case Study 2
## Offer amount logic for personal loan

A risk-adjusted offer-amount logic for TVS Credit's personal loan product: customers with
lower predicted risk get a higher loan offer than today's logic, and higher-risk customers get
a lower or equal offer — built and validated end-to-end from the EPIC 8 case study dataset.

## What's in this repo

| File | What it is |
|---|---|
| `EPIC8_01_Analysis_EDA.ipynb` | **Run this first.** Data understanding, quality checks, and exploratory analysis. Standalone — runs on Google Colab by just uploading the Excel file when prompted (or locally, keep the file in the same folder). |
| `EPIC8_02_Model_Offer_Logic.ipynb` | **Run this second (or independently — it's fully self-contained).** The risk model, capacity model, offer-amount logic, evaluation metric, iteration comparison, and the validation deliverable. Also Colab-ready with its own upload prompt. |
| `EPIC8_Dataset.xlsx` | The original case-study data (`Epic_Challenge_Base`, `Data_Dictionary`, `Naming_Conventions` sheets). Upload this when either notebook prompts for it. |
| `EPIC8_Offer_Logic_Dashboard.html` | A self-contained, interactive dashboard (no install needed — open in any browser) summarising the same results across seven tabs: Overview, Data & EDA, Risk model, Capacity model, Offer logic, Validation, and Model notes. |
| `dashboard_data.json` | The exact numbers powering the dashboard's charts, exported for transparency/reuse. |
| `requirements.txt` | Python packages needed to re-run the notebooks locally (not needed on Colab — they're pre-installed there). |

## Running on Google Colab

1. Open [colab.research.google.com](https://colab.research.google.com), upload
   `EPIC8_01_Analysis_EDA.ipynb` (or `EPIC8_02_Model_Offer_Logic.ipynb`).
2. Run all cells (**Runtime → Run all**).
3. When the upload cell runs, a file picker appears — select `EPIC8_Dataset.xlsx`.
4. That's it — no other setup needed. Each notebook is independent; run either one on its own.

## The approach, briefly

1. **Capacity model** — a gradient-boosted regressor trained to reproduce today's offer amount
   from bureau and behavioural features (R² 0.71 on held-out data). Acts as a ceiling.
2. **Risk model** — a tuned, feature-engineered, natively-calibrated classifier trained on
   disbursed loans to predict probability of 30+ DPD in the first 6 months on book. Hyperparameter
   search + 8 engineered ratio features lifted CV AUC to 0.62 and top-decile capture to 1.81x lift
   over random; isotonic calibration brought the Brier score to 0.013, matching the true 1.3% base
   rate almost exactly.
3. **Offer logic** — `proposed offer = capacity × risk multiplier(risk percentile)`, where the
   multiplier runs from 1.30× for the safest customer down to 0.70× for the riskiest, clipped to
   ₹40,000–₹6,00,000 and rounded to the nearest ₹10,000. The improved risk score supports this
   stronger swing while still hitting a perfect composite score.
4. **Evaluation** — a custom 0–100 metric built directly from the case brief's three weighted
   criteria (overall uplift 30%, risk-segment cut 40%, takeup-segment uplift 30%). Stress-tested
   across 12 combinations of tolerance band and risky-segment threshold — stayed at 100 throughout.
5. **Validation** — applied to the held-out `Validation` slice and summarised across quintiles as
   required by the brief (min/max amount, volume, average offer, % Ever-30 in MOB1–6).

## Headline results (Test set)

- Overall average offer: ₹1,78,155 (existing) → ₹1,86,538 (proposed), **+4.7%**
- Riskiest quintile average offer: **cut by ~24%** vs today's logic
- Out-of-sample validation confirms the pattern: the highest proposed-offer quintile shows the
  **lowest** observed 30+ DPD rate

## Model notes

See the dashboard's "Model notes" tab for the full detail, but briefly: the risk model is a rare-
event problem by nature (288 defaults in ~22,000 disbursed loans), so tuning and calibration
improve its usability and reliability rather than turning it into a high-precision underwriting
score. It's trained only on disbursed loans since that's the only population with observed
outcomes — a population-stability check on the score distribution between trained and scored
populations came back low (PSI 0.047), which is reassuring on covariate shift.

## How to reproduce

**On Colab:** see "Running on Google Colab" above — no install step needed.

**Locally:**
```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace EPIC8_01_Analysis_EDA.ipynb
jupyter nbconvert --to notebook --execute --inplace EPIC8_02_Model_Offer_Logic.ipynb
```

Keep `EPIC8_Dataset.xlsx` in the same folder for the local fallback path to work. The dashboard
(`EPIC8_Offer_Logic_Dashboard.html`) needs no build step — just open it in a browser.
