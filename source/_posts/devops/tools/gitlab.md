---
title: 在Centos7上安装GitLab
date: 2017-08-16 15:57:00
categories:
- devops
- tool
tags:
- version control
- devops
- gitlab
---

GitLab是利用 Ruby on Rails 的一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找

<!-- more -->

### 配置要求

RAM 4G+

### 依赖组件

根据官方文档说明有以下依赖，但实际中发现部分无需安装也可以.

1. Packages / Dependencies
2. Ruby
3. Go
4. Node
5. System Users
6. database
7. Redis
8. GitLab
9. Nginx
10. Git(2.8.4 or higher)

### 安装方式

安装方式有两种:Omnibus包安装和编译安装。

#### 一、Omnibus包安装（安装起来更快、更容易升级版本，而且包含了其他安装方式所没有的可靠性功能）

1.安装Packages / Dependencies

    [root@centos ~]# yum -y update

2.安装Ruby

Ruby安装有四种方式:

- 系统包管理工具(yum、apg-get等),简单快速,但是一般不是最新的版本
- 安装工具
- 第三方管理工具,RVM等
- 编译方式

因为我要安装ruby2.4,yum库中只能安装ruby2.0,所以我选择了编译安装


    [root@centos ~]# ruby -v
    bash: ruby: command not found...
    [root@centos ~]# curl -O https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.1.tar.gz
    [root@centos ~]# tar -zxvf ruby-2.4.1.tar.gz
    [root@centos ~]# cd ruby-2.4.1/
    [root@centos ~]# ./configure
    [root@centos ~]# make && make install

添加环境变量

    [root@centos ~]# vim ~/.bash_profile
        RUBY_ROOT=/usr/local/lib/ruby
        RUBY=$RUBY_ROOT/2.4.0
        GEM=$RUBY_ROOT/gem
        export PATH=$PATH:$RUBY:$GEM
    [root@centos ~]# source ~/.bash_profile
    [root@centos ~]# ruby -v
     ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-linux]
    [root@centos ~]# gem --version
     2.6.11


​    
3.System Users
<br />添加gitlab用户     
​    
[root@centos ~]# adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/gitlab gitlab

4.安装PostgreSQL

    [root@centos ~]# yum install -y https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
    [root@centos ~]# yum install -y postgresql96 postgresql96-server postgresql96-contrib 
    [root@centos ~]# vim ~/.bash_profile
        POSTGRESQL=/usr/pgsql-9.6
        LD_LIBRARY_PATH=$POSTGRESQL/lib
        PGDATA=/var/lib/pgsql/9.6/data
        export PATH=$PATH:$POSTGRESQL/bin:$PGDATA:$LD_LIBRARY_PATH
    [root@centos ~]# source ~/.bash_profile
    [root@centos ~]# systemctl enable postgresql-9.6

初始化

    [root@centos ~]# /usr/pgsql-9.6/bin/postgresql96-setup initdb  

设置密码登录

    [root@centos ~]# vim /var/lib/pgsql/9.6/data/pg_hba.conf
        #host    all             all             127.0.0.1/32            ident
        #改为
        #host    all             all             127.0.0.1/32            md5




启动

    [root@centos ~]# systemctl start postgresql-9.6

启动服务:

    [root@centos ~]# systemctl start postgresql-9.6

重启服务

    [root@centos ~]# systemctl restart postgresql-9.6

停止服务

    [root@centos ~]# systemctl stop postgresql-9.6

自动启动

    [root@centos ~]# systemctl enable postgresql-9.6

postgresql96:客户端 </br>
ostgresql96-server:服务端 </br>
postgresql96-contrib:常用的组件和方法 </br>
切换至postgres用户

    [root@centos ~]# su - postgres 
    #(或su postgres)
    #切换用户，执行后提示符会变为 '-bash-4.2$'
    bash-4.2$ psql --version
     psql (PostgreSQL) 9.6.4

