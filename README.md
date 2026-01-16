# Catechol Reaction Yield Prediction

This project contains my solution for the **Catechol Benchmark Hackathon (NeurIPS 2025 DnB)** on Kaggle.

The goal is to predict **reaction yields** for catechol chemistry from:
- Reaction conditions (temperature, residence time, stoichiometry, etc.)
- Solvent information (single- and mixed-solvent systems)
- Precomputed solvent descriptors and reaction fingerprints

Because of Kaggle’s competition rules, the **full data and utilities live on Kaggle**, and the main training code is run there.

---

## Kaggle Notebook (Full Solution)

The complete, runnable solution (including:
- data loading with the official competition utilities,
- feature engineering,
- model training,
- cross-validation,
- and `submission.csv` generation)

is implemented in this Kaggle notebook:

👉 **[View / Run the notebook on Kaggle] (https://www.kaggle.com/code/matthewmaree/ens-model)**

You can:
- see the exact code used for my submissions,
- run it directly in the Kaggle environment,
- and reproduce the cross-validation and final predictions.

---

## Approach Overview

At a high level, my solution does the following:

1. **Data Loading**
   - Uses the competition-provided utilities to load:
     - Single-solvent dataset
     - Full/mixed-solvent dataset
   - Uses dedicated cross-validation split generators supplied by the organisers:
     - Leave-one-out splits for single-solvent
     - Leave-one-ramp-out splits for the full dataset

2. **Feature Engineering**
   - Numeric features:
     - Temperature and residence time transforms (e.g. interactions, scaling)
   - Solvent features:
     - A combined solvent feature table built from:
       - Physicochemical descriptors (e.g. SPANge / ACS-type descriptors)
       - PCA-reduced features
       - Binary fingerprint-style descriptors (e.g. DRFP, fragment fingerprints)
   - Mixed solvents:
     - Separate A/B solvent descriptors
     - Linear mixing by solvent fraction (`SolventB%`)
   - Correlation-based feature filtering to remove redundant descriptors.

3. **Models**
   - **CatBoost** regressor with tuned hyperparameters.
   - **XGBoost** regressor with tuned hyperparameters.
   - (Random Forest was explored but performed worse in this setting.)

   Separate models are trained for:
   - Single-solvent reactions
   - Full/mixed-solvent reactions

4. **Ensembling**
   - A simple weighted ensemble of:
     - CatBoost model
     - XGBoost model
   - Weights chosen based on cross-validated performance for each dataset.
   - The ensemble consistently outperforms the individual base models.

5. **Evaluation**
   - Cross-validation using:
     - Leave-one-out for the single-solvent task
     - Leave-one-ramp-out splits for the full task
   - Evaluation metric: RMSE on held-out validation folds.
   - The final ensemble scores are competitive with the top Kaggle leaderboard entries.

---
