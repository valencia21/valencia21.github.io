---
layout: post
title:  "Predictive Models in Python with XGBoost, LightGBM and HyperOpt (Part 1)"
date:   2022-01-25 20:00:00 +0900
categories:
---

In [this notebook](https://github.com/valencia21/data-science-projects/blob/master/house-price-prediction/Models/sklearn/model.ipynb), I use the following approach to generate predictions on the Kaggle House Price Dataset:

- Data wrangling (Light exploration, followed by removing and transforming some variables)
- Fit a model using a Scikit-Learn pipeline 
  - Data Preprocessing + fitting XGBoost/LightGBM estimators with a Randomized Search across their respective hyperparameters
- Evaluate and visualize model performance
- Implement an automated approach to selecting optimal hyperparameters for each estimator (HyperOpt)
- Make predictions