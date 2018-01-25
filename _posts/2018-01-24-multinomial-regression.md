---
layout: post
title: "&#128202; Multinomial regression in R"
---

In my current project on [Long-term care](https://www.youtube.com/watch?v=kLf6SVEMd94) at some point we were required to use a regression model with multinomial responses. I was very surprised that in contrast to well-covered binomial GLM for binary response case, multinomial case is poorly described. Surely, there are half-dozen packages overplapping each other, however, there is no sound tutorial or vignette. Hopefully, my post will improve the current state.

We can distinguish two types of multinominal responses, namely nominal and ordinal. For nominal response a variable can possess a value from predifined finite set and these values are not oredered. For instance a variable `color` can be either `green` or `blue` or `green`. In machine learnin the problem is often referred to as a classification. In contrast to nominal case, for ordinal reponse variable the set of values has the relative ordereing. For example, a variable `size` can be `small < middle < large`. Furthermore, depending on a link function we can have logit or probit models.

## Nominal response models

According to Agresti (2002) we can the problem can be formulated by two similar approaches: through baseline-category logits or multivariate GLM. In general, these two approaches are equiualent with identical maximum-likelihood estimates, the only thing which is different is the formula representation. 

### Baseline-category logits (multinomial logit model)

The baseline-category logits is implemented as a function in three distinct packages, namely `nnet::multinom()` (referred as to log-linear model), `mlogit::mlogit`, `mnlogit::mnlogit` (claims to be more efficient implementation than `mlogit`, see [comparison of perfomances of these packages](https://www.r-bloggers.com/comparing-mnlogit-and-mlogit-for-discrete-choice-models/)).

Let $p_j = \mathbb{P}(Y = j \mid \boldsymbol{x})$ is a probability of dependent variable $Y$ to have value $j$ given a vector of explaratory variables' values $\boldsymbol{x}$. In total, there are $J$ categories, and obviously, due to second axiom of probability $\sum_j p_j = 1$. We fix a baseline category at level $J$ (or at any other level), and the model is as follows:

$$\log \frac{p_j}{p_J} = \alpha_j + \boldsymbol{\beta}'_j \boldsymbol{x}, \quad j = 1, ..., J - 1,$$

describing the effects of exlporatory $\boldsymbol{x}$ on logits of odds between a level $j$ and baseline level. Of course, using these $J-1$ equiations and the second axiom it's possible to come back to probabilities (which is a nice exercise, by the way): 

$$ p_j = \frac{\exp(\alpha_j + \boldsymbol{\beta}'_j \boldsymbol{x})}{1 + \sum_{h = 1}^{J-1}\exp(\alpha_h + \boldsymbol{\beta}'_h \boldsymbol{x})}$$

For each group $j$ the set of parameters $\alpha_j$ and $\boldsymbol{\beta}_j$ are distinct. Let's now estimate those $\alpha_j, \quad \boldsymbol{\beta}_j, \quad j = 1, ..., J - 1$ by different packages and make sure that estimates are identical. I use `marital.nz` data from `VGAM` package.

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

```r
library(mnlogit)
marital.nz_long <- mlogit.data(data = marital.nz, choice = "mstatus")
fit_mnlogit <- mnlogit(mstatus ~ 1 | age | 1, marital.nz_long)
matrix(fit_mnlogit$coefficients, ncol = 2, byrow = TRUE)
#           [,1]         [,2]
# [1,]  2.778666 -0.003538297
# [2,]  6.368056 -0.152745424
# [3,] -6.753157  0.099334560
```

Even though the latter package is very efficient and customizable, there are several points I am not a big fan of. First off, `mnlogit` works *only* with long data instead of common and familiar for regression wide. That's why we had to use `mlogit.data` to convert the data. Second, the formula's syntax is too confusing despite its customizability (yes, this word exists in English).

### Multinomial logit model as multivariate GLM

For this model instead of treating the response variable as a scalar we set to be a vector of $J-1$ elements ($J$-th is redundant). Then, $\boldsymbol{y}_i = (y_{i,1}, ..., y_{i, J-1})'$ and $\boldsymbol{\mu}_i = (p_{i,1}, ..., p_{i, J-1})'$, Therefore,

$$g_j(\boldsymbol{\mu}_i) = \log \frac{\mu_{i,j}}{1 - (\mu_{i,1}+...+\mu_{i, J-1})}$$

and

$$\boldsymbol{g}(\boldsymbol{\mu}_i) = \boldsymbol{X}_i \boldsymbol{\beta}$$

where $\boldsymbol{g}$ is a vector of link functions.

The package `vgam` deals exactly with cases of multivariate GLM and GAM. Let's compute estimates for this model, which sould coincide with previously calculated ones:

```r
library(VGAM)
fit_vgam <- vglm(mstatus ~ age, multinomial(refLevel = 1), 
                 data = marital.nz)
matrix(fit_vgam@coefficients, ncol = 2)
#           [,1]         [,2]
# [1,]  2.778666 -0.003538297
# [2,]  6.368056 -0.152745424
# [3,] -6.753157  0.099334560
```

## Ordinal response model: proportional odds model

For orinal response variable the model is slightly different. Let $Y$ be a categorical response variable with $J$ categories whic are ordered $1<...<J$. Therefore, it is possible to define cumulative probabilities as

$$\mathbb{P}(Y \leq j \mid \boldsymbol{x}) = p_1 + ... + p_j, \quad j = 1, ..., J$$

Then, cumulative logits are:

$$\text{logit}(\mathbb{P}(Y \leq j \mid \boldsymbol{x})) = \log\frac{\mathbb{P}(Y \leq j \mid \boldsymbol{x})}{1 - \mathbb{P}(Y \leq j \mid \boldsymbol{x})} = \log\frac{p_1 + ... + p_j}{p_{j+1} + ...+ p_J}, \quad j = 1, ..., J - 1$$

Let's now define the cumulative logits and exploratory variables $\boldsymbol{x}$:

$$\text{logit}(\mathbb{P}(Y \leq j \mid \boldsymbol{x})) = \alpha_j + \boldsymbol{\beta}' \boldsymbol{x}, \quad j = 1, ..., J-1$$

Note that $\boldsymbol{\beta}$ are the same for each logit. However, intercepts can be different and necesseraly are non-decreasing.

The model got its name from its property:

$$\text{logit}(\mathbb{P}(Y \leq j \mid \boldsymbol{x}_1)) - \text{logit}(\mathbb{P}(Y \leq j \mid \boldsymbol{x}_2)) = \log\frac{\mathbb{P}(Y \leq j \mid \boldsymbol{x}_1) / \mathbb{P}(Y \geq j \mid \boldsymbol{x}_1)}{\mathbb{P}(Y \leq j \mid \boldsymbol{x}_2) / \mathbb{P}(Y \geq j \mid \boldsymbol{x}_2)} = \boldsymbol{\beta}' (\boldsymbol{x}_1 - \boldsymbol{x}_2)$$

Again, there are at least four packages, which calibrate the proportional odds model. Let's quickly compare those estimates using Italian household data for 2006 dataset `ecb06it` from `VGAMdata` package. We try to explain ordinal variable `education` of 8 levels by numeric `age`.

```r
# install.packages("VGAMdata")
library(VGAMdata)
data(ecb06it)
# str(ecb06.it)
head(ecb06.it[, c("age", "education")])
#    age     education
# 1   58    highschool
# 4   81 primaryschool
# 5   52    highschool
# 9   67  middleschool
# 12  56  middleschool
# 16  72 primaryschool
```

- Package `MASS`

Perhaps the most famous function is `MASS::polr`.

```r
library(MASS)
fit_polr <- polr(formula = education ~ age, data = ecb06.it)
summary(fit_polr)$coefficients[, 1, drop = FALSE]
#                                  Value
# age                        -0.06417893
# none|primaryschool         -6.95688936
# primaryschool|middleschool -4.51869196
# middleschool|profschool    -3.06471919
# profschool|highschool      -2.73295822
# highschool|bachelors       -0.96907401
# bachelors|masters          -0.89517059
# masters|higherdegree        2.42815131
```

- Package `VGAM`

```r
fit_vglm <- vglm(formula = education ~ age, family = propodds, data = ecb06.it)
as.matrix(fit_vglm@coefficients)
#                      [,1]
# (Intercept):1  6.95576156
# (Intercept):2  4.51825182
# (Intercept):3  3.06430069
# (Intercept):4  2.73254206
# (Intercept):5  0.96867493
# (Intercept):6  0.89470432
# (Intercept):7 -2.42867591
# age           -0.06417086
```

- Package `ordinal`

```r
library(ordinal)
fit_clm <- clm(formula = education ~ age, data = ecb06.it)
as.matrix(fit_clm$coefficients)
#                                  [,1]
# none|primaryschool         -6.9557784
# primaryschool|middleschool -4.5182645
# middleschool|profschool    -3.0643131
# profschool|highschool      -2.7325541
# highschool|bachelors       -0.9686858
# bachelors|masters          -0.8947152
# masters|higherdegree        2.4286635
# age                        -0.0641711
```

Nice thing about this package is that it allows for using different link functions, i.e. `"logit"`, `"probit"`, `"cloglog"`, `"loglog"`, and `"cauchit"`. To my regret I know only `"logit"` and `"probit"` from this list.

- Package `rms`

```r
library(rms)
fit_lrm <- lrm(formula = education ~ age, data = ecb06.it)
as.matrix(fit_lrm$coefficients)
#                        [,1]
# y>=primaryschool  6.9557784
# y>=middleschool   4.5182645
# y>=profschool     3.0643131
# y>=highschool     2.7325541
# y>=bachelors      0.9686858
# y>=masters        0.8947152
# y>=higherdegree  -2.4286635
# age              -0.0641711
```

This function was rather instable. Adding more exploratory variable have thrown an error a couple of times.

Coefficients are consistent (difference in signs are explained by $\mathbb{P}(Y \leq j)$ and $\mathbb{P}(Y \geq j)$), which is good.

Perhaps, now you have a question which package to use? Well, I do not know, just choose one and stick to it. I will use probably `VGAM`, as long as it covers various models and seems like nicely documented.

References:

- Agresti, A. (2002) Categorical Data, Second edition, Wiley
- [STAT504](https://onlinecourses.science.psu.edu/stat504/node/176)
