---
layout: post
title: "&#129365; Dortmund real estate market analysis&#58; data preprocessing with caret"
---

This is rather a short note, which is more related to an amazing package `caret`, than to our data set. The package allows for manipulating the model with less typing, for instance cross-validation or data preprocessing can be done by just specifying a couple of arguments in the key function of package `train`.

Perhaps, the median imputation and $k$-nearest neighbors algorithm I will live for the better times, since Dortmund real estate data set contains no missed values. Furthermore, `caret` makes it possible to apply various transformation of data, e.g. centering, scaling, principle component analysis (PCA), independent component analysis (ICA) etc. As one can remember, the model with the smallest out-of-sample RMSE is GAM with inverse Gaussian responses and log-link function. GAM assumes applying smooth functions to regressors, and thus, centering and scaling won't improve our metric. On the other hand, I am not a big fan of centering and scaling, since both process make the sample dependent. In other words, subtracting the sample mean from each observation makes this observation dependent on the whole sample. 

On the other hand, PCA might be very useful, since `rooms` and `area` are correlated. Can this improve the model? Let's experiment and see. As usual, we start from loading packages and data.

```r
packages <- c("mgcv", "magrittr", "vtreat", "caret")
sapply(packages, library, character.only = TRUE, logical.return = TRUE)
# options(scipen=999)
rm(list = ls())

setwd("/Users/irudnyts/Documents/data/")
property <- read.csv("dortmund.csv")
```

Below our GAM with IG outcome and log-link function is rewritten in `caret` syntax yielding the same RMSE:

```r
model <- train(price ~ rooms + area, data = property,
               method = "gam", family = inverse.gaussian(link = "log"))
pred <- predict(model, property[, c("area", "rooms")])
(pred - property[, "price"]) ^ 2 %>% mean() %>% sqrt()
# [1] 134.9973
```

I don't utilize the power of the function `trainControl()`, which can be used for cross-validation, in order to obtain consistency with previous posts. The model with PCA transformation is shown below:

```r
model_pca <- train(price ~ rooms + area, data = property,
                   method = "gam", family = inverse.gaussian(link = "log"), 
                   preProcess = "pca")
pred <- predict(model_pca, property[, c("area", "rooms")])
(pred - property[, "price"]) ^ 2 %>% mean() %>% sqrt()
# [1] 146.4144
```

The in-sample RMSE is higher than our best model, which is not very encouraging. Even though it makes a little sense to go further, we calculate our-of-sample RMSE. 

```r
set.seed(3)
folds <- kWayCrossValidation(nRows = nrow(property), nSplits = 3)

pred <- rep(NA, nrow(property))
for(fold in folds) {
    model_pca <- train(price ~ rooms + area, data = property[fold$train, ],
                       method = "gam", family = inverse.gaussian(link = "log"), 
                       preProcess = "pca")
    
    pred[fold$app] <- predict(model_pca, property[fold$app, ])
}

sqrt(mean((pred - property$price) ^ 2))
# [1] 151.7058
```

I can summarize in one short sentence: PCA is not helpful for this case.