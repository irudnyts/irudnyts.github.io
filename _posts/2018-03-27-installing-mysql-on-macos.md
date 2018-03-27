---
layout: post
title: "&#128190; Installing MySQL on MacOS"
---

A couple of days ago I was asked to install MySQL on MacOS 10.13, and I was surprised that it was not a one-click installation, as in case of R. Unfortunately, even for me a documentation was a bit confusing, and I think it might be useful to have a guide of installation. 

## 1. Download .dmg file and install MySQL

One has to download .dmg file from [here](https://dev.mysql.com/downloads/mysql/). The app should be installed like a regular Mac app and the procedure is well covered [here](https://dev.mysql.com/doc/refman/5.6/en/osx-installation-pkg.html).


At the end of installation, when one has reached a summary, a separate windows will pop up with temporary password (as in a screenshot below). This password should be kept somewhere.

![](https://irudnyts.github.io/images/posts/2018-03-27-installing-mysql-on-macos/key.png)


## 2. Set aliases

In order to avoid changing directories all the time before evoking `mysql` we can set aliases for `mysql` and `mysqladmin` commands. To do so one has to open Terminal and execute the following commands (assuming that MySQL was installed to a default folder):

```shell
alias mysql=/usr/local/mysql/bin/mysql
alias mysqladmin=/usr/local/mysql/bin/mysqladmin
```

## 3. Start MySQL sever

Everything should go smooth so far. Now we need to start our sever. One can do it in Terminal:

```shell
cd /Library/LaunchDaemons
sudo launchctl load -F com.oracle.oss.mysql.mysqld.plist
```

or in System Preferences...

![](https://irudnyts.github.io/images/posts/2018-03-27-installing-mysql-on-macos/sys_pref.png)

... by clicking on "Start "

![](https://irudnyts.github.io/images/posts/2018-03-27-installing-mysql-on-macos/start.png)

## 4. Change the temporary password

Now we need to run MySQL to change a temporary password for a 'root' user. After calling the following command, Terminal will ask for a password which we saved when installing MySQL in the first step:

```shell
mysql -u root -p
```

If everything is done correctly, you should see something like this:

```shell
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 24
Server version: 5.7.21

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

To change the password we simply calling this command, where "MyNewPass" as you already guessed is a new password:

```sql
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
```

And then quite MySQL: 

```sql
QUIT
```

## 5. (Optional) Install Sequel Pro IDE for MySQL

I find Sequel Pro a quite useful and beautiful IDE for MySQL. To install it one has to download [a .dmg file](https://sequelpro.com/download#auto-start), open it, and drag & drop "Sequel Pro.app" to applications' folder.

To connect to a local MySQL one has choose Socket in menu and fill in a username (default "root") and the password that we changes in the previous step. 

![](https://irudnyts.github.io/images/posts/2018-03-27-installing-mysql-on-macos/sys_pref.png)

Enjoy!
