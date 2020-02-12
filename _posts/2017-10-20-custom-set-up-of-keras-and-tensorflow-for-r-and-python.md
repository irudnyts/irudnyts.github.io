---
layout: post
title: "&#128013; Custom set up of keras and TensorFlow for R and Python"
---

About a month ago RStudio [published on CRAN](https://blog.rstudio.com/2017/09/05/keras-for-r/) a nice package `keras`. This package is an interface to a famous library `keras`, a high-level neural networks API written in Python for using TensorFlow, CNTK, or Theano. In this post, the focus is on TensorFlow, as default backend engine developed by Google.

![](https://irudnyts.github.io/images/posts/2017-10-20-custom-set-up-of-keras-and-tensorflow-for-r-and-python/r_python.png)

Even though RStudio is not the first who developed such an interface (see [kerasR](https://cran.r-project.org/web/packages/kerasR/index.html)), they usually build robust and stable tools and software. Official [documentation](https://keras.rstudio.com) shows a pretty straight-forward way how to install and use the package. However, the installation procedure assumes to use Python 2.7 (default Python on macOS). As long as I also use Python and prefer to use 3.6 I decided to write a little guide for installation all the data science tools related to Python.

I assume that the user has a macOS system and preinstalled `gcc` (GNU Compiler Collection). During its first start RStudio propose to install `gcc`. To check if it is installed simply type in terminal 

```shell
gcc --version
```

which should return a version or `command not found` otherwise. No other prerequisites are needed.

## 1. Install Python 3
First, a package manager [Homebrew](https://brew.sh/) :beer: should be installed. No worries if you are not very familiar what's that. Simply execute this command in Terminal: 

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Then, we need to place Homebrew directory at the top of `PATH` environment variable. Wait, what?! OK, just execute this command to open command line text-editor `nano`

```shell
nano ~/.bash_profile
```

and paste this line `export PATH=/usr/local/bin:$PATH`. Close and open Terminal or execute 

```shell
source ~/.bash_profile
```
to make these changes active. Finally, actually install Python 3 by: 

```shell
brew install python3
```

To check if it's installed properly, type

```shell
python3 --version
```
which should return a version of Python. Nice thing that Homebrew installs `pip3` (a package manager for Python libraries) automatically.

## 2. Install TensorFlow

I personally prefer to install TensorFlow with "native" pip approach instead of virtualenv, since I do not have many projects relying on different packages' versions. Thus, to install TensorFlow it's enough to 

```shell
pip3 install tensorflow
```

## 3. Install keras

It is also as simple as installing TensorFlow:

```shell
pip3 install keras
```

should do the trick. 

## 3.1. Bonus: Install Rodeo IDE

I am addicted to sweet RStudio IDE. I had several attempts of trying Python, but I was always disappointed and pissed off by ugly IDE. Until I discovered Rodeo. Alright, maybe it's a bit buggy, but at least it does not look like Windows XP-ish retired lady Spyder. Guys, let's have a bit of a taste :innocent:

The easiest way to install Rodeo is to download and install it from `.dmg` file. During the first start-up Rodeo probably will ask to install several required packages. Since we did not install Anaconda distribution, we need to do it manually:

```shell
pip3 install jupyter_client ipykernel numpy pandas matplotlib jupyter
```

Then, Rodeo will ask to specify Python command. It is enough to insert `python3` to the input box.

## 4. Install and configure R package keras

There should not be any problems to install the package by a standard way from CRAN:

```r
install.packages("keras")
```

Standard installation procedure assumes, then, install Keras and TensorFlow by `install_keras()`. However, we have already installed these guys in conjunction with Python 3. Instead, we use alternative way of installation suggested by [this page](https://tensorflow.rstudio.com/tools/installation.html), i.e. locate TensorFlow and Python. Type 

```r
use_python("/usr/local/bin/python3")
```
to specify a path to Python 3 (if you are not sure about this path, you can always check it out by executing in Terminal `which python3`). Restart RStudio. Load `keras` library and get your hands dirty. Voil&agrave;! Unless you screwed up somewhere (which is usually the case for me), everything should work nicely.

Sources: [1](http://docs.python-guide.org/en/latest/starting/install3/osx/), [2](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-macos#step-6-—-creating-a-simple-program), [3](http://www.marinamele.com/2014/07/install-python3-on-mac-os-x-and-use-virtualenv-and-virtualenvwrapper.html), [4](https://www.tensorflow.org/install/install_mac), [5](https://keras.io)

Picture by [Statistical Statistics Memes](https://www.facebook.com/statsmemes/)
