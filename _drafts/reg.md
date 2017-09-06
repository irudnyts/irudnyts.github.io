---
layout: post
title: Regression: two fold model
---

```r
x <- runif(100)
x1 <- x + rnorm(100)
y <- x + rexp(n = 100)
summary(lm(y ~ x1))
summary(lm(y ~ x))
summary(lm(y ~ x + x1))
cor(x, x1)



```

