---
title: centos7安装jdk、docker
tags: docker
categories: 部署
abbrlink: 13516
date: 2020-07-29 14:59:59
---

# 安装JDk

1. oracle官网下载JDk：jdk-8u261-linux-x64.rpm

2. 安装JDK8：

```shell
rpm -ivh jdk-8u261-linux-x64.rpm
```

1. 配置环境变量

```shell
vim /etc/profile
```

3. 输入以下内容：

```shell
JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
JRE_HOME=/usr/java/jdk1.8.0_261-amd64/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

<!--more-->

然后执行以下命令生效：

```shell
source /etc/profile
```

# 安装mysql

```shell
# 下载mysql
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz
# 解压
tar -zxvf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz
# 创建data文件夹
mkdir data
```

```shell
# 在/etc目录下创建my.cnf文件
vim /etc/my.cnf
# 将以下内容写入
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=/usr/project/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/project/mysql/data
# 允许最大连接数/
max_connections=10000
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
#character-set-server=UTF8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
user=mysql
```

```shell
# 用户组
groupadd mysql
# 用户 （用户名/密码）
useradd -g mysql mysql
# 授权
chown -R mysql.mysql /usr/project/mysql/
# 切换mysql账户
su mysql

# 初始化mysql，注意保存root账户的密码
./bin/mysqld --initialize
# 启动mysql
cd support-files/
./mysql.server start
# 退出mysql账户，登录mysql
./bin/mysql -uroot -p
# 修改mysql密码
alter user 'root'@'localhost' identified by 'xxxxxx';
# 修改root账户远程登录（谨慎操作）
update user set host='%' where user='root' and host='localhost';
# 刷新
flush privileges;
```

```shell
# 将mysql加入到服务中
cp /usr/project/mysql/support-files/mysql.server /etc/init.d/mysql
# 添加开机启动
chkconfig mysql on
# 关闭mysql
systemctl stop mysql
```

# 安装tomcat

```shell
# 下载tomcat，或者https://tomcat.apache.org/download-90.cgi官网下载

wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.37/bin/apache-tomcat-9.0.37.tar.gz
# 解压
tar -zxv -f apache-tomcat-9.0.37.tar.gz
```

# 安装maven

# 安装Jenkins

   ```shell
# 下载Jenkins，清华大学开源软件镜像站
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/latest/jenkins.war
   ```

编写启动脚本

```shell
vim run.sh
#!/bin/bash

nohup java -jar jenkins.war --httpPort=6004
```

或者放到tomcat中，修改tomcat

```xml
#添加Context标签，访问端口直接指向jenkins
<Host name="localhost" appBase="webapps"
              unpackWARs="true" autoDeploy="true">
              <Context path="/" docBase="jenkins" reloadable="true"/>
              /Host>
```

根据提示粘贴密码验证

安装推荐的插件，大概率下载不下来，进去jenkins中，在``管理Jenkins->插件管理中选择高级选项卡``，升级站点输入，提交。

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

# 安装docker

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

## docker常用命令

```shell
# 查看docker镜像
docker images
# 删除镜像
docker rmi xxx（容器名称或id）
# 查看docker容器
docker ps -a
# 删除容器
docker rm xxx（容器名称或id）
# 查看日志
docker logs xxx（容器名称或id）
# 进入容器内部
docker exec -it xxx（容器名称或id） /bin/bash
# 从主机复制到容器
docker cp host_path containerID:container_path
# 从容器复制到主机
docker cp containerID:container_path host_path
```

## docker安装nginx

1. 拉取nginx镜像：
```shell
docker pull nginx
# 创建挂载目录
mkdir -p /usr/project/nginx/html
mkdir -p /usr/project/nginx/logs
mkdir -p /usr/project/nginx/conf
```

2. 上传nginx配置文件到/usr/project/nginx/conf目录下

```conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   /usr/share/nginx/html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

5. 创建容器：

```shell
docker run --name nginx -d -p 80:80  \
-v /usr/project/nginx/conf/nginx.conf:/etc/nginx/nginx.conf  \
-v /usr/project/nginx/html:/usr/share/nginx/html \
-v /usr/project/nginx/logs:/var/log/nginx nginx
```

## docker安装gitlab

1.  拉取镜像

```shell
docker pull gitlab/gitlab-ce:latest
```

2. 启动容器

```shell
# --hostname是配置宿主机的ip（克隆时的地址），否则ip为容器ID
# 6003端口为http端口，6002端口为ssh克隆端口
docker run -d \
	--hostname 255.255.255.255 \
    -p 8443:443 -p 6003:80 -p 6002:22 \
    --name gitlab \
    -v /usr/project/gitlab/config:/etc/gitlab \
    -v /usr/project/gitlab/logs:/var/log/gitlab \
    -v /usr/project/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

在生成的挂载目录gitlab/config下，找到gitlab.rb文件修改gitlab_rails['gitlab_shell_ssh_port'] = 6002。

这里遇到点问题，安装完成后，ssh克隆正常，但是http方式，url上却少了6003端口，为默认的80端口。

这里采取进入容器内部``/opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml``，修改其端口为6003，执行``gitlab-ctl restart``重启gitlab解决。相对比较麻烦！后续看有没有其他方便点的操作。

