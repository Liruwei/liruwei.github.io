---
title: flask + gunicorn + nginx 搭建 api server
layout: post
category: python3
tags: [python3]
---

最近工作任务不多，闲着蛋疼自己写了个项目。从需求、UI、开发、测试到发布都是一个人来搞，好过瘾 (⁎⁍̴̛ᴗ⁍̴̛⁎)。当然对于我这种记性不好的人来说，必须记录下来，用来装B和回忆。废话不多写直接进入主题...

# 环境

**运行环境**

* CentOS 7.4
* Node 6.12.2
* Python 3.6.1
* MySQL

**项目架构**

* Nginx (Web服务器层,这里只是用来做 **反向转发** )
* Gunicorn (WSGI层)
* Flask (Web框架层)

## 配置 Python

1. 安装 Python 3 环境
    
    1. 官方网站下载 Python3 安装包
    
        ~~~python
        # wget http://python.org/ftp/python/3.6.1/Python-3.6.1.tar.xz
        ~~~
    
    2. 解压安装包
        
        ~~~python
        # tar xf Python-3.6.1.tar.xz
        ~~~
        
    3. 编译安装

        下载zlib工具包，安装python需要
        
        ~~~python
        # yum -y install zlib*
        ~~~
        
        进入Python-3.6.1文件夹下，进行编辑安装        
        
        ~~~python
        # ./configure --prefix=/usr/local/python3
        # make & make install
        ~~~
        
         > --prefix设置的是python3要安装到的位置

    4. 添加到环境变量

        添加刚安装的python3版本的文件连接
        
        ~~~python
        # ln -s /usr/local/python3/bin/python3 /usr/bin/python3 
        
        查看python3版本信息
        # python3 -V
        Python 3.6.1
        ~~~
        
        添加pip的文件连接
        
        ~~~python
        查看pip版本信息
        # python3 -m pip -V
        pip 9.0.1 from /usr/local/lib/python3.6/site-packages (python 3.6)
        
        添加pip的文件连接
        # ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
        
        查看pip版本信息
        # pip3 -V
        pip 9.0.1 from /usr/local/lib/python3.6/site-packages (python 3.6)
        ~~~
        
        编辑/etc/profile文档，在末尾添加如下，并保存退出
        
        ~~~python
        # vi /etc/profile
        export PATH="/usr/local/python3/bin:$PATH"
        
        # source /etc/profile
        ~~~
        
        
    
## 配置 Gunicorn

Gunicorn 应该装在你的 [virtualenv]() 环境下。

安装virtualenv。

~~~python
# pip3 install virtualenv
~~~

创建存放virutalenv创建环境的目录

~~~python
# mkdir ~/env/api/service
~~~

> 这个路径只是用来存放virtualenv创建的运行环境，没特别要求。当然也可以用[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)来管理。

进入目录，创建并激活运行环境

~~~python
# cd ~/env/api/service

创建运行环境
# virtualenv api_service

激活
# source api_service/bin/activate
~~~

安装 gunicorn

~~~python
# pip3 install gunicorn
~~~

## 配置 Nginx

1. 添加Nginx到YUM源
    
    ~~~
    # sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
    ~~~

2. 安装Nginx
    
    ~~~
    # sudo yum install -y nginx
    ~~~

3. 启动Nginx

    ~~~
    # sudo systemctl start nginx.service
    ~~~
    
## 配置 MySQL

1. 下载安装包
    
    ~~~python
    # wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
    # sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
    ~~~
    
2. 安装/启动 mysql
    
    ~~~python
    更新yum软件包
    # yum check-update 
    更新系统
    # yum update
    安装mysql
    # yum install mysql mysql-server 
    启动
    # systemctl start mysqld
    ~~~
    
3. mysql设置root账户密码

    ~~~python
    # mysql -u root
    # mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
    ~~~

4. mysql创建远程访问用户

    ~~~python
    # mysql> create user admin;
    # mysql> GRANT ALL PRIVILEGES ON *.* TO admin@"%" IDENTIFIED BY 'admin' WITH GRANT OPTION; 
    # mysql> flush privileges;
    ~~~
    
5. 创建数据库

    ~~~python
    # mysql> CREATE DATABASE database_name DEFAULT CHARACTER SET "utf8";
    ~~~
    
    > **记得设置字符编码! 记得设置字符编码! 记得设置字符编码!**
    

# 上传发布

## 上传项目代码

把项目代码上传到服务器有很多种方法，你可以用`git`，也可以简单用`scp`命令，当然还可以用逼格高一个层次的rsync。

1. 服务端配置
    
    1. 修改rsync配置文件，创建同步节点
    
    ~~~python
    # vim /etc/rsyncd.conf
    
    [smp]
        path = /root/app/api_service
        list = true
        uid = root
        gid = root
        read only = false
    ~~~
    
    > **参数说明:**
    > **smp**: 代表配置节点的名称。**path**: 该节点对应的文件真实路径。**list**: 表示该节点是否可被发现。**uid**: 指定传输到这里文件所属的用户。**gid**: 指定传输到这里的文件所属的组。**read** only: 该目录是否只读
    
    2. 开启 rsync 服务
    
    ~~~python
    # rsync --daemon
    ~~~
    
    > 如果修改了rsyncd.conf文件，需要重启rsync服务 `kill -9 [PID] ; rsync --daemon`

