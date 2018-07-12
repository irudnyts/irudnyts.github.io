---
layout: post
title: "&#x1f331; Setting a seed in R, when using parallel computation"
---

Generally speaking, if the code does any simulations, it is a good practice to set a seed to make the code reproducible. Setting a seed ensures that the same (pseudo-)random numbers will be generated each time the script is executed. Surprisingly, I found really few posts dedicated to any convention, best practice, or routine of setting a seed in R. Further, when using multiple cores (parallelisation) for simulations, things can get slightly more complicated.

## Seeds in R

In base R there are two main objects to handle seeds: `set.seed()` and `.Random.seed`. For a vast number of problems it is enough to use `set.seed()`, which supplies an integer as a seed. The workflow, then, is as simple as follows:

```r
runif(2)
# > [1] 0.8636221 0.1782020

set.seed(1991)
runif(2)
# > [1] 0.1506231 0.2308308

runif(2)
# > [1] 0.0134826 0.5340390

set.seed(1991)
runif(2) # exactly the same random numbers as before
# > [1] 0.1506231 0.2308308
```

The second object, `.Random.seed`, allows saving and restoring the random number generator (RNG) state. Under the hood `.Random.seed` is a simple atomic integer vector, the first element of which specifies the kind of RNG and normal generator. For instance, the first element of `207` is referred to `"L'Ecuyer-CMRG"` RNG method, and `"Box-Muller"` for normal distribution. The rest of the elements of `.Random.seed` store the current random seed.

This object I find of a particular use because it can be saved without an explicit seed setting. What I mean is one does not need to provide an integer to `set.seed()`, which might be annoying, but rather just saving current seed:


```r
seed <- .Random.seed
runif(2)
# > [1] 0.5696378 0.3737989

runif(2)
# > [1] 0.7199003 0.5540470

runif(2)
# > [1] 0.4383970 0.6494643

.Random.seed <- seed
runif(2) # exactly the same random numbers as before
# > [1] 0.5696378 0.3737989
```

The object `.Random.seed` lives in the global environment, and therefore, should be set there. It can cause some issues if you trying to set the `.Random.seed` inside the function, not caring too much about environments. It means that changing it in the function by simple assignment won't change a seed (the value will be set in execution environment). The following straightforward idea can be used for saving a current seed or setting a custom one inside the function:

```r
reproducible_runif <- function(seed = NULL) {
    
    if(is.null(seed)) {
        seed <- .Random.seed
    } else {
        # .Random.seed <<- seed
        # mind the double arrow to assign in the parent enviroment or
        assign(x = ".Random.seed", value = seed, envir = .GlobalEnv)
    }
    
    return(list(x = runif(1), seed = seed))
    
}
```

Then, this function will return a random number, that can be reproduced: 

```r
r1 <- reproducible_runif()
r1$x
# > [1] 0.4215304

runif(10)
# >  [1] 0.1862207 0.2660995 0.5863689 0.1063663 0.5530690 0.9392229 0.9710050
#    [8] 0.1265786 0.1526233 0.1713895


r2 <- reproducible_runif(seed = r1$seed) # use the seed from the initial call
r2$x # exactly the same as for r1
# > [1] 0.4215304
```

References: 

