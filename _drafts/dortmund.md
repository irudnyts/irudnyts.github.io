---
layout: post
title: Dortmund real estate market analysis continued
---

```r
log(price) ~ area + rooms
price ~ area + rooms, family = gamma
price ~ area * rooms, family = gamma
prices ~ s(area) + s(rooms)
```

- Radndom forest
- XGboost 

- train-test data
- cross-validation
- RMSE
