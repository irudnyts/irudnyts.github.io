---
layout: post
title: 5-minutes data science&#58; Dortmund real estate market analysis &#40obtaining and tidying data&#41
---

Back in 2013 I spent two amazing months of my life in Dortmund. Taking into account that a number of my friends who moved (are moving) to Germany is increasing, I thought it would be nice to get an insight of the [last imperfect market](http://www.bbc.com/news/business-34531638) of real estate in Dortmund.

![](https://irudnyts.github.io/images/posts/2017-04-25-Dortmund-real-estate-market-analysis/monopoly.png)

The largest listning of property in Germany is [immobilienscout24](https://www.immobilienscout24.de), which is used as a data source. By utilizing the power of [rvest](https://blog.rstudio.org/2014/11/24/rvest-easy-web-scraping-with-r/) package and [SelectorGadget](https://cran.r-project.org/web/packages/rvest/vignettes/selectorgadget.html) (I totally recommend to check out these easy-readable links), it is possible to scrap the data from immobilienscout24. The following code simply loads packages, defines a function that checks whether or not the lengths of its arguments are the same. We will need this function later on. Then, `urls` are defined by pasting numbers of pages between the first and second parts of urls. Furthermore, we define the data frame `property` where the data will be stored.

```r
options(stringsAsFactors = FALSE)
library("rvest")
library("stringr")

rm(list = ls())

equal_lengths <- function(...) {
    sizes <- sapply(X = list(...), FUN = length)
    return(all(rep(sizes[1], length(sizes)) == sizes))
}

url_part1 <- "https://www.immobilienscout24.de/Suche/S-T/P-"
url_part2 <- "/Wohnung-Miete/Nordrhein-Westfalen/Dortmund?pagerReporting=true"
pages <- 1:44

urls <- paste0(url_part1, pages, url_part2)

property <- data.frame(price = character(),
                       area = character(),
                       rooms = character())
                       
```

At this step it seems natural to use a `for` loop, and go over each link. Unfortunately, guys from immobilienscout24 use rate limiting (I am not an expert, but it seems like). Thus, I figured out that we need to download the data in `while` loop instead, for which the condition to stop would be the length of urls equals to 0. Inside the loop each page is parsed, and we extract the following information about entries: `price`, `area` (living space), `rooms`, and `address`. If the rate limiting is triggered, then a part of data is lost, and therefore, the lengths of these variables will be different. Conditioning on the length equality, the content of variables is copied to `property`, and the processed url is removed from `urls`.

The `rvest` package was designed to work in conjunction with `magrittr` package, allowing for usage of the so-called [pipe](https://www.r-bloggers.com/why-bother-with-magrittr/) (also worth reading).

In this post we won't use the address variable for simplification. Of course, prices of rents strongly depend on the district, which will be a subject of further posts later on. Now we can enjoy smooth code with commented parts for addresses. 

```r
while(length(urls) > 0) {
    
    link <- urls[1]
    
    print(link)
    immo <- read_html(link)
    
    price <- immo %>% 
        html_nodes(".result-list-entry__primary-criterion:nth-child(1) .font-line-xs") %>%
        html_text()
    
    area <- immo %>% 
        html_nodes(".result-list-entry__primary-criterion:nth-child(2) .font-line-xs") %>%
        html_text()
    
    rooms <- immo %>% 
        html_nodes(".result-list-entry__primary-criterion:nth-child(3) .font-line-xs") %>%
        html_text()
    
    # add <- immo %>% 
    #     html_nodes(".margin-bottom-xs") %>%
    #     html_text()
    
    # address <- immo %>% 
    #     html_nodes("#resultListItems .link-underline") %>%
    #     html_text()
    
    if(equal_lengths(price, area, rooms)) {
        property <- rbind(property, 
                          data.frame(price, area, rooms))
        urls <- urls[-1]
        
    }
}
```

Having collected all the required data, we need to tidy it and coerce to appropriate formats. The desired strings contain in the end units (e.g. euro currency sign or square meter signs). We get rid of them using built-in function `gsub`. Also we need to drop `.` as thousands separator, and replace `,` with `.` for a decimal point. Finally, we can convert characters to numerics.

```r
property$price <- property$price %>% 
    gsub(pattern = " \u20AC", replacement = "", fixed = TRUE) %>%
    gsub(pattern = ".", replacement = "", fixed = TRUE) %>% 
    gsub(pattern = ",", replacement = ".", fixed = TRUE) %>%
    as.numeric()

property$area <- property$area %>%
    gsub(pattern = " m\u00B2", replacement = "", fixed = TRUE) %>%
    gsub(pattern = ",", replacement = ".", fixed = TRUE) %>%
    as.numeric()

property$rooms <- property$rooms %>%
    gsub(pattern = ",", replacement = ".", fixed = TRUE) %>%
    as.numeric()
```

Finally, removing `NA`s yields nice and clean data, on which we will calibrate our models in further post.

```r
property <- property[complete.cases(property), ]

```

The source file is available at my [gist](https://gist.github.com/irudnyts/9919fd110dabeea41c12894f2275adf9) and [data](https://gist.github.com/irudnyts/ec2a2af812d7b23b26294b01181d8791) downloaded on 2017 September 8.

Update: how could I miss that immobilienscout24 has an [API](https://api.immobilienscout24.de/)? However, it's not yet clear whether the time spent learning API would take less than writing this code.