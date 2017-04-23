---
layout: post
title: Consciously installing and loading R packages
---

One reason of R popularity is an ocean of packages. Even though it is pretty straightforward to manage packages, there are a couple of tricks, do’s and don’ts, and other things which require a care.

# Installing

The standard built-in function works perfectly:

```r
install.packages("ggplot2")
```

With several packages as well:

```r
install.packages(c("ggplot2", "plyr"))
```

Sometimes if one needs to install particular version of a package, `devtools` package provides the following function:

```r
# install.packages("devtools")
devtools::install_version(package = "ggplot2",
                          version = "2.1.0",
                          repos = "http://cran.us.r-project.org")
```
Finally, in order to see all installed packages I use more frequently `library()` instead of `installed.packages()`, which is an overkill in a way.

# Loading and using

Two functions could be used for loading packages, namely `require()` and `library()`. The *latter* is a little bit more *preferable*, because it throws an error if the package does not exist. `require()` only returns `FALSE` and generates a warning.

The function argument `package` can be both either a character vector (must be of length 1) or a name. In other words, both expression will load package `ggplot2`:

```r
library(ggplot2) # or
library("ggplot2")
```

The latter is possible thanks (or not thanks) to [http://adv-r.had.co.nz/Computing-on-the-language.html](non-standard evaluation) (NSE). I strongly recommend avoid NSE by using the one with quotes, which would save you from situations as follows (only God knows what kind of pervert would write this):

```r
ggplo2 <- "pryr"
library(ggplot2)
```

There is also a nice trick how to load a several packages at once.

```r
lapply(x, library, character.only = TRUE)
```

To see all packages loaded, we typically use `search()`.

One very important thing: do not use neither `library()` nor `require()` in packages’ scripts explicitly. Instead, specify all dependencies in `DESCRIPTION` file. Then, it is also best practice explicitly specify namespace of the desired package using the following syntax `name_of_package::name_of_function()`, e.g.:

```r
copula::Bernoulli(2)
```

This one can be also done when the package installed but not loaded in non-package scripts.

Typically nobody gives a care about these small details. I also suggests to stick to one particular way of doing things, but also be aware of possible drawbacks.