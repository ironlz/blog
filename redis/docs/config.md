# Redis配置项
本章介绍关于redis配置文件的一些概述  
## 配置文件
&emsp;&emsp;redis可以不指定配置文件启动，其内部内嵌了默认参数，但是这种方式只建议在测试或开发场景下使用。  
&emsp;&emsp;恰当的方式是提供一个单独的 ***redis.conf*** 文件，来记录redis的配置项。名字一般叫redis.conf，当然也可以使用其他名字  
redis.conf由一系列格式简单的配置项组成：  
&emsp;&emsp;`keyword argument1 argument2 ... argumentN`  
&emsp;&emsp;当参数内部有空格时，需要将参数通过单引号或者双引号括起来。  
&emsp;&emsp;单引号支持由反斜杠开头的转义字符，双引号则除了转义字符外，还支持使用\x十六进制编码的任意字符。  
&emsp;&emsp;redis的配置文件及配置项说明可以在各自的发行版中的redis.conf文件中找到
&emsp;&emsp;通过命令行参数传递配置项  
&emsp;&emsp;除了配置文件外，也可以在redis启动时，通过命令行参数来指定配置项。命令行参数支持的种类和格式与配置文件中完全一致  
`./redis-server --port 6380 --replicaof 127.0.0.1 6379`  
&emsp;&emsp;***需要注意的是通过命令行传递的配置项都在内存中的临时配置文件中，非持久化***
## 运行过程中改变redis配置  
&emsp;&emsp;服务运行中可以通过CONFIG GET/SET命令来获取/设置配置文件，不用对服务进行重启  
&emsp;&emsp;需要注意的是不是所有的参数都支持热更新，具体可以查看CONFIG GET/SET的命令详情  
&emsp;&emsp;通过这种方式所做的改变都是非持久化的，不会影响redis.conf配置文件，下次启动时依然以redis.conf中的配置项为准  
&emsp;&emsp;***可以通过CONFIG REWRITE命令，来更新redis.conf文件***。 注释不会改动，值为默认值的配置项不会新增，修改的配置项会自动修改，新增的会自动追加
## 将redis配置为缓存服务器  
&emsp;&emsp;如果将redis作为缓存服务器，可以通过设置maxmemory来指定最大缓存大小，通过指定maxmemory-policy allkeys-lru来指定内存淘汰策略  
&emsp;&emsp;这样一来应用程序就不需要再通过EXPIRE指定过期时间，当缓存空间不够时，redis会对所有的key执行lru算法  

# 配置文件详解
&emsp;&emsp;各版本配置文件示例  
&emsp;&emsp;&emsp;&emsp;[redis.conf(2.4)](../reference/redis-24.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(2.6)](../reference/redis-26.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(2.8)](../reference/redis-28.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(3.0)](../reference/redis-30.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(3.2)](../reference/redis-32.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(4.0)](../reference/redis-40.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(5.0)](../reference/redis-50.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(6.0)](../reference/redis-60.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(6.2)](../reference/redis-62.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(7.0)](../reference/redis-70.conf "redis.conf")  
&emsp;&emsp;&emsp;&emsp;[redis.conf(7.2)](../reference/redis-72.conf "redis.conf")  

## &emsp;配置项详解
### &emsp;&emsp;内存单位
&emsp;&emsp;内存支持数字或者带单位的配置，带b的话为1024进制，不带则为十进制，最小单位都为byte  
&emsp;&emsp;&emsp;&emsp;1k => 1000byte  
&emsp;&emsp;&emsp;&emsp;1kb => 1024byte  
&emsp;&emsp;&emsp;&emsp;1000 => 1000byte  
  
## &emsp;&emsp;daemonize  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：no  
&emsp;&emsp;&emsp;&emsp;content：`daemonize no`  
&emsp;&emsp;&emsp;&emsp;用于配置redis是否以守护进程(后台进程)的形式执行，如果配置为yes，则redis会将进程信息写入到/var/run/redis.pid文件中，该地址可以通过pidfile来进行修改

## &emsp;&emsp;pidfile  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：/var/run/redis.pid  
&emsp;&emsp;&emsp;&emsp;content：`pidfile /var/run/redis.pid`  
&emsp;&emsp;&emsp;&emsp;当redis以守护进程的形式运行时，则redis会将进程信息写入到/var/run/redis.pid文件中，可以通过该配置项修改其地址，一般建议以redis_\<port>.pid的形式来命名，从而避免在一个机器上启动多个redis-server互相影响  

## &emsp;&emsp;port  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：6379  
&emsp;&emsp;&emsp;&emsp;content：`port 6379`  
&emsp;&emsp;&emsp;&emsp;redis的对外监听端口，默认是6379，如果配置为0，则redis不会对外暴露socket端口，一般配合tls-port使用，用来关闭非加密连接的监听，从而只使用tls连接  
  
