---
layout: post
title: Elegantly inversion functions as an nice example of a function operator
---

In the course I assist this spring ([https://hec.unil.ch/hec/syllabus/descriptif/2015?dyn_lang=en](Simulation Methods in Finance and Insurance)), there are plenty of places where one has to inverse a function.

For instance, [https://en.wikipedia.org/wiki/Inverse_transform_sampling](inversion method) of generating pseudo random numbers from a general distribution, which requires to use an inverse c.d.f. (or quantile function). Further on, one has compute a so-called generator of [https://en.wikipedia.org/wiki/Copula_%28probability_theory%29#Most_important_Archimedean_copulas](Archimedean copula). Even for calculating a value at risk ([https://en.wikipedia.org/wiki/Value_at_risk](VaR)) the inversion might be useful. Of course, one can argue that all these methods have already been implemented in R, but the story is rather about inversion. In R there are no built-in [http://adv-r.had.co.nz/Function-operators.html](function operator) that returns a inverse of mathematical functions. This [http://stackoverflow.com/questions/10081479/solving-for-the-inverse-of-a-function-in-r](post) propose a rather elegant solution (with corrected syntax):

```r
inverse <- function(f, lower, upper) {
    function(y) uniroot(function(x) f(x) - y, lower = lower, upper = upper)[["root"]]
}
```

Instead of explicitly define an inverse function, now all mathematical functions can be inversed by a line of code.

```r
log2 <- inverse(exp, 0.1, 10)
log2(exp(1))
```

Please, note that I avoid default values for arguments `lower` and `upper`, ’cause they really require a care.