---
layout: post
title: "&#128202; Multinomial regression in R"
---

In my current project on [Long-term care](https://www.youtube.com/watch?v=kLf6SVEMd94) at some point we were required to use a regression model with multinomial responses. I was very surprised that in contrast to well-covered binomial GLM for binary response case, multinomial case is poorly described. Surely, there are half-dozen packages overplapping each other, however, there is no sound tutorial or vignette. Hopefully, my post will improve the current state.

We can distinguish two types of multinominal responses, namely nominal and ordinal. For nominal response a variable can possess a value from predifined finite set and these values are not oredered. For instance a variable `color` can be either `green` or `blue` or `green`. In machine learnin the problem is often referred to as a classification. In contrast to nominal case, for ordinal reponse variable the set of values has the relative ordereing. For example, a variable `size` can be `small < middle < large`. Furthermore, depending on a link function we can have logit or probit models.

## Nominal response model

According to Agresti (2002) we can the problem can be formulated by two similar approaches: through baseline-category logits or multivariate GLM. In general, these two approaches are equiualent with identical maximum-likelihood estimates, the only thing which is different is the formula representation. 

# Baseline-category logits (multinomial logit model)

The baseline-category logits is implemented as a function in three distinct packages, namely `nnet::multinom()` (referred as to log-linear model), `mlogit::mlogit`, `mnlogit::mnlogit` (claims to be more efficient implementation than `mlogit`).

Let $p_j = \mathbb{P}(Y = j|x)$ is a probability of dependent variable $Y$ to have value $j$ given a vector of explaratory variables' values $x$. In total, there are $J$ categories, and obviously, due to second axiom of probability $\sum_j p_j = 1$. We fix a baseline category at level $J$ (or at any other level), and the model is as follows:

$$\log \frac{p_j}{p_J} = \alpha_j + \beta'_j x, \quad j = 1, ..., J - 1,$$

describing the effects of exlporatory $x$ on logits of odds between a level $j$ and baseline level. Of course, using these $J-1$ equiations and the second axiom it's possible to come back to probabilities (which is a nice exercise, by the way): 

$$ p_j = \frac{\exp(\alpha_j + \beta'_j x)}{1 + \sum_{h = 1}^{J-1}\exp(\alpha_h + \beta'_h x)}$$

Let's now to estimate those $\beta_j, \quad j = 1, ..., J - 1$ by different packages. I use `marital.nz` data from `VGAM` package, which we cover later.

```r
# install.packages("VGAM")
library(VGAM)
data(marital.nz)
#   age ethnicity            mstatus
# 1  29  European             Single
# 2  55  European  Married/Partnered
# 3  44  European  Married/Partnered
# 4  53  European Divorced/Separated
# 5  45  European  Married/Partnered
# 7  30  European             Single
unique(marital.nz$mstatus)
# [1] Single             Married/Partnered  Divorced/Separated Widowed           
# Levels: Divorced/Separated Married/Partnered Single Widowed
```

The data contains "marital data mainly from a large NZ company collected in the early 1990s". Dependent variable `mstatus` has four unordered classes `Divorced/Separated`, `Married/Partnered`, `Single`, and `Widowed`. We use `age` as the only exploratory variable.

- Package `nnet`

```r
library(nnet)
fit_nnet <- multinom(mstatus ~ age, marital.nz)
coef(fit_nnet)
#                   (Intercept)          age
# Married/Partnered    2.778686 -0.003538729
# Single               6.368064 -0.152745520
# Widowed             -6.753123  0.099333903
```

- Package `mlogit`

```r
library(mlogit)
fit_mlogit <- mlogit(mstatus ~ 0 | age, data = marital.nz, shape = "wide")
matrix(fit_mlogit$coefficients, ncol = 2)
#           [,1]         [,2]
# [1,]  2.778666 -0.003538297
# [2,]  6.368056 -0.152745424
# [3,] -6.753157  0.099334560
```

- Package `mnlogit`

Also have a look how this guy [compared the perfomance of these functions](https://www.r-bloggers.com/comparing-mnlogit-and-mlogit-for-discrete-choice-models/)