## &emsp;&emsp;bind  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：127.0.0.1 -::1  
&emsp;&emsp;&emsp;&emsp;content：`bind 192.168.1.100 10.0.0.1`  
&emsp;&emsp;&emsp;&emsp;redis绑定的网卡，默认情况下bind配置项是被注释掉的，默认为绑定所有网卡，等价于配置\* -::\* (前一个\*为绑定所有的IPV4类型的网卡，后一个-::\*意思为绑定所有的IPV6网卡)  
&emsp;&emsp;&emsp;&emsp; `bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses`  
&emsp;&emsp;&emsp;&emsp; `bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6`  
&emsp;&emsp;&emsp;&emsp; `* -::*                     # like the default, all available interfaces`  
&emsp;&emsp;&emsp;&emsp;默认的绑定到所有网卡存在一定的安全隐患，因此redis配置文件中默认将其绑定到了回环地址，但是需要明确的是，其默认行为是绑定到所有网卡

## &emsp;&emsp;unixsocket与unixsocketperm  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：`unixsocket /tmp/redis.sock`  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; `unixsocketperm 755`  
&emsp;&emsp;&emsp;&emsp;content：`unixsocket /tmp/redis.sock`  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; `unixsocketperm 755`  
&emsp;&emsp;&emsp;&emsp;redis默认使用tcp对外提供服务，可以通过设置unxisocket来使redis通过unixsocket提供服务，这种方式绕过了tcp的协议层，因此可以获得更好的性能，但是需要注意的是这种方式只适合本地访问，即服务与redis处于同一个服务器上时，使用这种方式比默认的方式可以获得大约30%的性能提升。客户端可以使用redis-cli -s \<unixsocket>来进行连接，代码中使用unix:\<unixsocket>进行连接。当配置了unixsocket时可以同时配置unixsocket的权限，一般为755

## &emsp;&emsp;timeout  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：0  
&emsp;&emsp;&emsp;&emsp;content：`timeout 0`  
&emsp;&emsp;&emsp;&emsp;客户端空闲连接超时时间，单位为秒，默认为0（不超时）  

# &emsp;&emsp;loglevel  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：verbose  
&emsp;&emsp;&emsp;&emsp;content：`loglevel verbose`  
&emsp;&emsp;&emsp;&emsp;设置redis服务端的日志级别  
|级别|信息|
|--|--|
|debug|适用于开发、测试模式，会打印尽可能多的信息|    
|verbose|打印的信息比debug少一些，打印一些必要信息|  
|notice|打印的信息更少，主要是一些需要关注的警告及报错信息|  
|warning|只打印非常重要/致命的信息|  

## &emsp;&emsp;logfile  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：stdout  
&emsp;&emsp;&emsp;&emsp;content：`logfile stdout`  
&emsp;&emsp;&emsp;&emsp;指定redis-server的日志打印文件位置，如果配置成stdout的话，会打印到系统标准输出流，如果配置了daemonize的话，日志将会被发送到/dev/null  

## &emsp;&emsp;syslog-enabled  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：no  
&emsp;&emsp;&emsp;&emsp;content：`syslog-enabled no`  
&emsp;&emsp;&emsp;&emsp;启用syslog，关于syslog的配置可以参考[redis启用syslog或dockerlog]("https://betterstack.com/community/guides/logging/how-to-start-logging-with-redis/#step-1-configuring-logging-with-syslog")  

## &emsp;&emsp;syslog-ident  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：redis  
&emsp;&emsp;&emsp;&emsp;content：`syslog-ident redis`  
&emsp;&emsp;&emsp;&emsp;启用syslog后，配置当前进程日志的唯一标识，关于syslog的配置可以参考[redis启用syslog或dockerlog]("https://betterstack.com/community/guides/logging/how-to-start-logging-with-redis/#step-1-configuring-logging-with-syslog")  

## &emsp;&emsp;syslog-facility  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：local0  
&emsp;&emsp;&emsp;&emsp;content：`syslog-facility local0`  
&emsp;&emsp;&emsp;&emsp;启用syslog后，配置输出级别，可以参考[What are Syslog Facilities and Levels]("https://success.trendmicro.com/dcx/s/solution/TP000086250?language=en_US")  


## &emsp;&emsp;databases  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：16  
&emsp;&emsp;&emsp;&emsp;content：`databases 16`  
&emsp;&emsp;&emsp;&emsp;配置redis的数据库数量