登录数据库，执行后提示符变为 'postgres=#'

    bash-4.2$ psql -U postgres
    #设置postgres用户密码
    postgres=# ALTER USER postgres WITH PASSWORD '123456';
    #创建gitlab用户
    postgres=# CREATE USER gitlab SUPERUSER PASSWORD 'gitlab';
    #查看现有用户
    postgres=# \du
    postgres-# CREATE DATABASE gitlabhq_production;
     CREATE DATABASE
    #退出数据库
    postgres=# \q

在root用户下登录:

    [root@centos ~]# sudo psql -U gitlab -h 127.0.0.1 -d gitlabhq_production
     gitlabhq_production=#

5.安装Git

    [root@centos ~]# yum -y install git

6.安装Redis

①.到redis官网下载最新稳定版(https://redis.io),

    yum install gcc tcl -y
    curl -O http://download.redis.io/releases/redis-4.0.9.tar.gz
    tar -zxvf redis-4.0.9.tar.gz
    cd redis-4.0.9
    make
    mkdir /usr/local/redis
    # 进入src目录安装
    cd src && make PREFIX=/usr/local/redis install

②. 添加全局配置

    vim ~/.bash_profile
    
    REDIS_HOME=/usr/local/redis
    PATH=$PATH:$REDIS_HOME:/bin
    export PATH REDIS_HOME
    
    source ~/.bash_profile
    
    #测试
    redis-server --help

③.添加后台daemon

    cd ~/redis-4.0.9/utils
    ./install_server.sh
    # 默认配置
    Config file    : /etc/redis/6379.conf
    Log file       : /var/log/redis_6379.log
    Data dir       : /var/lib/redis/6379
    Executable     : /usr/local/redis/bin/redis-server
    Cli Executable : /usr/local/redis/bin/redis-cli
    
    默认redis 服务名称为redis_6379(可以在cd /etc/init.d内查看或更改)
若想更改为redisd作为服务名

    cd /etc/init.d
    mv redis_6379 redisd
    chkconfig --add redisd

④. 启动

    systemctl status redis_6379.service
    ss -tanl

⑤. 远程访问及ip限制

    vim /etc/redis/6379.conf
    
    #protected-mode yes改为
    protected-mode no
    
    # 注释掉, 如下
    #bind 127.0.0.1

限制ip访问需要之后在研究下，并不是填上要访问的ip就是允许此ip访问，应该是填上可以访问的本机网卡ip

6.安装Gitlab

①.在 CentOS 系统上，下面的命令将会打开系统防火墙 HTTP 和 SSH 的访问。

    sudo yum install -y curl policycoreutils-python openssh-server

[//]: # (    sudo systemctl enable sshd)
    sudo systemctl start sshd
    sudo firewall-cmd --permanent --add-service=http
    sudo systemctl reload firewalld

安装 Postfix 发送通知邮件, 如果想使用其他邮件系统可跳过此步骤(Sendmail 或者 配置一个自定义的 SMTP 服务,https://docs.gitlab.com/omnibus/settings/smtp.html)

    sudo yum install postfix
    sudo systemctl enable postfix
    sudo systemctl start postfix

②.添加 GitLab 镜像源并安装,EXTERNAL_URL 为gitlab的访问域名

    # 这是官方的yum源，安装速度会比较慢
    [root@centos ~]# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
    # 可以使用国内源，修改如下文件即可：
    vim /etc/yum.repos.d/gitlab_gitlab-ce.repo
    
    [gitlab-ce]
    name=gitlab-ce
    baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
    repo_gpgcheck=0
    gpgcheck=0
    enabled=1
    gpgkey=https://packages.gitlab.com/gpg.key
    
    #然后执行：
    [root@centos ~]# sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ce
    
    #修改配置
    [root@centos ~]# vim /etc/gitlab/gitlab.rb


​    
③.需要修改的配置
​    
#13 项目中显示的gitlab地址
external_url 'http://gitlab.example.com'

    #401 使用非集成的postgresql
    gitlab_rails['db_adapter'] = "postgresql"
    gitlab_rails['db_encoding'] = "utf8"
    #gitlab_rails['db_encoding'] = "unicode
    #gitlab_rails['db_collation'] = nil
    # gitlab_rails['db_database'] = "gitlabhq_production"
    # gitlab_rails['db_pool'] = 10
    gitlab_rails['db_username'] = "gitlab"
    gitlab_rails['db_password'] = "123456"
    gitlab_rails['db_host'] = '127.0.0.1'
    gitlab_rails['db_port'] = '5432'
    
    #423 使用非集成的redis
    gitlab_rails['redis_host'] = "127.0.0.1"
    gitlab_rails['redis_port'] = 6379
    gitlab_rails['redis_password'] = "123456"
    gitlab_rails['redis_database'] = 1
    
    #613 设置unicorn监听端口
    unicorn['listen'] = '127.0.0.1'
    unicorn['port'] = 8087
    
    #676 是否使用集成postgresql
    postgresql['enable'] = false
    
    #799 是否使用集成redis
    redis['enable'] = false
    
    #878 是否使用集成nginx,即使本地安装了Nginx也建议打开集成的nginx
    nginx['enable'] = true
    
    #879 集成的Nginx监听的端口,若要本地和集成的Nginx同时使用,则更改监听端口
    nginx['listen_port'] = 80
    #nginx['listen_port'] = 8088

④.使配置生效

    [root@centos ~]# sudo gitlab-ctl reconfigure
    #启动gitlab
    [root@centos ~]# gitlab-ctl start
    #重启
    [root@centos ~]# gitlab-ctl restart
    #停止
    [root@centos ~]# gitlab-ctl stop

#### 二、编译安装(待补充)


#### 安装成功

安装成功之后,访问域名或者ip,会提示更改root用户密码

{% asset_img gitlab_change_pwd.png 安装成功 %}

随后跳转登录,用户名为root

#### 汉化

查看已安装gitlab的版本

    cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
    #10.7.0

获取gitlab汉化包, 进入gitlab中文社区(https://gitlab.com/xhang/gitlab), 找到对应版本

    git clone https://gitlab.com/xhang/gitlab.git -b 10-7-stable-zh
    cat gitlab/VERSION
    #10.7.1

若中文版与已安装版本不同, 则可在分支中下拉，找到tag，在tag中找相应的版本,若有则在本地git库中pull

    git checkout -b v10.7.0-zh v10.7.0-zh
    # git checkout -b branch_name(新创建的分支名称) tag_name(对应tag名称)

则可以切换到相应版本

    # 备份
    tar -zcvf ~/gitlab_bak.tar.gz /opt/gitlab/embedded/service/gitlab-rails
    # 用中文版替换
    cp ~/gitlab-git/* /opt/gitlab/embedded/service/gitlab-rails -rf

若提示如下信息,可以不用理会

    cp: 无法以目录"/opt/gitlab/gitlab-git/log" 来覆盖非目录"/opt/gitlab/embedded/service/gitlab-rails/log"
    cp: 无法以目录"/opt/gitlab/gitlab-git/tmp" 来覆盖非目录"/opt/gitlab/embedded/service/gitlab-rails/tmp"

替换过程中可能出现覆盖确认

    vim ~/.bashrc
    # 将 alias cp='cp -i' 注释, 并重新登录  
    
    sudo gitlab-ctl reconfigure
    sudo gitlab-ctl start/restart


查看运行日志

    tail -f /var/log/gitlab/gitlab-rails/sidekiq.log


#### 使用非集成的Nginx

在Nginx中添加如下配置

    server{
        listen 8083;
        server_name gitlab.example.com;
    
        location / {
            # 这个大小的设置非常重要，如果 git 版本库里面有大文件，设置的太小，文件push 会失败，根据情况调整
            client_max_body_size 50m;
    
            proxy_redirect off;
            #以下确保 gitlab中项目的 url 是域名而不是 http://git，不可缺少
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            # 反向代理到 gitlab 内置的 nginx
            proxy_pass http://127.0.0.1:8088;
            index index.html index.htm;
        }
    }



#### 参考文献

http://www.cnblogs.com/wintersun/p/3930900.html
https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md

https://www.linuxidc.com/Linux/2017-11/148223.htm  //汉化
* 其他搜索未记录