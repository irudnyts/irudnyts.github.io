---
layout: post
title: Regression: two fold model
---

Workflow: 

1. Install the development version of `httpvu`

```r
devtools::install_github("rstudio/httpuv")`
```

2. Clone https://github.com/rstudio/websocket from GitHub by

```shell
git clone https://github.com/rstudio/websocket.git
```

3. Open `.rproj` file of `websocket` in RStudio, and click `Install and Restart`

4. In console run: 

```shell
R -e 'source("tmp/websocketServer.R"); httpuv::service(Inf)'
```

5. Install `jsonlite`

```r
install.packages("jsonlite")
```

6. Register new API Token at https://www.binary.com/en/user/security/api_tokenws.html

7. Register an app at https://developers.binary.com/applications/