2. 本地上传项目代码
    
    进入项目根目录,执行rsync命令同步文件
    
    ~~~python
    # rsync -avzP --progress --delete ./ root@32.24.123.233::smp 
    ~~~
    
    > root@32.24.123.233::smp 代表服务端的文件路径 
    > {user}@{ip}::{节点名}
    > 
    > --delete 参数，这样当本地删除的文件，远程端也会删除，保持完整的一致。
    
    下面是rsync简单说明
    
    ~~~
    命令格式 : rsync [OPTION] {source} {destination} 
    source : 同步来源
    destination : 你需要同步到的目的地
    
    OPTION详细说明:
    -v, --verbose 详细模式输出
    -q, --quiet 精简输出模式
    -c, --checksum 打开校验开关，强制对文件传输进行校验
    -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
    -r, --recursive 对子目录以递归模式处理
    -b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
    -suffix=SUFFIX 定义备份文件前缀
    -u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
    -l, --links 保留软链结
    -p, --perms 保持文件权限
    -o, --owner 保持文件属主信息
    -g, --group 保持文件属组信息
    -t, --times 保持文件时间信息
    -e, --rsh=COMMAND 指定使用rsh、ssh方式进行数据同步
    --delete 删除那些DST中SRC没有的文件
    --delete-excluded 同样删除接收端那些被该选项指定排除的文件
    --delete-after 传输结束以后再删除
    --ignore-errors 及时出现IO错误也进行删除
    --force 强制删除目录，即使不为空
    --timeout=TIME IP超时时间，单位为秒
    --progress 显示备份过程
    -z, --compress 对备份的文件在传输时进行压缩处理
    --exclude=PATTERN 指定排除不需要传输的文件模式
    --include=PATTERN 指定不排除而需要传输的文件模式
    --exclude-from=FILE 排除FILE中指定模式的文件
    --include-from=FILE 不排除FILE指定模式匹配的文件
    ~~~

## 运行项目

运行前，记得激活virtualenv运行环境，并`pip3 install` 项目需要的module。

1. 进入服务端项目根目录。

    ~~~python
    # cd ~/app/api_service
    ~~~
    
2. 激活virtualenv
 
    ~~~python
    # source SMP/bin/activate    
    ~~~

3. 后台运行运行gunicorn

    ~~~python
    # nohup gunicorn file_name:flex_name &
    ~~~
    
    > **file_name**入口文件名，不用带后缀。**flask_name**flsk实例的名称

4. 配置nginx
    
    1. 修改nginx配置文件，添加以下代码

    ~~~python
    # vim /etc/nginx/nginx.conf
    server {
        listen       8080;
        listen       [::]:8080 ipv6only=on;
        server_name  11.111.11.10:8080;

        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location = /favicon.ico {
          log_not_found off;
        }
    }
    ~~~
    
    > **server_name**这是HOST机器的外部地址，用域名也行。**proxy_pass**这里是指向 gunicorn host 的服务地址。
    
    2. 重启nginx
    
    ~~~python
    # sudo service nginx restart
    ~~~
    
    > 如果重启失败，简单点就杀死nginx进程再开启就好。
    
    好到目前为止,项目已经成功发布。
    
## Linux重启自动运行项目

采用 UpStart 配置Flask程序作为服务程序在Linux起动时运行

1. 建立起动配置文件

    ~~~python
    # sudo nano /etc/init/api_server.conf
    ~~~
    
    加入如下配置
    
    ~~~python
    description "The api service"

    start on runlevel [2345]
    stop on runlevel [!2345]
    
    respawn
    setuid root
    setgid www-data
    
    env PATH= /root/app/api_service/SMP/bin
    chdir /root/app/api_service/
    
    exec gunicorn -w 4 -b 127.0.0.1:8000 file_name:flask_name
    ~~~
    
    > **env**virtualenv 的路径。**chdir**项目路径。
    
2. 启动 api_server 服务
    
    ~~~python
    # sudo service api_service start
    ~~~
    
好！大功告成！

# 其他

## 常有命令

* `nohup {*} &` : 后台运行服务
* `netstat -ntpl` : 查看占用端口的进程

## 防火墙

当我们把所有东西都运行起来的时候，会发现外面还是访问不了。这时候通常是防火墙在作怪。
如果是购买阿里云的服务器，可进入阿里云管理控制台，找到对应的实例，查看实例安全组件=》配置安全规则。开放需要的端口。

## 资料

* [CentOS 7 安装mysql](https://www.jianshu.com/p/17fb10320d63)
* [mysql修改root密码和设置权限](http://www.cnblogs.com/wangs/p/3346767.html)
* [35个常用的mySQL基本操作](https://www.jianshu.com/p/24afa0e637cc)
* [CentOS源码安装Python3.6](https://www.jianshu.com/p/05ac00ff24a2)
* [Python 操作 MySQL 的正确姿势](https://www.qcloud.com/community/article/687813)


