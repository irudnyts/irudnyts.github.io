---
layout: post
title: Consciously&#58; installing and loading R packages
---

```r
DT <- data.table(a = letters[1:20], b = sample(1:20))

DT[, "a", with = TRUE]
DT[, "a", with = FALSE]
DT[, "a"]

DT[, a]
DT[, .(a)]

DT[, a, drop = FALSE]
DT[, a, drop = TRUE]

DT[, "a", drop = FALSE]
DT[, "a", drop = TRUE]

DT[, "a"] <- LETTERS[1:20]
DT[, a] <- LETTERS[1:20]

```

Fucking NSE :(

How to write the simplest function?