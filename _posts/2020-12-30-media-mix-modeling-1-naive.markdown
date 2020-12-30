---
layout: post
title:  "Media Mix Modeling - Step 1 (A Naive Approach)"
date:   2020-12-30 18:00:00 +0900
categories:
---

Media Mix Modeling (MMM) is a common approach to understanding the causal impact of marketing spend on revenue.

Traditionally, these have been simple regression models built from historical marketing spend and revenue data, also incorporating control variables such as fluctuations in price, seasonality, other promotions etc.

The [*adstock*](https://en.wikipedia.org/wiki/Advertising_adstock) function was subsequently introduced to capture the following:

- Lag effect - advertising campaigns increase awareness, but this awareness has a half-life that decays over time.
- Diminishing returns - above a certain threshold, incremental spend on advertising may have a linear increase on reach, but this is not matched by a linear increase in demand.

MMM has also been the subject of several recent research papers at Google, including a [summary of challenges in implementation](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45998.pdf) and a [Bayesian approach to estimating the adstock function](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/b20467a5c27b86c08cceed56fc72ceadb875184a.pdf).

This post serves as a step in my learning journey, purely for practice, building a traditional MMM by framing it as a simple optimization problem without incorporating adstock. I hope to cover improvements to the naive model in separate blog posts.

## Naive Approach

- Use historic spend/performance data to get a set of coefficients that capture the relationship between spend in various channels and revenue.
- Given a set of business requirements for how the budget should be spent, output a recommended level of spending across each marketing channel.
- That’s it.

## Obtaining Coefficients

Taking a naive approach, a linear regression on historical sales data can be represented by the equation below:

*<center>R = x1 + x2... + y1 + y2... + z1 + z2...</center>*

Where *R* is revenue, each *x* represents an ad channel, each *y* represents a  control variable (such as season, price) and each *z* represents a parameter (such as baseline sales without any advertising).

There are some obvious drawbacks to the naive approach:

- Coefficients only capture the relationship between the ad channel and revenue during the time period for the data provided, not accounting for lag or diminishing returns.
- Control variables and model parameters may be difficult to estimate without appropriate priors.
- The actual data required for this regression would need to be consistent across channels and at the same level of granularity, which may prove difficult in practice (where this may be spread across departments/ad agencies)

Theoretically, the regression should return realistic coefficients for each ad channel to be used in the next step.

## Framing as an optimization problem

In its simplest form, the resulting revenue given spend in 3 marketing channels, without accounting for lag or diminishing returns, can be expressed as follows:

*<center>Revenue = c1x1 + c2x2 + c3x3</center>*

Where each *c* represents the ad channel coefficient (relationship between channel spend and revenue) and each *x* represents total spend in that channel.

There may be additional business requirements that constrain spending. Following are three examples, including their representation as linear constraints.

#### Constraint 1: Total budget should not exceed ¥1,000,000

*<center>x1 + x2 + x3 <= 1,000,000</center>*

#### Constraint 2: Spend in channel 1 should not exceed 45% of total spending

*<center>x1 <= 0.45(x1+x2+x3)</center>*

#### Constraint 3: Spend in channels 1 & 2 should not exceed 60% of total spending

*<center>x1 + x2 <= 0.6(x1+x2+x3)</center>*

## SciPy implementation

The SciPy library provides a few optimization algorithms that can be applied to this use case. For an optimizer that takes into account constraints but not diminishing returns, we can use the [linprog](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linprog.html) class.

In the case below, there are 3 channels (*a*,*b*,*c*) subject to the constraints above, and the coefficients have been set such that *a* > *b* > *c*. As expected, the optimizer suggests:

- With a budget of ¥1,000,000 (constraint 1),
- ¥450,000 should be spent on channel *a* (constraint 2)
- ¥150,000 on channel *b* (constraint 3)
- The remaining ¥400,000 on channel *c*.

[Implementation in Colab](https://colab.research.google.com/drive/1JB_Gbdp2Tk7Zjc-EZIoZLrk1E6laUcA5?usp=sharing)

{% highlight python %}
from scipy.optimize import linprog

budget = 1000000

fun = [-a.coef,-b.coef,-c.coef]
lhs_ineq = [[1,1,1],[1,0,0],[1,1,0]]
rhs_ineq = [budget,(budget * 0.45),(budget * 0.6)]

res = linprog(fun, A_ub=lhs_ineq, b_ub=rhs_ineq, method="revised simplex")
{% endhighlight %}

{% highlight python %}
con: array([], dtype=float64)
     fun: -305000.0
 message: 'Optimization terminated successfully.'
     nit: 3
   slack: array([0., 0., 0.])
  status: 0
 success: True
       x: array([450000., 150000., 400000.])
{% endhighlight %}
