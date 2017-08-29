---
layout: post
title: Technically&#58; Deming versus simple linear regression
---

All courses that somehow covered regression models were starting almost in the same way: given bunch of $y$'s and $x$'s points, one needs to predict a value of $y$ for a certain $x$. Sounds quite easy. Without utilizing any statistical assumptions, we can just find the line, which is in a way closest to those points. So the model is as follows:

$$y \approx \beta_0 + \beta_1 x$$

Then typically a professor of a course leads to idea of minimizing the distances between observed variables and the fitted ones, i.e.:

$$\sum_i=1^n(y_i - (\beta_0 + \beta_1 x_i))$$

But those distances might be both positive or negative. Instead, we can use the absolute values, but squaring those distances leads to much nicer properties for optimization:

$$\sum_i=1^n(y_i - (\beta_0 + \beta_1 x_i))^2$$

Minimizing this term yields so-called linear or more commonly ordinary least squares (OLS) estimates.

So far, we did not use any of statistical assumptions. Of course due to world imperfection the relation between $y$ and $x$ is not exact and incorporates errors. These errors are modeled by presence of randomness. Let's define the other model, which includes a random term referred as error or disturbance term: 

$$y = \beta_0 + \beta_1 x + \epsilon$$

Since we know the theoretical distribution of $y$ and have a sample, we can use a powerful machinery of maximum likelihood estimation (MLE) method. It turns out that with several assumptions (homoscedastic, normal and uncorrelated errors) MLE is identical (!) to OLS (see Bradley 1973). I do not about you, but this fact shows the whole beauties of mathematics to me.

So far so good, but let's step back for a second and check ourselves. Minimizing the distances between observed variables and the fitted ones sounds very good, but wait a second... That's how the vertical distance defined, not the orthogonal. The minimal distance between a point and a line is actually orthogonal distance, not the vertical one. It means that OLS minimizes vertical distances instead of orthogonal, which is illustrated below.

![](https://irudnyts.github.io/images/v_vs_o.jpeg)

Minimizing the square orthogonal distances yields so-called Deming regression estimates, which is identical to MLE of errors-in-variables model with $\sigma = 1$. This model is used mostly in clinical chemistry. 
