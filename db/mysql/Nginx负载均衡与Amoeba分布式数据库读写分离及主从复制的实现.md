

# Nginx负载均衡与Amoeba分布式数据库读写分离及主从复制的实现

# 前言

本文主要讲解Nginx负载均衡与Amoeba分布式数据库及读写分离的实现。Nginx负载均衡部署部署分布式服务器，将相同的应用部署到多台机器上。 解决访问统一入口问题，在集群前面增加**负载均衡**设备，实现流量分发。另外使用Amoeba实现数据库**读写分离**及**主从复制**的实现。

![gratisography-157-thumbnail](img/Nginx负载均衡与Amoeba分布式数据库读写分离及主从复制的实现\gratisography-157-thumbnail.jpg)

# 前提

- JDK8
- Nginx
- MySQL
- Tomcat
- Amoeba
- 至少3台节点服务器

# 环境配置

## 安装JDK8

官网下载JDK的rpm包

```sh
rpm -ivh jdk8*****.rpm
```

配置环境变量

```shell
vim /etc/profile
#最后添加如下，适当调整自己路径名称
export JAVA_HOME=/usr/java/jdk1.8***
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

#最后刷新配置
source /etc/profile
```



## 安装Nginx

```sh
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
yum install -y nginx
systemctl start nginx.service
```



## 安装MySQL

```shell
wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum repolist all | grep mysql
yum install mysql-community-server
```



## 安装Tomcat

官网下载Tomcat，然后配置

## 安装Amoeba

