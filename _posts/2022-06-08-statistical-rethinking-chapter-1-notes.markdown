---
layout: post
title:  "Notes on Statistical Rethinking: Chapter 1"
date:   2022-06-08 10:30:00 +0900
categories:
---

## The Problem with Deductive Falsification

Many people believe that the best way to do science, the proper objective of statistical inference, is to test null hypotheses. 
This way of thinking was popularized by Karl Popper, who argued that we can better deepen our understanding of the world by developing hypotheses that are falsifiable. However, deductive falsification is impossible in the vast majority of contexts.

The problem lies here: Many process models, explanations as to why a hypothesis is true, correspond to the same hypothesis. A traditional approach is then to use a statistical model to validate the explanation. But a single statistical model can validate explanations that correspond to different hypotheses.

An example follows. Since the 1960s, evolutionary biologists have been interested in the idea that evolutionary changes in gene frequency are not caused by natural selection, but by mutation and drift. This leads to the two hypotheses in the diagram below.

{:refdef: style="text-align: center;"}
![hypothesis_process_model](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2022-06-08/hypothesis_process_model.png){: style="text-align:center"}
{: refdef}

For each hypothesis there are potentially hundreds of possible process models. In the case of the null hypothesis H<sub>0</sub>, researchers proposed P<sub>0A</sub> and P<sub>0B</sub>, which describe whether population size and structure have been stable for long enough for the distribution of alleles in a genetic sequence to reach a stable state.

For the hypothesis that evolutionary changes are caused by natural selection, again there are many possible process models. The diagram shows two; a model in which selection favours certain alleles, and another in which selection fluctuates through time, favouring different alleles.

M<sub>II</sub> refers to a statistical model that can capture the expected frequency distribution of a quantity - a histogram. Process model P<sub>0A</sub> would predict that alleles follow a power law distribution. However, P<sub>1B</sub> would also entail that the distribution of alleles follows a power law. In this case, a statistical test seems to validate both the hypothesis that evolutionary changes in gene frequency are caused by natural selection, and the exact opposite.

## Developing Better Hypotheses

One response to the above may be: "Why not develop more precise hypotheses?". Consider the hypothesis "All swans are white."

1. Imagine finding a swan that is a shade of grey. Is the swan white? This may be easy enough to debate, but in many fields the definition of the concept being measured is as much in question as its existence.
2. Most interesting hypotheses are continuous: "80% of swans are white." This calls for a method that describes the colour distribution as accurately as possible, rather than something to falsify.

So, what next?

## A Better Set of Tools

Statistical Rethinking proposes four main tools to design inquiry, extract information, make predictions, and argue the case for some causal explanation of a phenomenon. These are:

# Bayesian Data Analysis

This process involves counting the number of ways you could get a set of observations. If an explanation can cause the data to arise in more ways, that explanation is more plausible. It turns out that if you have an idea of prior plausibility, you can use new information to update plausibility iteratively using Bayes' Theorem.

# Model Comparison

Models can produce predictions. You should prefer models that produce better predictions. This may imply having some understanding of the future, but there are tools you can use without owning a crystal ball: Cross-validation and information criteria.

There is a trade-off when building predictive models between optimization and generalizability. A model, retrofit on past data, can notice extremely complex patterns. Applying that model to make predictions on new data can lead to poor predictions, because the model was overfit to the unique circumstances under which the first dataset originated. A complex model can lack generalizability to new data.

The tools here combat overfitting, allowing models to generalize better.

# Multilevel Models

While cross-validation and information criteria identify the risk of overfitting, multilevel models help us do something about it.

Observations are often clustered in ways that are not foreseen (for example, by location or year). Single-level models lack interpretability in the face of clustered observations.

1. If observations are sampled repeatedly for the same dimension (individual, location, time, etc.) single-level models can be misleading.
2. When some observations are sampled more frequently for one dimension, single-level models are again misleading.
3. If the research question requires an explanation of variation across different dimensions, single-level models are less interpretable than multilevel models.
4. Pre-averaging data across dimensions can remove variation, which removes important nuances in the model. Multilevel models preserve these nuances.

# Graphical Causal Models

Wind causes tree branches to sway, tree branches do not cause wind to blow. A statistical model would be awesome at finding this association, but it would be absolutely terrible at describing what caused what.

To make matters worse, it turns out that causally incorrect models can make better predictions than causally correct models. This is because confounded relationships, where two variables X and Y appear to be correlated but in reality both are affected by an unobserved Z variable, are still real relationships. Observing swaying branches really would predict wind.

Rather than relying on models alone, a complete picture requires background understanding of the question - and a reasonable causative mechanism to explain the plausibility of the data.

This can be done with graphical causal models, the simplest version of which is a directed acrylic graph (DAG).

{:refdef: style="text-align: center;"}
![DAG](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2022-06-08/DAG.png){: style="text-align:center"}
{: refdef}

The creation of such graphs requires the use of information outside the data. While this sounds inherently unscientific, no model can accept data alone and give a reliable representation of causal mechanisms. Developing an understanding of reality is an iterative process of developing theories, judging the plausibility of each theory given the observations, and continuing to dig in the direction most likely to lead to truth.