## &emsp;&emsp;save  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：3600 1 300 100 60 10000  
&emsp;&emsp;&emsp;&emsp;content：`save 3600 1 300 100 60 10000`  
&emsp;&emsp;&emsp;&emsp;配置redis的持久化策略，默认是关闭的（被注释掉了），语法为save \<seconds\> \<changes\> \[\<seconds\> \<changes\> ...\]，也可以配置成多行，第一个参数表示持久化冷却时间，第二个参数表示至少需要发生多少次写入。任意一个条件满足即会持久化数据。  

## &emsp;&emsp;rdbcompression  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：yes  
&emsp;&emsp;&emsp;&emsp;content：`rdbcompression yes`  
&emsp;&emsp;&emsp;&emsp;使用lzf压缩算法来压缩rdb文件，启用会导致cpu消耗增加，关闭会导致rdb文件体积增大。  

## &emsp;&emsp;dbfilename  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：dump.rdb  
&emsp;&emsp;&emsp;&emsp;content：`dbfilename dump.rdb`  
&emsp;&emsp;&emsp;&emsp;rdb文件的名称，注意这里不能配置成路径，仅仅是文件名称，保存路径需要通过dir配置参数配置。  

## &emsp;&emsp;dir  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：./  
&emsp;&emsp;&emsp;&emsp;content：`dir ./`  
&emsp;&emsp;&emsp;&emsp;工作文件目录，rdb文件和aof文件都会被保存到该目录下。  

## &emsp;&emsp;slaveof  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：无  
&emsp;&emsp;&emsp;&emsp;content：`slaveof <masterip> <masterport>`  
&emsp;&emsp;&emsp;&emsp;主从复制配置项，将当前实例作为从实例启动，从实例也是一个独立的实例，其也可以配置自己的数据持久化策略、对外暴露端口等  

## &emsp;&emsp;masterauth  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：无  
&emsp;&emsp;&emsp;&emsp;content：`masterauth <master-password>`  
&emsp;&emsp;&emsp;&emsp;主从复制配置项，如果master需要密码健全，则此处需要配置主实例的访问密码

## &emsp;&emsp;slave-serve-stale-data  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：yes  
&emsp;&emsp;&emsp;&emsp;content：`slave-serve-stale-data yes`  
&emsp;&emsp;&emsp;&emsp;当主从实例连接丢失时，副本的对外策略：  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1、当slave-serve-stale-data设置为yes时，副本节点会以当前节点的数据响应客户端仓库，需要注意此时的数据可能与主实例的存在差异（过期数据）  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2、当slave-serve-stale-data设置为false时，副本节点会返回给客户端"SYNC with master in progress"错误，但是INFO和SLAVEOF命令依然可以正常执行

## &emsp;&emsp;repl_ping_slave_period  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：10  
&emsp;&emsp;&emsp;&emsp;content：`repl-ping-slave-period 10`  
&emsp;&emsp;&emsp;&emsp;副本心跳发送间隔，单位为s，副本会以该参数指定的时间间隔向master发送PING命令

## &emsp;&emsp;repl-timeout  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：60  
&emsp;&emsp;&emsp;&emsp;content：`repl-timeout 60`  
&emsp;&emsp;&emsp;&emsp;主副本超时时间，单位为s；该超时时间控制几乎所有的主副本之间的操作，例如主副本数据传递的IO超时、主实例数据或PING响应超时时间等。一个很重要的点是这个时间应该比repl_ping_slave_period大，否则当主从之间流量很低时，slave会认为主从超时

## &emsp;&emsp;slave-priority  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：100  
&emsp;&emsp;&emsp;&emsp;content：`slave-priority 100`  
&emsp;&emsp;&emsp;&emsp;副本优先级，该参数是sentinel用来选主时的参考，sentinel会选择优先级最小的slave作为新的master，但是需要注意的是0代表不参与选举，即sentinel永远不会选择一个slave-priority为0的slave实例作为新master

## &emsp;&emsp;requirepass  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：无  
&emsp;&emsp;&emsp;&emsp;content：`requirepass foobared`  
&emsp;&emsp;&emsp;&emsp;redis密码，配置了该选项后，客户端访问redis时会要求使用AUTH鉴权才能进行后续操作。需要注意外部用户可以以高达15w/s的频率尝试密码，因此要保证配置的密码是一个随机强密码，否则很容易被破解

## &emsp;&emsp;rename-command  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：无  
&emsp;&emsp;&emsp;&emsp;content：`rename-command CONFIG ""`  
&emsp;&emsp;&emsp;&emsp;redis密码，配置了该选项后，客户端访问redis时会要求使用AUTH鉴权才能进行后续操作。需要注意外部用户可以以高达15w/s的频率尝试密码，因此要保证配置的密码是一个随机强密码，否则很容易被破解