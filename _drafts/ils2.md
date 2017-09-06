---
layout: post
title: ILS continued
---

```r
library("zoo")

date <- as.Date(as.yearmon(gsub(" ", "", ils$date.of.issue, fixed = TRUE)))
ils$date.of.issue[is.na(date)]
which(is.na(date))
date[257] <- as.Date(as.yearmon("Sep 2011"))
ils$date.of.issue <- date

gsub("^[^0-9]", "", ils$size)

regexpr("^[^0-9]", ils$size)

x <- regmatches(ils$size, regexpr("^[^0-9]+[0-9]", ils$size))

x
x[!(x %in% c("$", "¤"))]


```

