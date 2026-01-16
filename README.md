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

**[View / Run the notebook on Kaggle] (https://www.kaggle.com/code/matthewmaree/ens-model)**

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
   - Merges the competiton datasets with provided lookup tables, which contain additional molecular descriptors for the different solvents used,
     including solvent properties and molecular fingerprints
   - Uses dedicated cross-validation split generators supplied by the organisers:
     - Leave-one-solvent-out splits for single-solvent
     - Leave-one-ramp-out splits for the full dataset

2. **Feature Engineering**
   - Custom featurizer methods were built for the single-solvent and multi-solvent tasks, which include the following:
      - Numeric features:
         - Temperature and residence time transforms (e.g. interactions, scaling)
      - Solvent features:
         - A combined solvent feature table built from:
            - Physicochemical descriptors (e.g. SPANge / ACS-type descriptors)
            - PCA-reduced features
            - Binary fingerprint-style descriptors (e.g. DRFP, fragment fingerprints)
      - Correlation-based feature filtering to remove redundant descriptors
      - For the multi-solvent featurizer:
         - Separate A/B solvent descriptors
         - Linear mixing by solvent fraction (`SolventB%`)

4. **Models**
   - **CatBoost** regressor with tuned hyperparameters.
   - **XGBoost** regressor with tuned hyperparameters.
   - **RandomForrest** regressor with tuned hyperparamters.

   Separate models are trained for:
   - Single-solvent reactions
   - Full/mixed-solvent reactions
  
     - Model optimization:
        - Hyperparamters for the various models, in both the single-solvent and multi-solvents tasks were tuned offline using GridSearchCV on a cross-validation subset
        - The tuning cells are not included in the final Kaggle submission notebook, as the competion format requires a fixed cell strucutre and limited execution time 
        

5. **Ensembling**
   - A simple linear weighted ensemble of:
     - CatBoost model
     - XGBoost model
   - Weights chosen based on cross-validation performance for each dataset (weights optimized offline)
   - The RandomForrest model was found to harm performance when included in the ensemble model, so it was given a nominal weight of 0 for both single-solvent and mult-solvent ensemble models
   - The ensemble consistently outperforms the individual base models

6. **Evaluation**
   - Cross-validation using:
     - Leave-one-solvent-out for the single-solvent task
     - Leave-one-ramp-out splits for the full task
   - Evaluation metric: RMSE on held-out validation folds (final ensemble model obtained an RMSE of 0.08825 on the combined evaluation of the single-solvent and multi-solvent tasks)
   - The final ensemble scores are competitive with the top Kaggle leaderboard entries.

---