下载[Amoeba](https://sourceforge.net/projects/amoeba/files/)

配置Amoeba环境变量

```shell
vim /etc/profile
#最后添加如下，路径根据自己环境进行适当调整
export AMOEBA_HOME=/usr/local/amoeba/amoeba-mysql-3.0.5-RC
export PATH=$PATH:$AMOEBA_HOME/bin

#最后刷新配置
source /etc/profile
```

修改JVM内存设置

```shell
vim /opt/amoeba-mysql/jvm.properties
#找到JVM_OPTIONS并修改成如下
JVM_OPTIONS="-server -Xms1024m -Xmx1024m -Xss256k -XX:PermSize=16m -XX:MaxPermSize=96m"
```

# 负载均衡

## 系统架构

![Nginx负载均衡](img/Nginx负载均衡与Amoeba分布式数据库读写分离及主从复制的实现/Nginx%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png)

## Nginx配置

Nginx核心配置

```sh
upstream huyd.com {
        server  172.16.1.7      weight=5;
        server  172.16.1.8      weight=5;
}


server {
    listen       80;
    server_name  huyd.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
        proxy_pass      http://huyd.com;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```



# 分布式数据库

系统：Centos 7

JDK：版本1.8及以上

MySQL：版本5.7

Amoeba：Amoeba-mysql-3.0.5-RC

Amoeba服务器：172.16.1.6

主库地址：172.16.1.7

主库地址：172.16.1.8

从库地址：172.16.1.9

## 系统架构

![Amoeba分布式数据库](img/Nginx负载均衡与Amoeba分布式数据库读写分离及主从复制的实现/Amoeba%E5%88%86%E5%B8%83%E5%BC%8F%E6%95%B0%E6%8D%AE%E5%BA%93.png)

## 数据库主从复制

如上架构如中，实现了**一主一从**架构，关于多主单从或者一主多从情况，根据架构同理设计如下配置。

### master配置

```shell
vim /etc/my.cnf
#添加如下操作
[mysqld]
log-bin=mysql-bin
server-id=1
```

说明: 

1. log-bin:开启二进制日志，该日志是在事务提交时写日志文件的。默认大小是1G，后面加001,002这样的后缀顺加。 
2. server-id，唯一标识主机，mysql主从每个mysql实例配置都不一样就行。这个值默认是0，如果是0，主服务器拒绝任何从服务器的连接。 
3. innodb_flush_log_at_trx_commit=1 and sync_binlog=1 这两个参数控制着二进制日志刷新速度，后面再写文章单独分析。 
4. binlog_format=ROW控制着日志格式。



### slave配置

```shell
vim /etc/my.cnf
#添加如下操作
[mysqld]
server-id=2
```

说明: 
server-id唯一就行。如果默认为0，则拒绝连接主服务器。



### master创建slave复制用户

```shell
CREATE USER 'repl'@'192.168.31.%' IDENTIFIED BY '!password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'172.16.1.%';
```

说明： 
简单的创建用户，授予REPLICATION SLAVE权限。访问限制，密码，用户名等，根据实际情况自行设定，后面注意保持一致。



### 获取master日志坐标

进入master

```shell
FLUSH TABLES WITH READ LOCK;
#再开一个会话，连接mysql,执行
SHOW MASTER STATUS;
#输出如下信息
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      465 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

如上，记录mysql-bin.000001  465

说明 
1.要开另外的一个连接执行SHOW MASTER STATUS,是由于假如断开第一个执行FLUSH TABLES的会话，会自动释放读锁，还是小心点，重开会话吧。 
2.第二 个会话输出，有可能为空，如果是全新安装，还没有产生二进制日志，你还是先随便执行点修改操作，后面就有了，一般的select是不记录二进制日志的。



### 创建master数据快照

```shell
mysqldump --all-databases --master-data -uroot -p > /tmp/dbdump.db
```

### 创建slave复制

**master**，因为我们已经取到master上一致性的快速，可以解锁了，在master上执行

```shell
UNLOCK TABLES;
```

**slave**

```shell
#上传master是快照备份dbdump.db到slave的/tmp下

#salve的my.cnf配置mysqld块中增加
skip-slave-start=true
read_only=ON
relay-log=relay-bin
relay-log-index=relay-bin.index

#说明:skip-slave-start=true,跳过slave线程启动
#    其他的中断日志配置，就方便复制到其他salve,因为默认是主机名开关的文件。
#    read_only，我开启的只读模式

#启动slave
#    此时进入mysql,show processlist;是看不到复制线程的，show slave status \G显示也是空。

#设置master信息
CHANGE MASTER TO
MASTER_HOST='172.16.1.7',
MASTER_USER='repl',
MASTER_PASSWORD='!password',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=456;

#最后的MASTER_LOG_FILE='mysql-bin.000004',
#MASTER_LOG_POS=154可以省略，因为后面的倒入数据中如果有CHANGE MASTER TO MASTER_LOG_FILE='mysql-#bin.000004', MASTER_LOG_POS=154; 这样的语句。如果是手工冷备份过来的，则不能省略。

#为了方便，统一都写上的，哪种情况都合适，省得自己都晕了,就当上面那段话是空气，忘记吧。

#倒入数据到slave
bin/mysql -uroot -p < /tmp/dbdump.db 

6.查看slave状态
在slave中执行
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 172.16.1.7
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 1753
               Relay_Log_File: relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
...

我们看到
    Slave_IO_Running: No
    Slave_SQL_Running: No
复制线程还没有启动

7.手工启动slave复制线程
start slave

8.再次查看下slave状态
show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.16.1.7
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 1753
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
...

这回都YES，配置完成,顺便看看进程
mysql> show processlist;
+----+-------------+------------------+----------+---------+------+--------------------------------------------------------+------------------+
| Id | User        | Host             | db       | Command | Time | State                                                  | Info             |
+----+-------------+------------------+----------+---------+------+--------------------------------------------------------+------------------+
|  2 | amoeba      | 172.16.1.6:56058 | amoebadb | Sleep   |  505 |                                                        | NULL             |
|  8 | root        | localhost        | test_db  | Query   |    0 | starting                                               | show processlist |
|  9 | system user |                  | NULL     | Connect |   16 | Waiting for master to send event                       | NULL             |
| 10 | system user |                  | NULL     | Connect |   16 | Slave has read all relay log; waiting for more updates | NULL             |
+----+-------------+------------------+----------+---------+------+--------------------------------------------------------+------------------+

多了两个复制的进程
```

最后，注释跳过slave的配置

```shell
vim /etc/my.cnf
#找到如下行添加注释
#skip-slave-start=true

systemctl restart mysqld.service
#进入mysql
stop slave
start slave
```

## 测试

master中更新数据

```shell
create database test;
create table user (id int,name varchar(20));
insert into user values (1,'test');
select * from user;
```

slave上查询数据

```shell
show databases;
use test;
select * from user;
```

到这里，不出意外，应该可以实现主从同步数据了。本人测试为秒级同步，在大数据下没有测试过，目前不知道同步延迟率多高。



# 数据库读写分离

## Amoeba授权mysql远程账户

如下在主从机器上执行，为Amoeba授权访问权限

```shell
mysql -u root -p
mysql> grant all privileges on *.* to amoeba@"%" identified by "!password" WITH GRANT OPTION;
mysql> flush privileges;
```

## Amoeba配置

```shell
vim /opt/amoeba-mysql/conf/dbServers.xml
#如下配置
<dbServer name="abstractServer" abstractive="true">
                <factoryConfig class="com.meidusa.amoeba.mysql.net.MysqlServerConnectionFactory">
                        <property name="connectionManager">${defaultManager}</property>
                        <property name="sendBufferSize">64</property>
                        <property name="receiveBufferSize">128</property>

                        <!-- mysql port -->
                        <property name="port">3306</property>	#数据库端口

                        <!-- mysql schema -->
                        <property name="schema">amoebadb</property>	#数据库名称

                        <!-- mysql user -->
                        <property name="user">amoeba</property>	#用于连接mysql的用户名

                        <property name="password">!password</property>	#用于连接mysql的密码
                </factoryConfig>

                <poolConfig class="com.meidusa.toolkit.common.poolable.PoolableObjectPool">
                        <property name="maxActive">500</property>
                        <property name="maxIdle">500</property>
                        <property name="minIdle">1</property>
                        <property name="minEvictableIdleTimeMillis">600000</property>
                        <property name="timeBetweenEvictionRunsMillis">600000</property>
                        <property name="testOnBorrow">true</property>
                        <property name="testOnReturn">true</property>
                        <property name="testWhileIdle">true</property>
                </poolConfig>
        </dbServer>
        
         <dbServer name="master"  parent="abstractServer">
                <factoryConfig>
                        <!-- mysql ip -->
                        <property name="ipAddress">172.16.1.7</property>	#主机
                </factoryConfig>
        </dbServer>

        <dbServer name="slave"  parent="abstractServer">
                <factoryConfig>
                        <!-- mysql ip -->
                        <property name="ipAddress">172.16.1.8</property>	#从机
                </factoryConfig>
        </dbServer>

        <dbServer name="multiPool" virtual="true">
                <poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
                        <!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
                        <property name="loadbalance">1</property>

                        <!-- Separated by commas,such as: server1,server2,server1 -->
                        <property name="poolNames">slave,master</property>	#设置机器轮询顺序
                </poolConfig>
        </dbServer>


```





```shell
vim /opt/amoeba-mysql/conf/amoeba.xml
#如下配置
<amoeba:configuration xmlns:amoeba="http://amoeba.meidusa.com/">

        <proxy>

                <!-- service class must implements com.meidusa.amoeba.service.Service -->
                <service name="Amoeba for Mysql" class="com.meidusa.amoeba.mysql.server.MySQLService">
                        <!-- port -->
                        <property name="port">8066</property>	#Amoeba端口，开发人员使用

                        <!-- bind ipAddress -->
                        <!-- 
                        <property name="ipAddress">127.0.0.1</property>
                         -->

                        <property name="connectionFactory">
                                <bean class="com.meidusa.amoeba.mysql.net.MysqlClientConnectionFactory">
                                        <property name="sendBufferSize">128</property>
                                        <property name="receiveBufferSize">64</property>
                                </bean>
                        </property>

                        <property name="authenticateProvider">
                                <bean class="com.meidusa.amoeba.mysql.server.MysqlClientAuthenticator">

                                        <property name="user">root</property>	#用户名，开发使用，外部连接

                                        <property name="password">!password</property>#密码，开发使用，外部连接

                                        <property name="filter">
                                                <bean class="com.meidusa.toolkit.net.authenticate.server.IPAccessController">
                                                        <property name="ipFile">${amoeba.home}/conf/access_list.conf</property>
                                                </bean>
                                        </property>
                                </bean>
                        </property>

                </service>

                <runtime class="com.meidusa.amoeba.mysql.context.MysqlRuntimeContext">

                        <!-- proxy server client process thread size -->
                        <property name="executeThreadSize">128</property>

                        <!-- per connection cache prepared statement size  -->
                        <property name="statementCacheSize">500</property>

                        <!-- default charset -->
                        <property name="serverCharset">utf8</property>

                        <!-- query timeout( default: 60 second , TimeUnit:second) -->
                        <property name="queryTimeout">60</property>
                </runtime>

        </proxy>

        <!-- 
                Each ConnectionManager will start as thread
                manager responsible for the Connection IO read , Death Detection
        -->
        <connectionManagerList>
                <connectionManager name="defaultManager" class="com.meidusa.toolkit.net.MultiConnectionManagerWrapper">
                        <property name="subManagerClassName">com.meidusa.toolkit.net.AuthingableConnectionManager</property>
                </connectionManager>
        </connectionManagerList>

                <!-- default using file loader -->
        <dbServerLoader class="com.meidusa.amoeba.context.DBServerConfigFileLoader">
                <property name="configFile">${amoeba.home}/conf/dbServers.xml</property>
        </dbServerLoader>

        <queryRouter class="com.meidusa.amoeba.mysql.parser.MysqlQueryRouter">
                <property name="ruleLoader">
                        <bean class="com.meidusa.amoeba.route.TableRuleFileLoader">
                                <property name="ruleFile">${amoeba.home}/conf/rule.xml</property>
                                <property name="functionFile">${amoeba.home}/conf/ruleFunctionMap.xml</property>
                        </bean>
                </property>
                <property name="sqlFunctionFile">${amoeba.home}/conf/functionMap.xml</property>
                <property name="LRUMapSize">1500</property>
                <property name="defaultPool">master</property>#默认的数据库节点，除了增删改查以外所有语句都会在默认库执行


                <property name="writePool">master</property>#写库
                <property name="readPool">slave</property>#读库

                <property name="needParse">true</property>
        </queryRouter>
</amoeba:configuration>

```

## 登录测试

进行读写操作，可以发现读写使用的并不是同一个数据库。

```
mysql -uroot -p!password -h172.16.1.6 -P8066
```

至此，成功。

