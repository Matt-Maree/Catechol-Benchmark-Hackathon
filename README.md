# Catechol Reaction Yield Prediction

This repository contains a machine-learning pipeline for predicting **reaction yields** for catechol chemistry using experimental reaction conditions and molecular descriptors. 

The goal of the project is to model reaction yield as a regression problem, using a combination of:
- Reaction conditions
- Molecular and solvent descriptors
- Ensemble machine-learning models

Both **single-solvent** and **mixed-solvent** reaction datasets are supported.

This project is part of of my solution for the Catechol Benchmark Hackathon (NeurIPS 2025 DnB) hosted on Kaggle. 
Catechol Benchmark Hackathon (NeurIPS 2025 DnB)

---

## Project Overview

The workflow consists of:

1. Data loading and cleaning  
2. Feature engineering for reaction conditions and solvents  
3. Model training and hyperparameter optimisation  
4. Model ensembling  
5. Cross-validated evaluation of predictive performance  

---

## Data and Feature Engineering

### Reaction Data

The datasets include experimental reaction information such as:
- Reaction temperature
- Residence time
- Starting material and product quantities
- Solvent identities (single or mixed)
- Observed reaction yield

Data cleaning includes:
- Consistent handling of missing values
- Filtering invalid or unphysical entries
- Standardisation of numeric features

---

### Feature Engineering

Feature construction is handled through custom preprocessing functions and classes:

- Numeric feature engineering, including:
  - Squared terms
  - Interaction terms (e.g. temperature × residence time)
  - Stoichiometric imbalance features
- Solvent feature tables constructed by combining:
  - Physicochemical descriptors
  - PCA-reduced solvent representations
  - DRFP-based reaction fingerprints (where applicable)
- Support for:
  - Single-solvent reactions
  - Mixed-solvent reactions with compositional weighting

Correlation-based feature filtering is applied to remove redundant descriptors.

---

## Models

The following regression models are implemented and evaluated:

- Random Forest
- XGBoost
- CatBoost

For each model:
- Hyperparameters are optimised using cross-validated search
- Performance is evaluated using RMSE on held-out validation splits
- Separate models are trained for:
  - Single-solvent reactions
  - Mixed-solvent reactions

---

## Ensembling

- Multiple model predictions are combined using a weighted ensemble
- Ensemble weights are chosen based on cross-validated performance
- Ensembling improves robustness relative to individual models

---

## Results

- Cross-validated RMSE values are competitive with top Kaggle leaderboard scores
- The ensemble model consistently outperforms individual base models
- Performance is evaluated using multiple random seeds to assess stability

---

## Tools and Libraries

- Python
- NumPy
- Pandas
- scikit-learn
- XGBoost
- CatBoost