- [1](http://r.789695.n4.nabble.com/Best-way-to-reset-random-seed-when-using-set-seed-in-a-function-td918769.html)
- [2](http://r.789695.n4.nabble.com/How-to-properly-re-set-a-saved-seed-I-ve-got-the-answer-but-no-explanation-td4270483.html)
- [3](https://www.uni-muenster.de/ZIV.BennoSueselbeck/s-html/helpfiles/set.seed.html)

## Seeds for parallel 

The story is slightly different when using multiple cores (parallel execution). In this post I use a base package `parallel` and macOS, but the concept is pretty much the same for other packages and non-unix systems. The idea here to run independent simulations on each core.

Before stepping into details, let's consider an illustrative example. We run a classical function `parallel::mclapply()` that returns a random uniform number for each iteration. This function supplies a vector of ten elements as `X` argument, a simple wrapper around `runif(1)` to ignore elements of `X`, the number of cores (in my case 2 physical cores), and also we set `mc.set.seed = FALSE`. We run this expression two times (`unlist` is used for a more compact representation):

```r
library(parallel)

rn1 <- unlist(
    mclapply(X = 1:10,
             FUN = function(x) runif(1),
             mc.cores = 2,
             mc.set.seed = FALSE)
)

rn1
# > [1] 0.3495050 0.3495050 0.4159384 0.4159384 0.5376814 0.5376814 0.3279605
#   [8] 0.3279605 0.1527834 0.1527834

identical(rn1[seq(1, 10, by = 2)], rn1[seq(1, 10, by = 2)])
# > [1] TRUE
```

One can immediately notice a suspicious thing -- every second element equals to the previous one. The explanation is very simple: the same workspace is restored from the master process for each worker (or process). It means that `.Random.seed` will be extracted from the parent process, and therefore, RNG state will be the same for each worker. As result, the same sequence of random numbers will be generated by each of workers.

Of course this issue is not desirable. The alternative method is to have separate (distinct) seeds for each worker. The potential problem would be that the generated numbers might get into steps (i.e. been periodically repeated, therefore, correlated between streams). To resolve this `parallel` package utilizes `"L'Ecuyer-CMRG"` RNG, which has a quite long period with a small seed, ensuring streams do not get into steps easily. To set the RND to `"L'Ecuyer-CMRG"` one runs `RNGkind("L'Ecuyer-CMRG")`, also changing argument `mc.set.seed` of `mclapply` to `TRUE`:

```r
RNGkind("L'Ecuyer-CMRG")

rng1 <- unlist(
    mclapply(X = 1:10,
             FUN = function(x) runif(1),
             mc.cores = 2,
             mc.set.seed = TRUE)
)

rng1

# > [1] 0.67681994 0.54730337 0.05398847 0.19480448 0.94954659 0.35727778
#   [7] 0.17057359 0.83029494 0.37063552 0.24445617
```

Elements now are different, and `"L'Ecuyer-CMRG"` uses `nextRNGStream()` to generate a next "uncorrelated" seed. The pseudo code (taken from `vignette("parallel")`) explains this concept: 

```r
# > RNGkind("L'Ecuyer-CMRG")
# > set.seed(2002) # something 
# > M <- 16 ## start M workers 
# > s <- .Random.seed
# > for (i in 1:M) {
# +     s <- nextRNGStream(s)
# +     # send s to worker i as .Random.seed 
# + }
```

Let's run the same expression one more time: 

```r
rng2 <- unlist(
    mclapply(X = 1:10,
             FUN = function(x) runif(1),
             mc.cores = 2,
             mc.set.seed = TRUE)
)

rng2

# > [1] 0.67681994 0.54730337 0.05398847 0.19480448 0.94954659 0.35727778
#   [7] 0.17057359 0.83029494 0.37063552 0.24445617

identical(rng1, rng2)
# > [1] TRUE
```

The second `rng2` is absolutely identical to `rng1` that was run before. Coincidence? Nope. The thing is the `.Random.seed` of master process is NOT affected by worker processes (see pseudo-code). That is why we will have the same numbers during a second, third, and any other run, unless the `.Random.seed` will be changed (e.g. by `runif(1)` in a master process).


Note that even if `mc.set.seed` is `TRUE`, but RNG is different from `"L'Ecuyer-CMRG"`, then using `set.seed()` won't establish reproducibility.

In the end, the package `parallel` is a little bit vague when it comes to RNG, so that I have to read `vignette("parallel")` (Section 6), dozens of cross-refereed helps (`?mclapply` RNG section refers to `?mcparallel`, which requires to read `?nextRNGStream`), and finally deep dive into sours non-exported function via `parallel:::mc.set.seed`.
