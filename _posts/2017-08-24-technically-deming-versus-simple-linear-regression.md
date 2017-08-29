---
layout: post
title: Technically&#58; Deming versus simple linear regression
---

All courses that somehow covered regression models were starting almost in the same way: given bunch of $y$'s and $x$'s points, one needs to predict a value of $y$ for a certain $x$. Sounds quite easy. Without utilizing any statistical assumptions, we can just find the line, which is in a way closest to those points. So the model is as follows:

$$
y \approx \beta_0 + \beta_1 x
$$

Then typically a professor of a course leads to idea of minimizing the distances between observed variables and the fitted ones, i.e.:

$$
\sum_i=1^n(y_i - (\beta_0 + \beta_1 x_i))  
$$

But those distances might be both positive or negative. Instead, we can use the absolute values, but squaring those distances leads to much nicer properties for optimization:

$$
\sum_i=1^n(y_i - (\beta_0 + \beta_1 x_i))^2  
$$

Minimizing this term yields so-called linear or more commonly ordinary least squares (OLS) estimates.

So far, we did not use any of statistical assumptions. Of course due to world imperfection the relation between $y$ and $x$ is not exact and incorporates errors. These errors are modeled by presence of randomness. Let's define the other model, which includes a random term referred as error or disturbance term: 

$$
y = \beta_0 + \beta_1 x + \epsilon
$$

Since we know the theoretical distribution of $y$ and have a sample, we can use a powerful machinery of maximum likelihood estimation (MLE) method. It turns out that with several assumptions (homoscedastic, normal and uncorrelated errors) MLE is identical (!) to OLS (see Bradley 1973). I do not about you, but this fact shows the whole beauties of mathematics to me.

So far so good, but let's step back for a second and check ourselves. Minimizing the distances between observed variables and the fitted ones sounds very good, but wait a second... That's how the vertical distance defined, not the orthogonal. The minimal distance between a point and a line is actually orthogonal distance, not the vertical one. It means that OLS minimizes vertical distances instead of orthogonal, which is illustrated below.

![](https://irudnyts.github.io/images/v_vs_o.png)

Minimizing the square orthogonal distances yields so-called Deming regression estimates, which is identical to MLE of errors-in-variables model with $\sigma = 1$. This model is used mostly in clinical chemistry. 











As a part of my PhD program I have to attend the [summer school](http://saa-iss.ch) organized by our department. During this summer school Prof. Braun (one of speakers) mentioned a super nice [resource](http://www.artemis.bm/) of catastrophe bonds (cat bonds) & insurance-linked securities (ILS). It provides the information, such as the size, the trigger etc. about most of ILS.

Apart from the information in the [large table](http://www.artemis.bm/deal_directory/), the column "Issuer" contains clicable elemtents, from where one can get an extended information about one or the other ILS. The script conceptually is orginized quite simply: (1) get all links from the table, and (2) go over each link and extranct a contract's data. As in post about [Dortmund real estate](https://irudnyts.github.io/Dortmund-real-estate-market-analysis/) we use the `rvest` package for web-scrapping (read that post if you are not very familiar with web-scrappling).

```r
# load packages
library("rvest")
library("stringr")

# define links
base_url <- "http://www.artemis.bm"
url <- "http://www.artemis.bm/deal_directory/"

# get links from Issuer column
links <- paste0(base_url,
                read_html(url) %>%
                    html_nodes(".table-style01 a") %>%
                    html_attr("href"))
```

The cool thing about these ILS individual "pages" is that they use the same format. I.e. each page orginized as a bullited list, where the beggining of the line is the description of the variable (bold), and after the column the value follows. For instnace, for [Spectrum Capital Ltd. ](http://www.artemis.bm/deal_directory/spectrum-capital-ltd-series-2017-1/) special purpose vehicle (4th from the top) the size is $430m, which comes after word **Size:**. Thus, we need to keep CSS selectors for each element of the list, as well as the beggining of the phraze, which we will cut in order to get only the value. We also need to define the function which will extract the text from the CSS selector, and cut the start pharze:

```r
selectors <- paste0("#perex li:nth-child(", 1:9, ")")

start_phrazes <- c("Issuer / SPV: ",
                   "Cedent / Sponsor: ",
                   "Placement / structuring agent/s: ",
                   "Risk modelling / calculation agents etc: ",
                   "Risks / Perils covered: ",
                   "Size: ",
                   "Trigger type: ",
                   "Ratings: ",
                   "Date of issue: ")
                   
extract <- function(selector, link, start_phraze) {
    x <- read_html(link) %>% html_node(selector) %>% html_text()
    sub(pattern = start_phraze, replacement =  "", x = x, fixed = TRUE)
}
```

It's naturall to keep all the data in `data.frame`, where rows are ILS, and columns are variables. Finally, the data is downloaded in two nested loops: the first is to go over all securities, and the second one is to extract the each element (CSS selector) of bulited list. Oh, and let's not forget to spruce up the column names of the data frame :)

```r
ils <- data.frame(matrix(data = NA,
                         nrow = length(links),
                         ncol = length(start_phrazes)))
                         
# CAUTION: it's time-consuming, as long as it has to load 479 web-pages
for(i in seq_along(links)) {
    for(j in seq_along(selectors)) {
        ils[i, j] <- extract(selector = selectors[j],
                             link = links[i],
                             start_phraze = start_phrazes[j])
    }
}

colnames(ils) <- tolower(start_phrazes) %>%
    gsub(pattern = " ", replacement =  ".") %>%
    sub(pattern = "(./|:.).*", replacement =  "")
```

Tha data is downloaded (hopefully). In the second part I will tidy it and draw a couple of interesting facts about cat bonds.