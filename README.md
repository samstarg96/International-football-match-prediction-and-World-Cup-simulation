# 2026 World Cup outcome model and simulation

This project estimates the probabilities of a home win, draw, and away win in men's international football, then uses those probabilities to simulate the 2026 World Cup 10,000 times. It is a **retrospective pre-tournament exercise**: the match features and model training stop on **10 June 2026**, and the tournament team rankings are dated **11 June 2026**. The saved results should not be presented as a forecast published before the tournament.

## What the project does

1. Builds pre-match Elo ratings from historical results, using earlier matches only.
2. Reconstructs 90-minute results where goal-event records allow it, engineers recent-form and match-context features, and compares logistic regression, XGBoost, random forest, and simple probability baselines.
3. Selects a model using mean multiclass log loss across 12 expanding-window validation years (2014–2025), fits it through 10 June 2026, and simulates the 48-team tournament.

The selected five-match-form logistic model has mean validation log loss **0.8844** and mean accuracy **59.2%**. The Elo-only baseline has mean log loss **0.8951** on the same folds. The fitted model predicts 90-minute outcomes; the simulation then models group results and approximates knockout advancement.

![Mean validation log loss by model and baseline](04%20Output/model_and_baseline_comparison.png)

## Saved simulation results

| Team | Share of 10,000 simulated titles |
| --- | ---: |
| Spain | 34.83% |
| Argentina | 18.53% |
| France | 7.05% |
| England | 6.74% |
| Mexico | 6.33% |

These are shares under this model and its tournament assumptions, not measured real-world chances. The complete saved [title shares](04%20Output/monte_carlo/world_cup_2026_win_shares_logit_10000.csv), [stage-reach shares](04%20Output/monte_carlo/world_cup_2026_stage_reach_shares_logit_10000.csv), [model comparison](04%20Output/model_and_baseline_comparison.csv), and [calibration table](04%20Output/selected_model_calibration.csv) are included. See the [calibration chart](04%20Output/selected_model_calibration.png) and [title-share chart](04%20Output/monte_carlo/world_cup_2026_win_shares_logit_10000.png).

## Reproduce the analysis

Python 3.11 was used for this project. From the project folder on Windows:

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements.txt
.\.venv\Scripts\python -m jupyter lab
```

Run these notebooks in order, either with Jupyter started from the project folder or from `02 Scripts`:

1. [`02 Scripts/01_build_elo_ratings.ipynb`](02%20Scripts/01_build_elo_ratings.ipynb) — creates Elo history and current ratings.
2. [`02 Scripts/02_compare_models.ipynb`](02%20Scripts/02_compare_models.ipynb) — runs temporal validation and saves fitted model bundles.
3. [`02 Scripts/03_simulate_world_cup.ipynb`](02%20Scripts/03_simulate_world_cup.ipynb) — simulates the tournament and saves summary tables and charts.

The notebooks create `03 Temp` and `04 Output` as needed. Only the small final tables and charts in `04 Output` are kept in Git. Fitted model files, the large match-level simulation log, other generated files, and exploratory notebook drafts stay local. Re-running the notebooks recreates the outputs. `requirements-lock.txt` records the versions in the original Python environment for reference.

## Data and sources

- Historical match results and goal events are from [martj42/international_results](https://github.com/martj42/international_results), supplied under [CC0 1.0](01%20Input/LICENSE). The included files are a snapshot, not an automatically updated feed.
- [`world_cup_2026_teams.csv`](01%20Input/world_cup_2026_teams.csv) records the 48 teams, groups, host flags, and a dated FIFA ranking. The source links are recorded in its columns.
- [`world_cup_2026_third_place_routes.csv`](01%20Input/world_cup_2026_third_place_routes.csv) records the 495 third-place allocation combinations. It includes the source link to the [FIFA 2026 regulations](https://digitalhub.fifa.com/m/636f5c9c6f29771f/original/FWC2026_regulations_EN.pdf) and its extraction source.

## Limits of interpretation

- In the 24,148-match modelling sample, 47.6% of 90-minute targets can be reconstructed from complete goal events. For 43.6%, the reported final score is used as an approximation; a small subset of extra-time wins may therefore be labelled as 90-minute wins. The remaining 8.8% are known draws despite incomplete events.
- Validation covers international matches generally, rather than a World Cup-only holdout. The same temporal folds were used to choose settings and report performance, so the 0.8844 log loss is a model-selection estimate rather than an untouched test result. The hyperparameter search is deliberately compact because of runtime, so it does not establish the best possible setting for each model.
- Group ties use FIFA ranking after points because the model does not generate scorelines. Knockout draws are resolved by renormalising the two 90-minute win probabilities; extra time and penalties are not modelled separately.
- Team Elo and form remain fixed during each simulated tournament. Model error, calibration error, and these rule approximations can have a larger effect than Monte Carlo sampling noise.

The public repository contains the analysis and a representative result snapshot. It does not contain the original `.venv`, generated intermediates, or the 125 MB match-by-match simulation CSV.
