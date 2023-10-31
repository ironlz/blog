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

## &emsp;&emsp;tcp-keepalive  
&emsp;&emsp;&emsp;&emsp;sinece：2.6  
&emsp;&emsp;&emsp;&emsp;default：0  
&emsp;&emsp;&emsp;&emsp;content：`tcp-keepalive 0`  
&emsp;&emsp;TCP连接的keepalive时间，单位为s，如果该配置项设置的为非零值，则通过设置SO_KEEPALIVE来进行客户端和服务端之间的连接保活，这样有两个好处：  
&emsp;&emsp;1）检测不活跃的客户端  
&emsp;&emsp;2）使连接在网络设备的角度上看使活的  
&emsp;&emsp;在linux系统中，这个指定的值用来决定ACKs的发送周期，需要注意如果配置了这个，那么关闭一个连接的耗时就变成了这个时间的两倍。在其它的操作系统上，这个周期取决于内核配置。一个比较合理的值是将其设置为60s

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
&emsp;&emsp;&emsp;&emsp;重命名命令，在不可信环境下可以讲一些敏感命令重命名为不容易被猜到的命令，从而规避通用客户端采用通用命令来执行相关操作

## &emsp;&emsp;maxclients  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：0  
&emsp;&emsp;&emsp;&emsp;content：`maxclients 128`  
&emsp;&emsp;&emsp;&emsp;最大连接数，默认为0（无限制），redis会在连接数达到这个值后，向新建连接返回‘max number of clients reached’报错

## &emsp;&emsp;maxmemory  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：无  
&emsp;&emsp;&emsp;&emsp;content：`maxmemory <bytes>`  
&emsp;&emsp;&emsp;&emsp;redis使用的最大内存大小。当redis使用的内存超过该配置项后，redis会尝试通过内存驱逐策略（maxmemmory-policy）来删除key从而释放内存，当redis无法移除key或者内存策略被设置为noeviction时，redis会只响应读取命令，对于写入命令会报内存不足的错误。  
&emsp;&emsp;&emsp;&emsp;需要注意如果实例存在从实例时，配置该选项需要讲主从复制所需要的缓冲区内存剔除掉，否则可能会导致实例进入一个驱逐循环：网络问题或主从同步时，key内存+outbuffer超过占用内存大小，实例执行key驱逐，如此循环直至实例清空。简单说如果实例存在slave，则需要尽量调高这个maxmemory占用，为主从同步的outbuffer留下空间，同时也需要为操作系统留下足够的RAM空间供数据传输使用（驱逐策略为noeviction时不需要考虑这一点）

## &emsp;&emsp;maxmemory-policy  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：volatile-lru  
&emsp;&emsp;&emsp;&emsp;content：`maxmemory-policy volatile-lru`  
&emsp;&emsp;&emsp;&emsp;该配置项用于配置内存驱逐策略：  
|策略名称|策略行为|
|--|--|
|volatile-lru|在设置了过期时间的key范围内，淘汰最久未使用的key|
|allkeys-lru|对所有的key执行LRU算法|
|volatile-random|在设置了过期时间的key范围内，随机淘汰|
|allkeys->random|对所有的key随机淘汰|
|volatile-ttl|在设置了过期时间的key中，淘汰最接近过期时间的key|
|noeviction|不淘汰|  

## &emsp;&emsp;maxmemory-samples  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：3  
&emsp;&emsp;&emsp;&emsp;content：`maxmemory-samples 3`  
&emsp;&emsp;&emsp;&emsp;redis驱逐策略所选择的key并不是绝对精准的，而是在选择范围内通过采样来确定要删除哪些key，因此可以通过修改该参数，来调整redis每次采样时选择的样本集的大小

## &emsp;&emsp;appendonly  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：no  
&emsp;&emsp;&emsp;&emsp;content：`appendonly no`  
&emsp;&emsp;&emsp;&emsp;是否开启AOF策略，AOF是redis的另一种持久化策略，可以和RDB持久化同时使用，RDB会根据save配置的持久化策略定期将整个数据集持久化到硬盘，因为持久化过程比较重，因此一般适用于允许部分数据丢失的场景（两次持久化间隔中发生CRASH时会导致写入的数据丢失）。AOF持久化策略则适用于不希望丢失任何数据的场景，但是因为写入命令不停的持久化到AOF文件中，因此可能会导致持久化文件很大，因此需要注意BGREWRITEAOF的配置，它可以在AOF文件过大时重写AOF文件。

## &emsp;&emsp;appendfilename  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：appendonly.aof  
&emsp;&emsp;&emsp;&emsp;content：`appendfilename appendonly.aof`  
&emsp;&emsp;&emsp;&emsp;配置AOF持久化文件的名称，默认为appendonly.aof

