Deploying Hugo website on Project Pages in 5 minutes



Prerequisits: 
- Installed R, RStudio, `blogdown` package, and site generator Hugo
- Existing website [](http://username.github.io/) (e.g., see [https://www.jekyllnow.com](https://www.jekyllnow.com))


* Initialize an empty repo with a nice name 
* Create a new RStudio project and call it with the same name (do not forget to check the option to intialize the git repo)
* In RStudio: 
    - `library(blogdown)`
    - `blogdown::new_site(theme = "matcornic/hugo-theme-learn")` (I wanted to use [Hugo-theme-learn](https://learn.netlify.com/en/), which is (availible from GitHub)[https://github.com/matcornic/hugo-theme-learn])
    If you are using the RStuio project, it automatically initialize the web-page at that folder.
    - Commit
    - Add remote and push changes 
    	git remote add origin git@github.com:irudnyts/deep.git
	git push -u origin master
	
	
	    - Change baseURL to `baseURL = "https://username.github.io/project/"`

So far so good,
There are several ways how to deploy a website using GitHub services. Since Hugo is not supported by GitHub (only Jekyll does), [Yihui Xie](https://bookdown.org/yihui/blogdown/github-pages.html#fnref39) propose two solutions: either initialize the repo under `/public` or separate files into two folder, namely `source/` and `username.github.io/`. For the former case, one does not store all the scripts that generate the website, while the second solution would work only to GitHub Pages (i.e., with `username.github.io/` domain, but not with subdomain of `username.github.io/custom`).

The other solution is to change the location from which the website is built. [Project Pages](https://help.github.com/en/articles/user-organization-and-project-pages) allows for building directly from the `master` branch, `/docs` folder of master branch, or from the `gh-pages` branch.  We stick to the second way, which is nicely explained [here](https://gohugo.io/hosting-and-deployment/hosting-on-github/). By default, Hugo generates the website into `\public`, however, the default behaviour can be overrriden by using `publishDir = "docs"` in the configuration file `config.toml`. 

* Add `.nojekyll` file by `blogdown::file.create('.nojekyll')`
* Add `publishDir = "docs"` into `config.toml`
* Delete `\public` folder
* Serve Site
* `commit` and `push`


Go to your repo in GitHub, open Settings, navigate to section GitHub Pages and select master branch /docs folder
