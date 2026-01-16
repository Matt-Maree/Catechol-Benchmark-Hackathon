# Catechol Reaction Yield Prediction

This project builds a multi-model ensemble (CatBoost + XGBoost) for predicting reaction yields in the **[Catechol Benchmark Hackathon (NeurIPS 2025 DnB)](https://www.kaggle.com/competitions/catechol-benchmark-hackathon/overview).** Custom solvent featurization, correlation-filtered descriptors, and dataset-specific cross-validation schemes were used to achieve competitive RMSE performance. Full solution is implemented in a Kaggle notebook due to competition constraints.

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

is implemented in this Kaggle notebook: **[View / Run the notebook on Kaggle](https://www.kaggle.com/code/matthewmaree/ens-model)**

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
   - Merges the competition datasets with provided lookup tables, which contain additional molecular descriptors for the different solvents used,
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
            - PCA-reduced solvent descriptors (provided by the competition organisers)
            - Binary fingerprint-style descriptors (e.g. DRFP, fragment fingerprints)
      - Correlation-based feature filtering to remove redundant descriptors
      - For the multi-solvent featurizer:
         - Separate A/B solvent descriptors
         - Linear mixing by solvent fraction (`SolventB%`)

3. **Models**
   - **CatBoost** regressor with tuned hyperparameters
   - **XGBoost** regressor with tuned hyperparameters
   - **RandomForest** regressor with tuned hyperparameters (tested but excluded from final ensemble)
  
   - Separate models are trained for:
      - Single-solvent reactions
      - Full/mixed-solvent reactions
     
   - Multi-output setup:
      - Each model predicts three targets per reaction: the remaining starting material and the yields of the two products
      - RandomForest trains three separate regressors (one per target)

   - Output post-processing:
      - All predicted yields are clipped at a minimum value of 0 (negative yields are chemically impossible)
      - If the sum of the three yields exceeds 1, the values are rescaled so that their sum equals 1 (combined yield of material greater than 1 is chemically impossible) 
      
   - Model optimization:
      - Hyperparameters for all models, both single-solvent and multi-solvent, were tuned offline using GridSearchCV on a cross-validation subset
      - The tuning cells are not included in the final Kaggle submission notebook, as the competition format requires a fixed cell structure and limited execution time  
        
4. **Ensembling**
   - A simple linear weighted ensemble of:
     - CatBoost model
     - XGBoost model
   - Weights chosen based on cross-validation performance for each dataset (weights optimized offline)
   - The RandomForest model was found to harm performance when included in the ensemble model, so it was given a nominal weight of 0 for both single-solvent and multi-solvent ensemble models
   - The ensemble consistently outperforms the individual base models

5. **Evaluation**
   - Cross-validation using:
     - Leave-one-solvent-out for the single-solvent task
     - Leave-one-ramp-out splits for the full task
   - Evaluation metric: RMSE on held-out validation folds (final ensemble model obtained an RMSE of 0.08825 on the combined evaluation of the single-solvent and multi-solvent tasks)
   - The final ensemble scores are competitive with the top Kaggle leaderboard entries.

## Reproducibility Notes
   - All data loading and utility functions come from the official Kaggle competition package
   - Full code is inside the Kaggle notebook due to fixed-cell requirements
   - Hyperparameter tuning was performed offline and documented here, but not included in the final Kaggle notebook due to execution-time limits

## Future Improvements
   - Implement more sophisticated ensembling methods, such as RidgeRegression
   - Add additional models to the ensemble, such as LightGBM or NGBoost
   - Explore solvent-specific error trends to assess robustness
   - Experiment with neural networks or GNNs for solvent embeddings

## Key Skills Demonstrated
   - Multi-output regression for chemical yield prediction
   - Gradient boosting model development (CatBoost, XGBoost)
   - Feature engineering using chemical descriptors and fingerprints
   - Correlation filtering and output post-processing
   - Custom featurizer design for mixed-solvent systems
   - Cross-validation strategy design for chemical datasets
   - Weighted model ensembling
   - Competition workflow management and reproducibility