## &emsp;&emsp;appendfsync  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：everysec  
&emsp;&emsp;&emsp;&emsp;content：`appendfsync everysec`  
&emsp;&emsp;&emsp;&emsp;AOF的写入策略：当符合策略时，redis会调用fsync()来通知系统将缓存中的数据写入到磁盘，一些系统会立即执行，但是也有的系统会尽可能去执行   
|策略名称|意义|
|--|--|
|no|不会主动调用fsync通知系统持久化缓冲区数据，有操作系统自行决定何时flush，持久化速度快，但是依然存在数据丢失的可能性|
|always|每当有写入命令被执行时，立即调用fsync持久化该数据，会造成大量的磁盘写入，因此速度最慢，但是最安全|
|everysec|两次fsync的调用间隔不会小于1s，数据安全和写入速度的折中选项|


## &emsp;&emsp;no-appendfsync-on-rewrite  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：no  
&emsp;&emsp;&emsp;&emsp;content：`no-appendfsync-on-rewrite no`  
&emsp;&emsp;&emsp;&emsp;当AOF的fsync策略不为no时，那么当一个后台存储进程（如save或AOF重写时），磁盘将面临大量的IO操作，在一些linux配置中，redis将会阻塞在fsync操作中，此时没有办法可以使redis恢复。因此为了解决这个问题，可以在BGSAVE或BGREWRITEAOF执行过程中禁用fsync。这意味着当另一个子进程在执行持久化时，执行期间redis的fsync策略将退化为no，在linux的默认策略下，可能存在最多30s数据丢失的隐患。只有在redis实例面临延迟问题时才考虑设置该选项为yes，否则保持默认值no就可以了。

## &emsp;&emsp;auto-aof-rewrite-percentage  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：100  
&emsp;&emsp;&emsp;&emsp;content：`auto-aof-rewrite-percentage 100`  
&emsp;&emsp;&emsp;&emsp;当AOF文件增长超过多少时重写

## &emsp;&emsp;auto-aof-rewrite-min-size  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：64mb  
&emsp;&emsp;&emsp;&emsp;content：`auto-aof-rewrite-min-size 64mb`  
&emsp;&emsp;&emsp;&emsp;AOF重写的基准大小。   
&emsp;&emsp;&emsp;&emsp;AOF重写可以通过手动调用BGREWRITEAOF命令实现，也可以通过配置auto-aof-rewrite-percentage及auto-aof-rewrite-min-size来使其自动执行。redis会记录启动时或最后一次重写时AOF的文件大小，当当前AOF文件的大小与记录的大小超过一定的比例（auto-aof-rewrite-percentage）时，会自动触发AOF重写。当然有一个最小阈值（auto-aof-rewrite-min-size），即当AOF文件不超过这个大小时，不考虑对其进行重写  

## &emsp;&emsp;slowlog-log-slower-than  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：10000  
&emsp;&emsp;&emsp;&emsp;content：`slowlog-log-slower-than 10000`  
&emsp;&emsp;&emsp;&emsp;命令被慢日志记录的阈值，单位毫秒。redis会统计命令的执行时间，超过指定时间的命令会被记录到慢日志中，用户可以通过slowlog命令查看慢日志的相关信息。需要注意的是redis只会统计命令的实际执行时间，并不会统计IO的时间。0的话会统计所有命令，负数的话不进行统计

## &emsp;&emsp;slowlog-max-len  
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：128  
&emsp;&emsp;&emsp;&emsp;content：`slowlog-max-len 128`  
&emsp;&emsp;&emsp;&emsp;慢日志的记录数量，循环记录。可以通过SLOWLOG RESET命令来清空慢日志，从而节省内存

## &emsp;&emsp;~~vm-enabled~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：no  
&emsp;&emsp;&emsp;&emsp;content：`vm-enabled no`  
&emsp;&emsp;&emsp;&emsp;虚拟内存允许redis使用比实际RAM更大的内存，可以将不常用的数据保存到SWAP分区。***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;~~vm-swap-file~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：/tmp/redis.swap  
&emsp;&emsp;&emsp;&emsp;content：`vm-swap-file /tmp/redis.swap`  
&emsp;&emsp;&emsp;&emsp;redis交换文件的地址，注意不同的redis实例应该使用不同的交换文件，交换文件所在的磁盘最好使用SSD，为了保证安全，最好为redis单独创建一个文件夹，并将文件夹的权限授予redis的启动用户，避免其他进程触及到该文件。***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;~~vm-max-memory~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：0  
&emsp;&emsp;&emsp;&emsp;content：`vm-max-memory 0`  
&emsp;&emsp;&emsp;&emsp;SWAP文件的最大大小 ***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;~~vm-page-size~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：32  
&emsp;&emsp;&emsp;&emsp;content：`vm-page-size 32`  
&emsp;&emsp;&emsp;&emsp;Redis虚拟内存的页大小，单位byte，一个对象可以使用多个连续页存储，但是一个页却不能存储多个对象，因此当页配置过大时，会造成资源浪费。当redis实例以小对象为主时，页面大小建议设置为64或32byte，如果以大对象为主的话，则根据实际调大即可 ***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;~~vm-pages~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：134217728  
&emsp;&emsp;&emsp;&emsp;content：`vm-pages 134217728`  
&emsp;&emsp;&emsp;&emsp;Redis虚拟内存页数，每8页将会在RAM中消耗1byte，swap文件的大小=vm-page-size * vm-pages，如果使用默认的每页32byte及134217728页，将总计占用4GB的swap文件及16MB的RAM，最好根据应用实际情况来配置 ***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;~~vm-max-threads~~  
&emsp;&emsp;&emsp;&emsp;***@Deprecated***   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：4  
&emsp;&emsp;&emsp;&emsp;content：`vm-max-threads 4`  
&emsp;&emsp;&emsp;&emsp;虚拟IO线程的数量，虚拟IO线程用于对SWAP文件的读写，由于虚拟线程同时还负责内存和虚拟内存之间的编解码，因此在物理IO满足的情况下，调大该参数可以获得更好的读写效率 ***需要注意VM特性在2.4版本中已废弃，后续不建议使用***

