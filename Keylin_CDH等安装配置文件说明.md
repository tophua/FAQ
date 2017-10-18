<!-- TOC -->

- [1. 服务配置说明](#1-服务配置说明)
    - [1.1. CentOS系统设置](#11-centos系统设置)
    - [1.2. 硬件配置](#12-硬件配置)
    - [1.3. 组件配置要求](#13-组件配置要求)
        - [1.3.1. HDFS 分布式存储](#131-hdfs-分布式存储)
        - [1.3.2. zookeeper 分布式一致性服务](#132-zookeeper-分布式一致性服务)
        - [1.3.3. HBase 分布式列数据库](#133-hbase-分布式列数据库)
    - [1.4. 所有配置前的先决配置](#14-所有配置前的先决配置)
    - [1.5. LEVEL_1配置](#15-level_1配置)
        - [1.5.1. 服务](#151-服务)
        - [1.5.2. 功能](#152-功能)
        - [1.5.3. 结点服务配置](#153-结点服务配置)
    - [1.6. LEVEL_2 配置](#16-level_2-配置)
        - [1.6.1. 服务](#161-服务)
        - [1.6.2. 功能](#162-功能)
        - [1.6.3. 结点服务配置](#163-结点服务配置)
    - [1.7. 组件安装](#17-组件安装)
        - [1.7.1. MySQL](#171-mysql)
            - [1.7.1.1. 安装](#1711-安装)
            - [1.7.1.2. 配置](#1712-配置)
                - [1.7.1.2.1. /etc/my.cnf 进行备份，将以下内容 覆盖原有文件内容](#17121-etcmycnf-进行备份将以下内容-覆盖原有文件内容)
                - [1.7.1.2.2. 设置随着系统启动](#17122-设置随着系统启动)
                - [1.7.1.2.3. 启动数据库](#17123-启动数据库)
                - [1.7.1.2.4. 设置数据库 root 密码(注意初始密码为空，直接 enter 即可)](#17124-设置数据库-root-密码注意初始密码为空直接-enter-即可)
                - [1.7.1.2.5. JDBC DRIVER 安装](#17125-jdbc-driver-安装)
                - [1.7.1.2.6. 创建数据库 和 各个角色 用户名 和密码](#17126-创建数据库-和-各个角色-用户名-和密码)
        - [1.7.2. 安装 Kerberos](#172-安装-kerberos)
            - [1.7.2.1. 安装前关闭 selinux](#1721-安装前关闭-selinux)
            - [1.7.2.2. 配置 JCE](#1722-配置-jce)
            - [1.7.2.3. 安装 KDC Server， krb5-client](#1723-安装-kdc-server-krb5-client)
            - [1.7.2.4. 创建数据库](#1724-创建数据库)
            - [1.7.2.5. 启动 kerberos server](#1725-启动-kerberos-server)
            - [1.7.2.6. 创建 kerberos 管理员](#1726-创建-kerberos-管理员)
            - [1.7.2.7. 在其他结点进行简单测试](#1727-在其他结点进行简单测试)
            - [1.7.2.8. KBR5的 cache的位置问题](#1728-kbr5的-cache的位置问题)
        - [1.7.3. 安装 CM/AGENT](#173-安装-cmagent)
            - [1.7.3.1. CM安装方式](#1731-cm安装方式)
                - [1.7.3.1.1. rpm安装](#17311-rpm安装)
                - [1.7.3.1.2. yum安装](#17312-yum安装)
                - [1.7.3.1.3. 启动 CM](#17313-启动-cm)
                    - [1.7.3.1.3.1. CM 启动过程中的问题](#173131-cm-启动过程中的问题)
            - [1.7.3.2. 安装agent](#1732-安装agent)
                - [1.7.3.2.1. 安装过程中的问题](#17321-安装过程中的问题)
                    - [1.7.3.2.1.1. 缺少DNS 服务器导致 指向ip地址错误](#173211-缺少dns-服务器导致-指向ip地址错误)
                    - [1.7.3.2.1.2. Cloudera Manager 正在获取安装锁](#173212-cloudera-manager-正在获取安装锁)
                    - [1.7.3.2.1.3. 安装完成后，出现  ：host monitor时发生内部错误](#173213-安装完成后出现--host-monitor时发生内部错误)
            - [1.7.3.3. Cloudera Manager 开启 kerberos认证](#1733-cloudera-manager-开启-kerberos认证)
                - [1.7.3.3.1. 界面配置过程](#17331-界面配置过程)
                - [1.7.3.3.2. 创建hadoop hdfs用户](#17332-创建hadoop-hdfs用户)
                - [1.7.3.3.3. 创建 hdfs 其他用户](#17333-创建-hdfs-其他用户)
                - [1.7.3.3.4. JAVA API 访问 HDFS](#17334-java-api-访问-hdfs)
                    - [【没有配置成功】添加 Java KeyStore KMS for transp encrypted](#没有配置成功添加-java-keystore-kms-for-transp-encrypted)
                    - [1.7.3.3.4.1. CM 部分设置说明](#173341-cm-部分设置说明)
        - [1.7.4. 安装Kylin](#174-安装kylin)
        - [1.7.5. 运行过程中的一些问题](#175-运行过程中的一些问题)
            - [1.7.5.1. reduce shuffle throws "java.lang.OutOfMemoryError: Java heap space](#1751-reduce-shuffle-throws-javalangoutofmemoryerror-java-heap-space)
            - [1.7.5.2. HDFS坏块的恢复](#1752-hdfs坏块的恢复)

<!-- /TOC -->



# 1. 服务配置说明

## 1.1. CentOS系统设置
```
cat /etc/security/limits.conf
*               soft    nproc           65536
*               hard    nproc           65536
*               soft    nofile          65536
*               hard    nofile          65536
```


## 1.2. 硬件配置
| 配置名称        | CORE  | RAM  | DISK |  
| ---------------|-------|------|------|
| hw0          |  2    | 8GB | 1TB     |
| hw1          |  4    | 8GB | 1TB     |
| hw2          |  8    | 16GB| 1TB     |
| hw3          |  8    | 32GB | 1TB    |
| hw4          |  8    | 32GB | 1TB    |


## 1.3. 组件配置要求
### 1.3.1. HDFS 分布式存储
* 默认情况下副本数为2,因此一个数据块总数量为3.因此HDFS 结点数量至少为3.
* 更改副本数量：cdh home->HDFS->配置-> search(复制因子:dfs.replication)

### 1.3.2. zookeeper 分布式一致性服务
* zookeeper 正常工作结点数量为 N/2+1, N=2n 与 N=2n-1 的容忍度相同，N的值至少为3。因此 至少要有3个结点。

### 1.3.3. HBase 分布式列数据库
* 可部署至 DataNode 所在的结点


## 1.4. 所有配置前的先决配置

- JDK: 每个结点均配置，版本一致。
- 特别注意：HADOOP 2.6.0 版本中，使用 JDK8 并且 开启 kerberos 认证，会出现无法读取凭证的 bug，这个问题在 hadoop 2.6.1 开始修复。JDK7 不存在这个问题。

> bug 说明:  https://community.hortonworks.com/questions/117897/hbase-connection-expiration-on-kerberized-cluster.html


| hostName            | role                     | 配置        |
| --------------------|--------------------------|------------|
| cmf_server.cz       | cloudera-scm-server      | node1          |
| 所有结点             | cloudera-scm-agent，kerberos.client      |    |
| kerberos_server.cz  | kerberos.KDC             |            |


## 1.5. LEVEL_1配置

### 1.5.1. 服务
* HIVE，HDFS，YARN，MAPREDUCE, MySQL

### 1.5.2. 功能
* 数据存储，简单的SQL语句查询


### 1.5.3. 结点服务配置
| hostName          | role                     | 配置      |
| ------------------|--------------------------|----------|
| name_node1.cz     | hdfs.NameNode            |          |
| name_node2.cz     | hdfs.SecondaryNameNode   |          | 
| node1.cz          | hdfs.DataNode,yarn.NodeManager            |          |
| node2.cz          | hdfs.DataNode,yarn.NodeManager           |          |
| node3.cz          | hdfs.DataNode,yarn.NodeManager            |          |
| hive.cz           | hive.HiveServer2,Hive Metastore Server,WebHCat Server |      |
| yarn_server.cz           | yarn.ResourceManager, yarn.JobHistory Sever  |          |
| mysql.cz          | mysql_server             |                        |
||||


## 1.6. LEVEL_2 配置

### 1.6.1. 服务
在LEVEL_1的基础上，增加 zookeeper,hbase,kylin

### 1.6.2. 功能
在LEVEL_2的基础上，增加复杂功能

> 这里的问题在于 一般情况，HBASE 与 HDFS 在同一个结点，其他计算框架单列结点（待决定）。

### 1.6.3. 结点服务配置
| hostName          | role                     | 配置      |
| ------------------|--------------------------|----------|
| name_node1.cz     | hdfs.NameNode            |          |
| name_node2.cz     | hdfs.SecondaryNameNode   |          | 
| node1.cz          | hdfs.DataNode,yarn.NodeManager,hbase_regionserver |          |
| node2.cz          | hdfs.DataNode,yarn.NodeManager,hbase_regionserver |          |
| node3.cz          | hdfs.DataNode,yarn.NodeManager,hbase_regionserver |          |
| hive.cz           | hive.HiveServer2,Hive Metastore Server,WebHCat Server |      |
| yarn_server.cz    | yarn.ResourceManager, yarn.JobHistory Sever      |          |
| mysql.cz          | mysql_server             |                        |
| zookeeper1.cz     | zookeeper_server         |          |
| zookeeper2.cz     | zookeeper_server         |          |
| zookeeper3.cz     | zookeeper_server         |          |
| hbase_sever.cz    | HBase_master             |          |



## 1.7. 组件安装
### 1.7.1. MySQL 
> 官网安装说明 https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_ig_installing_configuring_dbs.html#concept_i2r_m3m_hn

#### 1.7.1.1. 安装
```
RHEL
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
$ sudo yum install mysql-server
$ sudo systemctl start mysqld
```

#### 1.7.1.2. 配置
##### 1.7.1.2.1. /etc/my.cnf 进行备份，将以下内容 覆盖原有文件内容

```
[mysqld]
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
# symbolic-links = 0

key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system
#and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

# For MySQL version 5.1.8 or later. For older versions, reference MySQL documentation for configuration help.
binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=STRICT_ALL_TABLES
```
##### 1.7.1.2.2. 设置随着系统启动

```
# /sbin/chkconfig mysqld on 
# /sbin/chkconfig --list 
mysqld mysqld 0:off 1:off 2:on 3:on 4:on 5:on 6:off
```
- 在centos 7 中，chkconfig 已经被 systemctl 替代
```
# systemctl enable mysqld.service
```
##### 1.7.1.2.3. 启动数据库
```
# service mysqld start
```

##### 1.7.1.2.4. 设置数据库 root 密码(注意初始密码为空，直接 enter 即可)
```
$ sudo /usr/bin/mysql_secure_installation
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
All done!
```
##### 1.7.1.2.5. JDBC DRIVER 安装
 
> 下载驱动  https://dev.mysql.com/downloads/connector/j/5.1.html

在所有结点上执行以下命令

```
# mkdir -p /usr/share/java/
# cp mysql-connector-java-5.1.31/mysql-connector-java-5.1.31-bin.jar /usr/share/java/mysql-connector-java.jar
```
- 注意上面这个命令 对文件进行重新命名，否则 一些组件 找不到 java mysql 驱动！！

##### 1.7.1.2.6. 创建数据库 和 各个角色 用户名 和密码

| Role                  | Database   | User   | Password   |
|-----------------------|------------|--------|------------|
| Activity  Monitor     | amon       | amon   | amon1234   |
| Reports Manager       | rman       | rman   | rman1234   |
| Hive Metastore Server | metastore  | hive   | hive1234   |
| Sentry Server                        | sentry   | sentry  | sentry1234 |
| Cloudera Navigator Audit Server      | nav      | nav     | nav1234    |
| Cloudera Navigator Metadata Server   | navms    | navms   | navms1234  |
||||

对于以上每行 执行以下指令：

```
$ mysql -u root -p
Enter password:

mysql> create database ${database} DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on ${database}.* TO 'user'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.00 sec)
```


### 1.7.2. 安装 Kerberos

> 配置说明: https://www.cloudera.com/documentation/enterprise/latest/topics/cm_sg_kdc_def_domain_s2.html

#### 1.7.2.1. 安装前关闭 selinux

1. 查看SELinux状态
```
/usr/sbin/sestatus -v      ##如果SELinux status参数为enabled即为开启状态
SELinux status:                 enabled
```
2. 关闭SELinux：

（1) 临时关闭（不用重启机器）：
```
setenforce 0                  ##设置SELinux 成为permissive模式
                              ##setenforce 1 设置SELinux 成为enforcing模式
```

（2）修改配置文件，重启机器： <br/>
修改/etc/selinux/config 文件 <br/>
将SELINUX=enforcing改为SELINUX=disabled <br/>
重启机器即可 <br/>

#### 1.7.2.2. 配置 JCE

如果在认证过程中使用 AES-256 加密算法，则必须使用安装JCE Policy File
> 配置说明: https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_sg_s2_jce_policy.html

JCE 文件下载地址：
> JDK1.7 JCE: http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html

> JDK1.8 JCE: http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html 

解压后，将两个jar包 复制 到 ${JAVA_HOME}/jre/lib/security, 并对原有 jar 进行备份

#### 1.7.2.3. 安装 KDC Server， krb5-client
在 kdc server 结点上 执行
```
# yum install krb5-server krb5-libs krb5-auth-dialog krb5-workstation  -y
```

在所有结点上执行：
```
# yum install krb5-devel krb5-workstation -y
```

在结点 kerberos_server 修改 /etc/krb5.conf, 并复制到其他所有结点

```
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = HADOOP.EXAMPLE.COM
 #default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 HADOOP.EXAMPLE.COM = {
   kdc = node4:88
   admin_server = node4:749
   default_domain = hadoop.example.com
 }

[domain_realm]
 .hadoop.example.com = HADOOP.EXAMPLE.COM
 hadoop.example.com = HADOOP.EXAMPLE.COM

```

在结点 kerberos_server 修改 /var/kerberos/krb5kdc/kdc.conf<br>
configure the default realm for the Hadoop cluster by substituting your Kerberos realm in the following realmsproperty:
```
[realms]
 HADOOP.EXAMPLE.COM = {
 #master_key_type = aes256-cts
 acl_file = /var/kerberos/krb5kdc/kadm5.acl
 dict_file = /usr/share/dict/words
 admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
 supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 max_life = 1d
 max_renewable_life = 7d
 renewable = true
 default_principal_flags = +renewable, +forwardable
 } 
...

```

#### 1.7.2.4. 创建数据库

在 kerberos server 上
```
# kdb5_util create -r HADOOP.EXAMPLE.COM -s
```

- 如果遇到数据库已经存在的提示，可以把 /var/kerberos/krb5kdc/ 目录下的 principal 的相关文件都删除掉。默认的数据库名字都是 principal。可以使用 -d 指定数据库名字。

#### 1.7.2.5. 启动 kerberos server
```
# chkconfig --level 35 krb5kdc on
# chkconfig --level 35 kadmin on
# service krb5kdc start
# service kadmin start
```

#### 1.7.2.6. 创建 kerberos 管理员
 在 kerberos server 上 添加账户：root/admin，密码：root
 ```
 # kadmin.local -q "addprinc root/admin" 
 ```

> https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_sg_s3_cm_principal.html

```
# kadmin.local
# kadmin:  addprinc -pw <Password> cloudera-scm/admin@HADOOP.EXAMPLE.COM
```

在kadmin 下的一些命令：
- kadmin:list_principals：列出所有principals
- kadmin:cpw ${userName}： 重新设置密码

#### 1.7.2.7. 在其他结点进行简单测试
```
$ kinit root
Password for root@HADOOP.EXAMPLE.COM:
$ klist -e
Ticket cache: KEYRING:persistent:0:0 （注意默认情况 ticket cache 不再存储在 /tmp/ 中，而是在Linux 中的 KeyRing 中）
Default principal: root@HADOOP.EXAMPLE.COM

Valid starting       Expires              Service principal
2017-09-17T00:55:54  2017-09-18T00:55:54  krbtgt/HADOOP.EXAMPLE.COM@HADOOP.EXAMPLE.COM
        Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96

$ kinit -R  #如果该命令执行失败，表明KDC配置不正确

```



普通用户的部分命令：
- kinit ${userName} ：初始化用户凭证
- kinit -e : 查看当前凭证
- kinit -R ： 刷新当前凭证，
- kinit -kt /xx/xx/kerberos.keytab ${userName}：通过 keytab 初始化用户凭证
- kdestroy： 销毁当前凭证

#### 1.7.2.8. KBR5的 cache的位置问题
Centos7 和 6 存储ccache的方式似乎是不一样的，还是 高版本的 kerberos 默认的存储的位置变化了，有待于查证

> MIT 关于 ccache 说明：https://web.mit.edu/kerberos/krb5-devel/doc/basic/ccache_def.html

hadoop 开启kerberos 只能读取 /tmp/krb5cc_\`id -u\` 中的凭证。
在 Centos7 中的用户凭证 似乎 默认保存在 keyring 中， 执行hadoop命令时会出现 找不到凭证的异常。解决方式时 在每个结点添加环境变量：
```
 export KRB5CCNAME=FILE:/tmp/krb5cc_`id -u` 
```



### 1.7.3. 安装 CM/AGENT

> Cloudera Manager 版本和下载信息 <br/>
https://www.cloudera.com/documentation/enterprise/release-notes/topics/cm_vd.html

#### 1.7.3.1. CM安装方式
##### 1.7.3.1.1. rpm安装
> yum RHEL/CentOS/Oracle 7 <br/>
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/`${version}`/

可以直接使用数字版本号替换 version 值，达到直接下载 rpm 包的目的。

如果安装 5.8.2 版本，
在 https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.2/ 下载以下两个RPM包：
```
cloudera-manager-daemons-5.8.2-1.cm582.p0.17.el7.x86_64.rpm
cloudera-manager-server-5.8.2-1.cm582.p0.17.el7.x86_64.rpm
```

##### 1.7.3.1.2. yum安装
> yum repo <br/>
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo

或将CM源 加入 到 /etc/yum.repos.d/ 中, cloudera-manager.repo 内容通常为 cm 的最新版本,内容如下：

```
[cloudera-manager]
# Packages for Cloudera Manager, Version 5, on RedHat or CentOS 7 x86_64           	  
name=Cloudera Manager
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/
gpgkey =https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera    
gpgcheck = 1
```

这个repo表明是 CM 5 系列中 最新的版本，如果需要安装指定版本，如 5.8.2，则要修改 
```
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/
为 
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.2/
```
执行命令
```
# sudo yum install cloudera-manager-daemons cloudera-manager-server
```

##### 1.7.3.1.3. 启动 CM
```
# service cloudera-scm-server start
```

    启动成功后，访问 http://Server host:7180
    用户/密码: admin/admin

启动失败的原因可在 CM 安装结点上查看：
```
/var/log/cloudera-scm-server
```
###### 1.7.3.1.3.1. CM 启动过程中的问题

- JAVA_HOME not set

将 export JAVA_HOME =xxxx  追加到 /etc/default/cloudera-scm-server 中，这个文件 仅会影响到 cloudera manager的启动，不会影响其他的角设。并且 其他角色的 JAVA_HOME 可以在 CM 启动后，在CM 中进行设置 指定。

原文：
```
Cloudera Manager Server host: /etc/default/cloudera-scm-server. 
This affects only the Cloudera Manager Server process, and does not affect the Cloudera Management Service roles.
```

#### 1.7.3.2. 安装agent

##### 1.7.3.2.1. 安装过程中的问题

###### 1.7.3.2.1.1. 缺少DNS 服务器导致 指向ip地址错误
```
/etc/redhat-release ==> CentOS 7 
正在检测 Cloudera Manager Server...
BEGIN host -t PTR 192.168.100.101 
101.100.168.192.in-addr.arpa domain name pointer localhost. 
END (0) 
using localhost as scm server hostname 
BEGIN which python 
/usr/bin/python 
END (0) 
BEGIN python -c 'import socket; import sys; s = socket.socket(socket.AF_INET); s.settimeout(5.0); s.connect((sys.argv[1], int(sys.argv[2]))); s.close();' localhost 7182 
Traceback (most recent call last): 
File "<string>", line 1, in <module> 
File "/usr/lib64/python2.7/socket.py", line 224, in meth 
return getattr(self._sock,name)(*args) 
socket.error: [Errno 111] Connection refused 
```
- 一种不优雅的解决方式：在所有结点上执行  mv /usr/bin/host /usr/bin/host.bak
- 设置 DNS 解决这个问题。

###### 1.7.3.2.1.2. Cloudera Manager 正在获取安装锁

解决方法：进入出现问题结点，执行以下命令，再次安装
```
# cd /tmp
# ls -a
# rm -rf /tmp/scm_prepare_node.*
# rm -rf  /tmp/.scm_prepare_node.lock 
```

###### 1.7.3.2.1.3. 安装完成后，出现  ：host monitor时发生内部错误

可能的2种解决方法：
- 删除host monitor主机上/var/lib/cloudera-host-monitor/下所有文件

执行以下命令：

在 host monitor 所在结点上
```
# rm -rf /var/lib/cloudera-host-monitor/*
```

在CM 所在结点上
```
# service cloudera-scm-server restart
```

- 直接杀掉 supervisord 进程
```
# kill -9 $(pgrep -f supervisord)
# service cloudera-scm-agent restart
```

#### 1.7.3.3. Cloudera Manager 开启 kerberos认证

> 配置说明： https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_sg_s4_kerb_wizard.html#concept_qpj_x5y_l4

##### 1.7.3.3.1. 界面配置过程
- 在CM界面上 Domain Security Settings > Account Policies > Kerberos Policy
- 一直 Continue

完成后，安装任何服务都将默认开启kerberos认证。

##### 1.7.3.3.2. 创建hadoop hdfs用户
> https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_sg_s5_hdfs_principal.html

安装hdfs 完成后，在KDC 中会默认创建HDFS的principl，使用随机生成的密码，不能直接使用，需要配置新的 hdfs \<superuser\>.

```
Designating a Non-Default Superuser Group
To designate a different group of superusers instead of using the default hdfs account, follow these steps:

1. Go to the Cloudera Manager Admin Console and navigate to the HDFS service.
2.Click the Configuration tab.
3.Select Scope > HDFS (Service-Wide).
4.Select Category > Security.
5.Locate the Superuser Group property and change the value to the appropriate group name for your environment. For example, <superuser>.
6.Click Save Changes to commit the changes.
7.Restart the HDFS service.

```

如果在上面的执行过程 superhdfs 作为 hdfs superuser, 则在KDC中，加入名称为 superhdfs 的  principal.
```
kadmin:  addprinc hdfs@HADOOP.EXAPLE.COM
```

以superhdfs身份访问 hdfs时，执行下列命令并输入密码。
```
$ init superhdfs@HADOOP.EXAMPLE.COM
```


##### 1.7.3.3.3. 创建 hdfs 其他用户
通过 hdfs superuser 用户在 hdfs:/user/下 创建对应目录并改变目录拥有者即可。

##### 1.7.3.3.4. JAVA API 访问 HDFS

HDFS 开启kerberos 后， JAVA API 需要通过加载 keyTab 文件访问。
>生成 keytab 说明：https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cdh_sg_kadmin_kerberos_keytab.html

```
#生成包含单个principal的keytab
$ kadmin
kadmin:  xst -k test.keytab test@HADOOP.EXAMPLE.COM

#合并多个keytab
$ ktutil
ktutil:  rkt test1.keytab
ktutil:  rkt test2.keytab
ktutil:  wkt test.keytab #将 test1.keytab,test2.keytab 合并到 test.keytab
ktutil:  clear           #清空之前的内容

# 展示keytab 中的内容
$ klist -e -k -t hdfs.keytab

#从keytab中加载凭证
$ kinit -k -t hdfs.keytab hdfs/fully.qualified.domain.name@YOUR-REALM.COM

```


JAVA CODE：
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.security.UserGroupInformation;

import java.io.IOException;

public class KerberosTest {

    //OK的
    public static void main(String[] args) throws IOException {

        System.setProperty("java.security.krb5.conf", "/etc/krb5.conf");

        final String user = "test2@HADOOP.EXAMPLE.COM";
        final String keyPath = "/home/test/hdfs_java/test2.keytab";

        String hdfsUrl="hdfs://node1:8020";
        Configuration conf = new Configuration();

        //配置安全认证方式为kerberos
        //conf.setBoolean("hadoop.security.authorization", true);
        conf.set("hadoop.security.authentication", "Kerberos");

        conf.set("fs.defaultFS", hdfsUrl);
        conf.set("java.security.krb5.kdc", "  /var/kerberos/krb5kdc/kdc.conf");

        //设置datanode的principal值为“hdfs/_HOST@YOU-REALM.COM”
        conf.set("dfs.datanode.kerberos.principal", user);
        
        UserGroupInformation.setConfiguration(conf);
        
        UserGroupInformation.loginUserFromKeytab(user, keyPath);

        FileSystem fs=FileSystem.get(conf);

        Path root=new Path("/user/test2/");
        FileStatus[] statuss=fs.listStatus(root);

        for(FileStatus t:statuss){

            System.out.println(t.getPath().getName());

        }
        
    }
}
```

###### 【没有配置成功】添加 Java KeyStore KMS for transp encrypted 


> Cloudera Security: https://www.cloudera.com/documentation/enterprise/latest/topics/security.html

> Cloudera Security Overview

- Authentication
- Data in Transit Encryption
- Understanding Keystores and Truststores 理解概念
- Configuring Cloudera Manager Clusters for TLS/SSL : CM 内部通信加密
- - 用Keytool和OpenSSL生成和签发数字证书： http://blog.csdn.net/yhmhappy2006/article/details/6591076 
- Configuring TLS/SSL Encryption for CDH Services：各个服务之间加密
- Configuring TLS/SSL for Navigator Audit Server
- Configuring TLS/SSL for Navigator Metadata Server
- Configuring TLS/SSL for Kafka (Navigator Event Broker)

> 指导一些步骤如何去做
> https://www.cloudera.com/documentation/enterprise/latest/topics/sg_how_tos_intro.html

> How To Obtain and Deploy Keys and Certificates for TLS/SSL
> https://www.cloudera.com/documentation/enterprise/latest/topics/how_to_obtain_server_certs_tls.html

这里配置时失败的。。。。。。。再看下，下面的文档
https://www.cloudera.com/documentation/enterprise/latest/topics/cm_sg_create_key_trust.html


> Understanding Keystores and Truststores <br/>
> https://www.cloudera.com/documentation/enterprise/latest/topics/cm_sg_create_key_trust.html

> Configuring Encrypted HDFS Data Transport <br/>
> https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_sg_hdfs_encrypt_transport.html
- 添加 KMS 服务
- 密钥管理员用户/组：root/root
- 设置 jks位置 /var/lib/jks,密码:1234
- 生成ACL

- 设置

Secure DataNode configuration is invalid. There are two recommended configurations: (1) DataNode Transceiver Port and Secure DataNode Web UI Port (TLS/SSL) both >= 1024, DataNode Data Transfer Protection set, Hadoop TLS/SSL enabled; (2) DataNode Transceiver Port and DataNode HTTP Web UI Port both < 1024, DataNode Data Transfer Protection not set, Hadoop TLS/SSL disabled.

DataNode 收发器端口
dfs.datanode.address

安全 DataNode Web UI 端口 (TLS/SSL)
dfs.datanode.https.address



###### 1.7.3.3.4.1. CM 部分设置说明
- 添加自定义参数： 配置-> 高级配置代码段
- cluster->service-> 配置-> 类别 -> 高级


### 1.7.4. 安装Kylin
> http://kylin.apache.org/docs21/install/index.html

```
After Kylin started you can visit http://hostname:7070/kylin. 
The default username/password is ADMIN/KYLIN. It’s a clean Kylin homepage with nothing in 
http://hostname:7070/kylin
```

### 1.7.5. 运行过程中的一些问题

#### 1.7.5.1. reduce shuffle throws "java.lang.OutOfMemoryError: Java heap space
> https://issues.apache.org/jira/browse/MAPREDUCE-6447

```
by default:
mapreduce.reduce.shuffle.input.buffer.percent=0.9
mapreduce.reduce.shuffle.memory.limit.percent=0.25
mapreduce.reduce.shuffle.parallelcopies=5

0.9*0.25*5=1.125>1.
```
可以将 mapreduce.reduce.shuffle.memory.limit.percent 变小解决

#### 1.7.5.2. HDFS坏块的恢复
> HDFS 参数说明： http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

> http://www.cnblogs.com/prayer21/p/4819789.html

|name                          | default value | description |
|------------------------------|---------------|----------------------|
| dfs.blockreport.intervalMsec | 21600000      | Determines block reporting interval in milliseconds. 向NameNode 报告datablock 周期（毫秒）|
| dfs.datanode.directoryscan.interval | 21600  | Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk. 内存和磁盘数据集块校验周期 |        
| dfs.datanode.directoryscan.threads | 1       | How many threads should the threadpool used to compile reports for volumes in parallel have. 内存和磁盘周期性校验线程并发数量。 |
|具备损坏副本的块监控阈值|||
||||

dfs.blockreport.intervalMsec，dfs.datanode.directoryscan.interval 默认值都是6小时，可修改为10分钟。






