---
layout: post
title: "&#128230; Managing dependencies in packages"
---

Managing usual dependencies of a package is clearly covered in [R packages by Hadley Wickham](http://r-pkgs.had.co.nz). Typically, that would be the end of a tutorial or a post. However, teaching recently how to develop a package, I encountered a couple of super interesting and non-trivial questions that would not have a conventional solution. I guess this post would be a perfect place to share my thoughts on that meter, as well as a nice excuse to restart blogging.

![](https://irudnyts.github.io/images/posts/2019-12-03-managing-dependencies-in-packages/lego.png)

## Non-CRAN packages

When developing the package, the standard place to list dependencies (i.e., external packages that your package needs) is `Imports:` in `DESCRIPTION`. Full stop here. These packages are required to be installed so that your package works. And they will be installed automatically when installing your package via `install.packages()` (see default behavior of `dependencies` argument). However, packages in `Imports:` field are supposed to be published on CRAN. That could be an issue if your package uses functionality from packages that are not (yet) published on CRAN. This is the exact question I was asked by one of my students: where do I specify non-CRAN dependencies? 

I was sure that there exists a common workflow to do it. After a minute of extensive research, I found out that CRAN policy explains it quite vaguely. Further, there were three Stackoverflow questions about it (see below in References). The answer that I found was quite satisfactory: Dirk Eddelbuettel proposes to list the package in `Sugests:` and specify the additional repository in special free-form filed `Additional_repositories:`. He also suggests using `drat` package to create CRAN-like R packages repository, which from my view is a bit overkill. So my solution would be to list the name of the package in `Suggests:` and mention the link to its GitHub repo (almost surely the source is stored on GitHub) in `Additional_repositories:`. 

**Update:** As it was kindly pointed out by Sébastien Rochette, `devtools` supports a `Remotes:` field exactly for that purpose. Simply specify the repos in the format `username/reponame` separated by commas (one can also add the type of the source if it is not GitHub, e.g., `gitlab::username/reponame`). And that is it.

That would be the nice end of the story but how would you let know the end-user that you need this package to be pre-installed? The workaround I found is to rise a message from the function, where this dependence is used and ask the user to install it, for example:

```r
my_function <- function() {
    if (!("nonCRANpkg" %in% rownames(installed.packages()))) {
        message("Please install package nonCRANpkg.")
    }
}
```

The problem is that the user should come back to the installation process at the point when they use `my_function() `. In addition, it probably affects the expected output of the function or even worse if the function is internal one and not exported into the namespace. That is why, from my personal view, the installation of all dependencies should be tackled way before the first call of `my_function()`. And here the function `.onAttach()` comes in handy. This function allows displaying messages when the package is loading. We simply need to inform the user that they need to install the dependence before using our package (mind the difference between `message()` and `packageStartupMessage()`): 

```r
.onAttach <- function(libname, pkgname) {
    
    if (!("nonCRANpkg" %in% rownames(installed.packages()))) {
        packageStartupMessage(
            paste0(
                "Please install `nonCRANpkg` by",
                " `devtools::install_github('username/nonCRANpkg')`"
            )
        )
    }
    
}
```

To summarize in a nutshell: mention the package name in the field `Suggests:` of `DESCRIPTION`, link to its repo in `Remotes:` (in the same file), and write a simple `.onAttach()` function (it should be stored in the file `zzz.R`).

## Shiny demo app 

It is always a cool idea to compliment the package with a Shiny app so that a user can have an interactive interface to play around with the functionality of the package. We typically store scripts of those demo apps in `inst\shiny-examples\name_of_app` and add a function `runDemo()` to run them (see a wonderful post by Dean Attali in the references for details). Those apps are very likely to have their own dependencies, as well as they definitely require `shiny` namespace to be loaded. That is why we see all these `library()` calls at the beginning of Shiny apps' scripts. 

Obviously, (1) we want to ensure that the user has all required packages installed, and (2) avoid using `library()` in package's scripts. The solution is very simple -- specify all Shiny app dependencies in `Imports:` and use the usual `::` to access functions from respective namespaces. 

To sum up all the previous take-home points, I created a `dummypkg` for illustration, which is stored at GitHub repo [`irudnyts\dummypkg`](https://github.com/irudnyts/dummypkg). It contains a barebone example of non-CRAN dependencies, as well as a tiny Shiny app with dependencies. Managing those dependencies is super important since we do not want our packages to look like jack-in-the-boxes.

Many thanks go to Ana Lucy Bejarano Montalvo who inspired me by asking those questions and Sébastien Rochette for pointing out `Remotes:` filed. 

## References 
1. [R packages by Hadley Wickham](http://r-pkgs.had.co.nz)
2. [Devtools dependencies](https://cran.r-project.org/web/packages/devtools/vignettes/dependencies.html)
3. [Include non-CRAN package in CRAN package](https://stackoverflow.com/questions/33335321/include-non-cran-package-in-cran-package)
4. [R package building: How to import a function from a package not on CRAN](https://stackoverflow.com/questions/43773066/r-package-building-how-to-import-a-function-from-a-package-not-on-cran)
5. [How to make R package recommend a package hosted on GitHub?](https://stackoverflow.com/questions/36105257/how-to-make-r-package-recommend-a-package-hosted-on-github?rq=1)
6. [R package dependencies not installed from Additional_repositories
](https://stackoverflow.com/questions/29419776/r-package-dependencies-not-installed-from-additional-repositories)
7. [Supplementing your R package with a Shiny app](https://deanattali.com/2015/04/21/r-package-shiny-app/)
8. [Lego PNG image with transparent background](http://pngimg.com/download/51459)
9. [A Framework for Building Robust Shiny Apps](https://github.com/ThinkR-open/golem)