---
layout: post
title:  "Predictive Models in Tensorflow + Blending Predictions from 3 Models"
date:   2022-03-15 10:30:00 +0900
categories:
---

In [this notebook](https://htmlpreview.github.io/?https://github.com/valencia21/data-science-projects/blob/master/house-price-prediction/Models/tensorflow/tf_model_final.html), I use the following approach to generate predictions on the Kaggle House Price Dataset:

- Build a simple Sequential model in Keras/Tensorflow
- Use the Weights and Biases (WandB) platform to select optimal hyperparameters and record experiments.
  - Experiments are evaluated using K-Fold Cross Validation. Mean RMSE across folds for each experiment are custom logged in WandB.
- Make predictions.
- Blend predictions from the previous post (XGBoost, LightGBM) and this post.
  - By taking the mean of predictions, taking the weighted mean of predictions, and defining a meta-model.
  - Predictions are evaluated on a holdout dataset, kept separate from the training set since the beginning of the project.