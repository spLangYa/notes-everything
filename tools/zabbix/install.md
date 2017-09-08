# zabbix 安装

## 前言

zabbix安装有很多方法，如源码安装、包安装、容器安装，本文介绍centos 7下的包安装方法；如读者需要了解其他安装方法请阅读：https://www.zabbix.com/documentation/3.4/manual/installation。  
zabbix安装包括两个部分，zabbix-server安装,zabbix-agent安装；


## 安装server

### 安装系统

下载最小版centos 7,安装系统，本文不详细介绍系统安装

### 安装仓库配置包

```
# rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.centos.noarch.rpm
```

### server安装

```
# yum install zabbix-server-mysql zabbix-web-mysql
```

### 安装数据库

* 安装数据库方法请查看 [MYSQL安装](/mysql/install.md)

* 生成数据库
```
shell> mysql -uroot -p<password>
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
mysql> quit;
```

* 导入初始化数据库
```
# zcat /usr/share/doc/zabbix-server-mysql-3.4.1/create.sql.gz | mysql -uzabbix -p zabbix
```
上诉脚本会需要我们输入密码才能够正常执行，并且请注意上面命令的版本``3.4.1``有可能不同，如果提示找不到文件，可以查看一下具体版本号，重新执行该命令； 

* 检查server是否正常安装
```
# rpm -q zabbix-server-mysql
```
* 配置数据库
根据实际情况，配置数据库
```
# vi /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=<password>
```
注： 上面几个配置并不连续，读者可以自行搜索几个配置项所在位置；

### selinux配置

需要设置selinux,方法如下(推荐方法2)：
1. 关闭selinux（不推荐）
```
setenforce 0
```
2. 更新selinux策略
```
yum update selinux-policy.noarch selinux-policy-targeted.noarch
```
//todo 这个是否有必要执行
//Having SELinux status enabled in enforcing mode, you need to execute the following command to enable successful connection of Zabbix frontend to the server:
```
# setsebool -P httpd_can_connect_zabbix on
```

### 启动zabbix 
```
# systemctl start zabbix-server
# systemctl enable zabbix-server
```

### zabbix前端参数配置

``/etc/httpd/conf.d/zabbix.conf``文件，读者可以自行修改；
```
php_value max_execution_time 300
php_value memory_limit 128M
php_value post_max_size 16M
php_value upload_max_filesize 2M
php_value max_input_time 300
php_value always_populate_raw_post_data -1
# php_value date.timezone Europe/Riga
```

### 启动http服务

```
# systemctl start httpd
```

### 允许http端口访问

打开端口
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
* zone #作用域
* add-port=80/tcp  #添加端口，格式为：端口/通讯协议
* permanent  #永久生效，没有此参数重启后失效

重启防火墙
```
firewall-cmd --reload
```

### 用网页访问zabbix
在浏览器中输入地址``http://{ip_address}/zabbix``,访问zabbix主页，如果能打开则安装成功。

![](assets/2017-09-08-12-24-16.png)


## 安装zabbix-server前端




