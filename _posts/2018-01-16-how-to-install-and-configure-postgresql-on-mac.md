---
layout: post
title: "MacOS使用homebrew安装配置PostgreSQL-10"
date: 2018-01-16 21:51:59 +0800
categories: 环境配置
tags: PostgreSQL Mac brew
author: Qindongliang
---

* content
{:toc}

&emsp;&emsp;平时在MacOS上做开发，经常使用PostgreSQL数据库，每次都远程连接不太方便，所以在Mac上安装PostgreSQL。在Ubuntu上推荐从源代码安装。在Mac上，因为brew相当好用，想尝试使用brew安装。首先确认下，brew下的软件包是不是最新的，使用如下命令：

{% highlight Shell %}
brew info postgresql
{% endhighlight %}




&emsp;&emsp;输出信息如下：
{% highlight Shell %}
postgresql: stable 10.1 (bottled), HEAD
Object-relational database system
https://www.postgresql.org/
Conflicts with:
  postgres-xc (because postgresql and postgres-xc install the same binaries.)
/usr/local/Cellar/postgresql/10.1 (3,392 files, 39.0MB) *
  Built from source on 2017-12-31 at 22:49:17 with: --with-dtrace --with-python3
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/postgresql.rb
==> Dependencies
Required: openssl ✔, readline ✔
==> Requirements
Optional: python ✔, python3 ✔
==> Options
--with-dtrace
	Build with DTrace support
--with-python
	Enable PL/Python2
--with-python3
	Enable PL/Python3 (incompatible with --with-python)
--without-perl
	Build without Perl support
--without-tcl
	Build without Tcl support
--HEAD
	Install HEAD version
==> Caveats
To migrate existing data from a previous major version of PostgreSQL, see:
  https://www.postgresql.org/docs/10/static/upgrading.html

  You will need your previous PostgreSQL installation from brew to perform
  `pg_upgrade` or `pg_dumpall` depending on your upgrade method.

  Do not run `brew cleanup postgresql` until you have performed the migration.

To have launchd start postgresql now and restart at login:
  brew services start postgresql
Or, if you don't want/need a background service you can just run:
  pg_ctl -D /usr/local/var/postgres start
{% endhighlight %}

&emsp;&emsp;可以看到，brew可以安装最新版本的PostgreSQL，并且给出了详细的依赖和安装选项及其使用方法。于是决定就用brew安装PostgreSQL。

## 1. 安装过程

&emsp;&emsp;按照上面列出的信息，安装过程就非常简单了：
{% highlight Shell %}
brew install postgresql --with-dtrace --with-python3
{% endhighlight %}

&emsp;&emsp;安装完成之后使用如下命令设置服务并开机自启：
{% highlight Shell %}
brew services start postgresql
{% endhighlight %}

## 2. 使用brew cask 安装pgAdmin4

&emsp;&emsp;非常简单，使用如下命令即可：
{% highlight Shell %}
brew cask install pgAdmin4
{% endhighlight %}

## 3. 后记
&emsp;&emsp;赞叹brew以及cask是多么地好用！
