---
title: 安装tomcat服务器
date: 2017-01-06 20:04:59
tags:
  - ubuntu
  - tomcat
  - hudson
---

# 更新系统

```bash
$ apt-get update
$ apt-get upgrade
```

# 安装java

```bash
$ apt-get install default-jdk
```

# 安装mysql

```bash
$ apt-get install mysql-server
```

# 安装tomcat

```bash
$ apt-get install tomcat7 tomcat7-admin
```

添加管理员账号

```bash
$ vim /etc/tomcat7/tomcat-users.xml
```

```xml
<tomcat-users>
    <user username="admin" password="password" roles="manager-gui,admin-gui"/>
</tomcat-users>
```

重启tomcat

```bash
$ service tomcat7 restart
```

# 安装hudson

1. 在[hudson官网](https://www.eclipse.org/hudson/download.php)下载最新版hudson
2. 打开`http://server_IP_address/manager`
3. 上传`hudson.war`

修改配置文件

```bash
$ vim /etc/default/tomcat7
```

添加`JAVA_OPTS`中的配置

```properties
JAVA_OPTS="-DHUDSON_HOME=/home/hudson/"
```

新建hudson文件夹

```bash
$ mkdir /home/hudson
```

修改文件的所属用户和用户组

```bash
$ chown tomcat7:tomcat7 hudson/
```

# 安装git

```bash
$ apt-get install git
```

# 安装maven

```bash
$ apt-get install maven
```