## &emsp;&emsp;hash-max-zipmap-entries     
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：512  
&emsp;&emsp;&emsp;&emsp;content：`hash-max-zipmap-entries 512`  
&emsp;&emsp;&emsp;&emsp;Redis的Hash对象将会被以特定的方式编码，从而节省内存。hash-max-zipmap-entries指明了当元素数量不超过多少时，使用这种编码结构

## &emsp;&emsp;hash-max-zipmap-value   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：64  
&emsp;&emsp;&emsp;&emsp;content：`hash-max-zipmap-value 64`  
&emsp;&emsp;&emsp;&emsp;Redis的Hash对象将会被以特定的方式编码，从而节省内存。hash-max-zipmap-value指明了当元素大小不超过该值时，使用这种编码结构

## &emsp;&emsp;list-max-ziplist-entries   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：512  
&emsp;&emsp;&emsp;&emsp;content：`list-max-ziplist-entries 512`  
&emsp;&emsp;&emsp;&emsp;Redis的List对象将会被以特定的方式编码，从而节省内存。list-max-ziplist-entries指明了当元素数量不超过该值时，使用这种编码结构

## &emsp;&emsp;list-max-ziplist-value   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：64  
&emsp;&emsp;&emsp;&emsp;content：`list-max-ziplist-value 64`  
&emsp;&emsp;&emsp;&emsp;Redis的List对象将会被以特定的方式编码，从而节省内存。list-max-ziplist-value指明了当元素大小不超过该值时，使用这种编码结构

## &emsp;&emsp;set-max-intset-entries   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：512  
&emsp;&emsp;&emsp;&emsp;content：`set-max-intset-entries 512`  
&emsp;&emsp;&emsp;&emsp;Redis会对64位以内的十进制整数的Set集合采用特殊编码，因此可以通过set-max-intset-entries来指明当集合内的元素数量不超过该值时，采用该编码形式

## &emsp;&emsp;zset-max-ziplist-entries   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：128  
&emsp;&emsp;&emsp;&emsp;content：`zset-max-ziplist-entries 128`  
&emsp;&emsp;&emsp;&emsp;Redis会对ZSet集合采用特殊编码，因此可以通过zset-max-ziplist-entries来指明当集合内的元素数量不超过该值时，采用该编码形式

## &emsp;&emsp;zset-max-ziplist-value   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：64  
&emsp;&emsp;&emsp;&emsp;content：`zset-max-ziplist-value 64`  
&emsp;&emsp;&emsp;&emsp;Redis会对ZSet集合采用特殊编码，因此可以通过zset-max-ziplist-entries来指明当集合内的元素大小不超过该值时，采用该编码形式

## &emsp;&emsp;activerehashing   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：yes  
&emsp;&emsp;&emsp;&emsp;content：`activerehashing yes`  
&emsp;&emsp;&emsp;&emsp;是否激活重hash。重hash策略会占用1/100的cpu时间来对redis主hash表（redis最上层key的映射关系）进行重hash，主hash表采用延迟重hash的策略，对主hash表的访问越多，会推进rehash进程。因此当redis服务空闲时，rehash操作将不会完成，并额外占用一部分内存。激活rehash会造成约额外2毫秒的延迟，如果对延迟特别敏感可以考虑关闭该选项。但是如果对内存占用特别敏感，建议开启这个选项，可以尽快释放已被删除的key占用的物理内存。

## &emsp;&emsp;include   
&emsp;&emsp;&emsp;&emsp;sinece：2.4  
&emsp;&emsp;&emsp;&emsp;default：/path/to/local.conf  
&emsp;&emsp;&emsp;&emsp;content：`include /path/to/local.conf`  
&emsp;&emsp;&emsp;&emsp;可以配置一个或多个其他的配置文件，当前配置文件会和额外配置文件合并，作为最终的配置来加载redis