---
layout: post
title: "Ubuntu 16.04 从源代码安装 PostgreSQL"
date: 2018-01-01 23:45:27 +0800
categories: 环境配置
tags: PostgreSQL Ubuntu
author: Qindongliang
---


如果是在Windows上安装，可以直接从官网下载最新版的二进制安装包；如果是在MacOS上安装，最佳方式是[使用Homebrew安装最新版]({% post_url 2018-01-16-how-to-install-and-configure-postgresql-on-mac %})。

Linux下发行版本可以使用相应版本的包管理器来进行安装，但是一般默认的的安装都不是最新版本，想要安装最新版可以官网下载最新的二进制安装包，但是如果给远程服务器安装的话，这种方式就不太方便了。所以本文记录的是以源代码方式给远程Ubuntu16.04服务器安装最新版本的PostgreSQL。




## 1. 编译安装过程简短介绍

- 第一步：下载最新源代码。
- 第二步：编译安装。遵循通常的三步：
  + ./configure
  + make
  + make install
- 第三步：安装完成后的步骤
  + 使用initdb命令初始化数据库簇
  + 启动数据库实例
  + 设置开机自启动PostgreSQL服务
  + 创建应用需要的数据库（可选）

## 2. 安装详细步骤

#### 2.1 下载最新源代码并解压
登录[PostgreSQL官网](https://www.postgresql.org/)，选择[Download](https://www.postgresql.org/download/),然后选择左侧的[Source](https://www.postgresql.org/ftp/source/)，找到对应的最新版源代码。复制最新版源代码的[下载链接](https://ftp.postgresql.org/pub/source/v10.1/postgresql-10.1.tar.gz)。
- （1）. 使用ssh登录到远程的VPS主机（Ubuntu16.04.3）。
{% highlight Shell %}
ssh root@xxx.xxx.xxx.xxx
{% endhighlight %}
- （2）. 切换路径到`/usr/local/src/`。
{% highlight Shell %}
cd /usr/local/src
{% endhighlight %}
- （3）. 使用wget下载源代码。
{% highlight Shell %}
wget https://ftp.postgresql.org/pub/source/v10.1/postgresql-10.1.tar.gz
{% endhighlight %}

输出信息如下：

{% highlight Shell %}
    --2018-01-01 07:05:53--  https://ftp.postgresql.org/pub/source/v10.1/postgresql-10.1.tar.gz
    Resolving ftp.postgresql.org (ftp.postgresql.org)... 174.143.35.246, 217.196.149.55, 87.238.57.227, ...
    Connecting to ftp.postgresql.org (ftp.postgresql.org)|174.143.35.246|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 25905595 (25M) [application/x-gzip]
    Saving to: 'postgresql-10.1.tar.gz'

    postgresql-10.1.tar.gz       100%[===========================================>]  24.71M  21.9MB/s    in 1.1s

    2018-01-01 07:05:54 (21.9 MB/s) - 'postgresql-10.1.tar.gz' saved [25905595/25905595]
{% endhighlight %}

- （4）. 使用tar解压。
{% highlight Shell %}
tar -xvf postgresql-10.1.tar.gz
{% endhighlight %}

#### 2.2 编译及安装

- （1）. 需要提前安装的软件包：

  + gcc
  + make

  + zlib1g-dev
默认情况下，安装会用到数据库中的压缩功能。[官网介绍](https://www.postgresql.org/docs/10/static/install-requirements.html)：The zlib compression library is used by default. If you don't want to use it then you must specify the --without-zlib option to configure. Using this option disables support for compressed archives in pg_dump and pg_restore.

  + libreadline6-dev
如果想在psql中方便地使用上下键翻查命令的功能，按照官方手册的说明，需要使用readline开发包。[官网介绍](https://www.postgresql.org/docs/10/static/install-requirements.html)：The GNU Readline library is used by default. It allows psql (the PostgreSQL command line SQL interpreter) to remember each command you type, and allows you to use arrow keys to recall and edit previous commands. This is very helpful and is strongly recommended. If you don't want to use it then you must specify the --without-readline option to configure.

  + libperl-dev
最好使用。

  + python-dev
最好使用。

  + libssl-dev
为了可以设置远程ssh访问，需要ssl开发包。


如果不是以root用户登录的。需要在以下命令前加上`sudo`

{% highlight Shell %}
apt-get install zlib1g-dev

apt-get install libreadline6-dev

apt-get install libperl-dev

apt-get install python-dev

apt-get install libssl-dev
{% endhighlight %}

- （2）编译安装及说明

  + 进入解压后的源代码目录。&emsp;&emsp;&emsp;&emsp;`root@ubuntu-512mb-sfo1-01:/usr/local/src# cd postgresql-10.1/`
  + [官网源代码安装说明](https://www.postgresql.org/docs/10/static/install-procedure.html)中对每个配置选项有详细的说明。

  + configure命令&emsp;&emsp;&emsp;&emsp;使用--prefix指定PostgreSQL的安装路径。完整的命令如下：`./configure --prefix=/usr/local/pgsql/10.1/ --with-perl --with-python --with-openssl `
  + build &emsp;&emsp;&emsp;&emsp;包括文档和扩展模块在内的所有内容，使用如下命令构建：`make world`
  + Install &emsp;&emsp;&emsp;&emsp;对应于`make world`，使用如下命令安装：`make install-world`
  + 最终会提示：`PostgreSQL, contrib, and documentation installation complete.`

#### 2.3 安装完成后的步骤

（1） 添加PostgreSQL数据库的超级用户postgres所对应的系统用户
{% highlight Shell %}
adduser postgres
{% endhighlight %}
这是PostgreSQL数据库的惯例，这样在postgres系统用户下就可以使用psql直接登录数据库。

（2） 建立data目录作为数据簇实例的目录,并改变这个目录的用户为刚刚建立的postgres
根据编译源码安装时的安装目录，设置如下：
{% highlight Shell %}
mkdir /usr/local/pgsql/10.1/data
chown postgres /usr/local/pgsql/10.1/data
{% endhighlight %}

（3） 初始化数据簇
切换到刚刚建立的postgres用户
{% highlight Shell %}
su - postgres
{% endhighlight %}
初始化
{% highlight Shell %}
/usr/local/pgsql/10.1/bin/initdb -D /usr/local/pgsql/10.1/data
{% endhighlight %}

（4） 在postgres系统用户下，设置环境变量等
为了能够在shell中使用psql等命令，需要添加环境变量，采用如下方式：
{% highlight Shell %}
vim ~/.profile
{% endhighlight %}

  + 在打开的文件末尾输入以下内容：

{% highlight Shell %}
  LD_LIBRARY_PATH=/usr/local/pgsql/10.1/lib
  export LD_LIBRARY_PATH

  PATH=/usr/local/pgsql/10.1/bin:$PATH
  export PATH

  MANPATH=/usr/local/pgsql/10.1/share/man:$MANPATH
  export MANPATH
{% endhighlight %}

  + 然后使其即时生效：

{% highlight Shell %}
source ~/.profile
{% endhighlight %}

  + 此时在命令行下输入`psql`，会得到如下信息：

{% highlight Shell %}
psql: could not connect to server: No such file or directory
  Is the server running locally and accepting
  connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
{% endhighlight %}

  + 这代表服务尚未启动，但是程序安装成功，数据簇初始化成功，环境变量添加成功。
一次性启动服务，可以使用如下命令：

{% highlight Shell %}
/usr/local/pgsql/10.1/bin/postgres -D /usr/local/pgsql/10.1/data >logfile 2>&1 &
{% endhighlight %}

  + 正常的话，会输出一个PID，然后再输入`psql`，就会得到如下信息：

{% highlight Shell %}
psql (10.1)
Type "help" for help.

postgres=#
{% endhighlight %}

  + 这代表已经正常进入了psql交互命令环境，可以操作数据库了。

（5） 设置PostgreSQL开机自启动

  + 在源代码的`../contrib/start-scripts/`中`linux`文件是为Linux系统写的启动脚本.
拷贝~/postgresql-10.1/contrib/start-scripts/linux 到/etc/init.d/postgresql
切换到`/etc/init.d/`目录中，更改postgresql的权限：
{% highlight Shell %}
chmod a+x postgresql
{% endhighlight %}

  + 然后`vim postgresql`打开脚本，按照其中的指示修改文件，主要修改以下几处：

{% highlight Shell %}
  .....

  # Installation prefix
  prefix=/usr/local/pgsql/10.1

  # Data directory
  PGDATA="/usr/local/pgsql/10.1/data"

  # Who to run the postmaster as, usually "postgres".  (NOT "root")
  PGUSER=postgres

  ......
{% endhighlight %}

  + 在脚本开头几行的注释中，给出了如何添加开机自启动的步骤。但是在我的机器上始终执行不成功。然后使用了一种偷懒的方法。在Linux启动的时候，/etc/rc.local是最后执行的脚本，只需要将启动postgresql的命令放在这个脚本中就可以了。切换到`/etc/`目录下，`vim rc.local`打开rc.local。在最后`exit 0`之前，加入如下的语句：
{% highlight Shell %}
  service postgresql start
{% endhighlight %}

  + 保存退出之后，`reboot`重启系统。待系统重启成功自后，使用如下的命令验证是不是自启动成功了。
{% highlight Shell %}
  service --status-all
{% endhighlight %}

  + 看postgresq那个服务之前是`+`就代表启动成功，`-`则代表启动不成功。如果要进一步验证的话们可以使用：
{% highlight Shell %}
  su - postgres
  psql
{% endhighlight %}

  + 如果能正常地显示：

{% highlight Shell %}
psql (10.1)
Type "help" for help.

postgres=#
{% endhighlight %}
就代表服务启动成功。
这时相当于系统用户postgres以同名数据库用户的身份，登录数据库，这是不用输入密码的。如果一切正常，系统提示符会变为"postgres=#"，表示这时已经进入了数据库控制台。以下的命令都在控制台内完成。建议为了数据库安全（养成好习惯）马上为超级用户postgres设置密码。使用\password命令，为postgres用户设置一个密码如下：

{% highlight psql %}
  \password postgres
{% endhighlight %}

## 3. 带有`--with-systemd`的安装及其优势

前面的安装过程中，设置开机自启动着实费了一番功夫。后来查资料的过程中，了解到通过systemd进行系统服务管理更加简便，于是，有了下面的内容。

使用systemctl命令进行postgresql服务的管理需要在`./configure`的时候使用参数`--with-systemd`。从上文的`2.2 编译及安装`中的`（2）编译安装及说明`开始，configure命令使用如下：

{% highlight Shell %}
  ./configure --prefix=/usr/local/pgsql/10.1/ --with-perl --with-python --with-openssl --with-systemd
{% endhighlight %}

之后的编译安装命令同上不变。

安装完成之后，参照PostgreSQL官网的[Chapter 18. Server Setup and Operation](https://www.postgresql.org/docs/10/static/server-start.html)使用systemd管理postgresql的service unit file。如下：

```
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify
User=postgres
ExecStart=/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

service unit file的位置：`/etc/systemd/system/postgresql.service`

启动postgresql：

{% highlight Shell %}
systemctl start postgresql
{% endhighlight %}

设置开机启动postgresql：

{% highlight Shell %}
systemctl enable postgresql
{% endhighlight %}

至此，postgresql服务器端安装和自启动配置全部完成！

对比之下，可以知道，使用systemd更加便利！









## 参考：
1. [PostgreSQL官方手册之III. Server Administratio--16. Installation from Source Code](https://www.postgresql.org/docs/10/static/installation.html)
2. [唐成-PostgreSQL修炼之道:从小工到专家](https://www.amazon.cn/dp/B00WUBYVIO/ref=sr_1_1?s=books&ie=UTF8&qid=1514786248&sr=1-1)
3. [阮一峰-PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)