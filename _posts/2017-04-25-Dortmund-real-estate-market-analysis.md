---
layout: post
title: 5-minutes data science&#58; Dortmund real estate market analysis
---

Back in 2013 I spent two amazing months of my life in Dortmund. Taking into account that a number of my friends who moved (are moving) to Germany is increasing, I thought it would be nice to get an insight of the [last imperfect market](http://www.bbc.com/news/business-34531638) of real estate in Dortmund.

![](http://static4.businessinsider.com/image/54c26cf9eab8ead5409e2908-1190-625/80-things-you-probably-dont-know-about-the-monopoly-board-game.jpg)

The largest listning of property in Germany is [immobilienscout24](https://www.immobilienscout24.de), which is used as a data source. By utilizing the power of [rvest](https://blog.rstudio.org/2014/11/24/rvest-easy-web-scraping-with-r/) package and [SelectorGadget](https://cran.r-project.org/web/packages/rvest/vignettes/selectorgadget.html) (I totally recommend to check out these easy-readable links), it is possible to scrap the data from immobilienscout24. The following code simply loads packages, defines a funciton that checks whether or not the lengths of its arguments are the same. We will need this function later on. Then, `urls` are defined by pasting numbers of pages between the first and second parts of urls. Furthemore, we define the data frame `property` where the data will be stored.

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
pages <- 1:37
    
urls <- paste0(url_part1, pages, url_part2)

property <- data.frame(price = character(),
                       area = character(),
                       rooms = character(),
                       address = character())
                       
```

At this step it seems natural to use a `for` loop, and go over each link. Unfortunately, guys from immobilienscout24 use rate limiting (I am not an expert, but it seems like). Thus, I figured out that we need to download the data in `while` loop instead, for which the condition to stop would be the length of urls equals to 0. Inside the loop each page is parsed, and we extract the following information about entries: `price`, `area` (living space), `rooms`, and `address`. If the rate limiting is triggered, then a part of data is lost, and therefore, the lengths of these variables will be different. Conditioning on the length equality, the content of variables is copied to `property`, and the processed url is removed from `urls`.

The `rvest` package was designed to work in conjuntion with `magrittr` package, allowing for usage of the so-called [pipe](https://www.r-bloggers.com/why-bother-with-magrittr/) (also worth reading).

```r

while(length(urls) > 0) {
    
    link <- urls[1]
    # print(link)
    
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
    
    address <- immo %>% 
        html_nodes(".nine-tenths .font-s") %>%
        html_text()

    if(equal_lengths(price, area, rooms, address)) {
        property <- rbind(property, 
                          data.frame(price, area, rooms, address))
        urls <- urls[-1]
    }
}

```

Having collected all the required data, we need to tidy it. The desired values are wrapped into ugly strings of spaces and slashes. By using the fucntion `str_match` of `stringr` package and regular expressions we squeeze desired values and coerce them from characters to numerics (also changing `,` to `.` for decimal point):

```r
property$price <- str_match(string = property$price,
                            pattern = "\n                (.*?) €")[, 2] %>%
    gsub(pattern = ",", replacement = ".") %>% 
    as.numeric()

property$area <- str_match(string = property$area,
                           pattern = "\n                (.*?) m²")[, 2] %>%
    gsub(pattern = ",", replacement = ".") %>%
    as.numeric()

property$rooms <- str_match(string = property$rooms,
                            pattern = "\n            \n              \n              \n                (.*?)\n              \n            \n            \n              Zi.\n            \n          ")[, 2] %>%
    gsub(pattern = ",", replacement = ".") %>%
    as.numeric()
```

There is no such a complication with addresses. It is reasonable to split objects into disctricts of Dortmund in order to use them as categorical predictors. Fortunatelly, addresses already contain the district name, the only thing remained to do was to cut the city and the street on each side:

```r
property$part <- str_match(string = property$address,
                           pattern = "(?<=, )(.+?),")[, 2]
```

Finally, removing `NA`s and running regression we obtaine the result:

```r
property <- property[complete.cases(property), ]

regression <- lm(formula = price ~ area + rooms + part, data = property)
summary(regression)
```

```r
> summary(regression)

Call:
lm(formula = price ~ area + rooms + part, data = property)

Residuals:
    Min      1Q  Median      3Q     Max 
-567.81 -103.93  -18.19   92.55  663.64 

Coefficients:
                      Estimate Std. Error t value Pr(>|t|)    
(Intercept)           443.3267    75.6235   5.862 8.36e-09 ***
area                   -1.4884     0.4761  -3.126  0.00188 ** 
rooms                  65.1908    15.5173   4.201 3.15e-05 ***
partAplerbecker Mark -141.2787   116.7578  -1.210  0.22685    
partAsseln           -119.3271   101.7316  -1.173  0.24138    
partBarop            -151.3410   110.7633  -1.366  0.17245    
partBerghofen        -182.7808   101.8378  -1.795  0.07329 .  
partBittermark         34.6048   105.4699   0.328  0.74297    
partBodelschwingh    -142.9589    98.6854  -1.449  0.14807    
partBövinghausen     -119.6720   101.6747  -1.177  0.23976    
partBrackel           -21.6105    96.2813  -0.224  0.82250    
partBrechten         -110.4749   117.0637  -0.944  0.34578    
partBrünninghausen   -229.1042   221.3155  -1.035  0.30109    
partDerne             -45.5897   101.6726  -0.448  0.65406    
partDorstfeld         -86.3573    81.4394  -1.060  0.28949    
partEichlinghofen     -69.8952   101.8562  -0.686  0.49290    
partEving            -140.2674    96.1653  -1.459  0.14531    
partHacheney         -337.1148   165.0815  -2.042  0.04167 *  
partHermannstr.25     -94.1003   220.7104  -0.426  0.67004    
partHöchsten           28.4088   163.7371   0.174  0.86233    
partHolte            -157.0026   220.5757  -0.712  0.47693    
partHolzen             -7.4759   110.3032  -0.068  0.94599    
partHombruch          -38.4546    94.1354  -0.409  0.68308    
partHörde            -184.0244    79.0112  -2.329  0.02026 *  
partHostedde         -189.8682   220.8087  -0.860  0.39027    
partHuckarde          -37.6234   116.7250  -0.322  0.74734    
partHusen            -223.7983   220.9160  -1.013  0.31153    
partInnenstadt        -46.5747    71.5722  -0.651  0.51552    
partKirchderne       -101.8915   220.7812  -0.462  0.64464    
partKirchhörde       -101.9276    96.8586  -1.052  0.29316    
partKirchlinde         43.3669   139.6549   0.311  0.75629    
partKleinholthausen  -145.6172   221.9201  -0.656  0.51202    
partKley             -152.0055   105.5547  -1.440  0.15048    
partKörne              -2.2295    84.9296  -0.026  0.97907    
partLanstrop         -159.6905   101.7698  -1.569  0.11726    
partLindenhorst       -62.3053   163.5996  -0.381  0.70349    
partLoh               -27.3892   220.5940  -0.124  0.90124    
partLücklemberg      -337.4220   104.0199  -3.244  0.00126 ** 
partLütgendortmund   -109.9575    85.5038  -1.286  0.19905    
partMarten            -19.2578    94.0885  -0.205  0.83791    
partMengede          -113.8495   110.2794  -1.032  0.30240    
partMenglinghausen   -206.2400   221.0335  -0.933  0.35124    
partNette              58.9445   116.7648   0.505  0.61391    
partNeuasseln         -41.2169   139.5156  -0.295  0.76779    
partNiederhofen      -133.3651   140.4491  -0.950  0.34280    
partOespel            112.5005   220.8365   0.509  0.61068    
partOestrich          -88.8599   101.7142  -0.874  0.38275    
partPersebeck        -113.3202   139.5304  -0.812  0.41709    
partRenninghausen     240.0315   164.3887   1.460  0.14489    
partSchanze           410.9222   220.7088   1.862  0.06322 .  
partScharnhorst      -150.2160   125.9257  -1.193  0.23348    
partSchnee           -312.1277   238.0191  -1.311  0.19035    
partSchüren          -157.9926    89.0735  -1.774  0.07672 .  
partSölde             -75.6235   116.7063  -0.648  0.51730    
partSommerberg          6.2761   220.6487   0.028  0.97732    
partSyburg              1.0166   221.3594   0.005  0.99634    
partWambel            124.5736   110.3359   1.129  0.25943    
partWellinghofen      -86.8161   111.2234  -0.781  0.43544    
partWesterfilde      -104.8225   125.7529  -0.834  0.40493    
partWestrich          -85.8633   220.5658  -0.389  0.69723    
partWichlinghofen    -143.4139   127.3170  -1.126  0.26053    
partWickede          -119.1238    83.4495  -1.427  0.15407    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 209.2 on 494 degrees of freedom
Multiple R-squared:  0.1822,	Adjusted R-squared:  0.08127 
F-statistic: 1.805 on 61 and 494 DF,  p-value: 0.0003824
```
What can be outlined from regression analysis? Well, not much. Only around 18 % of the variance is explained by such variables, based on value of R-squared. The intercept is significant and has a value around 443 euros, that is a kind of base price for apartments. The positive and significant value of the coefficeint for `rooms` coincides with intuition: with additional room we have to pay 65 euros more (even though, from my view, it is rather low). What I do not like is the `area` coefficient, which significant and negative, which means with one square meter increase, an apartment is 1.6 euro cheaper to rent. The vast majority of district levels are negative, which also seem strange.

To dive deeper I suggest to use `glm` assuming non-Gaussian residuals. Furthermore, there are dozens of other features that drive the price, like a floor, when an apartment was renovated etc. 

The source file is available at my [gist](https://gist.github.com/irudnyts/9919fd110dabeea41c12894f2275adf9).

Update: how could I miss that immobilienscout24 has an [API](https://api.immobilienscout24.de/)? 