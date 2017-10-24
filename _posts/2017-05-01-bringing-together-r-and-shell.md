---
layout: post
title: "&#128187; Bringing together R and Shell"
---

I believe in our era of RStudio and interactive data analysis, R scripts rarely needed to be run from Shell. The same applies to the opposite: executing Shell commands from R is quite uncommon. However, some cases exist for which this is necessary.

## Invoke a Shell command from R

This could be done by two functions, namely `system` and `system2`. The latter is considered as a more portable one, and supplies a command with command's arguments separately.  In contrast, one has to provide the complete command with its arguments in case of `system`. The main difference between functions is the behavior under different systems (Linux vs Windows) and can be found in official documentation. The following two examples do the same thing using `system`

```r
system("mkdir /Users/irudnyts/Desktop/new_dir")
system("ls /Users/irudnyts/Desktop")
```

and `system2`

```r
system2(command = "mkdir", args = "/Users/irudnyts/Desktop/new_dir")
system2(command = "ls", args = "/Users/irudnyts/Desktop")
```

## Call R script from Shell

Calling R scripts from the command line is pretty straightforward. Command `Rscript` will do the job for you:

```shell
Rscript /Users/irudnyts/Desktop/my_script.R
```

Nice thing about `Rscript` that it's possible to pass arguments to a script from Shell. For this simply type the line below at the beginning of the R script:

```r
## my_script.R
args <- commandArgs(trailingOnly = TRUE)
print(as.numeric(args[1]) + as.numeric(args[2]))
```

Then, pass arguments in the same way as for usual Shell command:

```shell
Rscript /Users/irudnyts/Desktop/my_script.R 100 200
```

This is rather a hint than a complete guide. But for me it was sufficiently enough to save a precious time when running scripts in a row.