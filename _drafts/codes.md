---
layout: post
title: Consciously&#58; different codes
---

```r
a <- 1
f <- function(a = a) {
    a
}
f()
```

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

```r
function(a = 1) {
	a + 1
}(1)

(function(a = 1) {
	a + 1
})(1)

(f <- function(a = 1) {
    a + 1
})(1)



```