# Redis配置项
本章介绍关于redis配置文件的一些概述  
## 配置文件
redis可以不指定配置文件启动，其内部内嵌了默认参数，但是这种方式只建议在测试或开发场景下使用。  
恰当的方式是提供一个单独的 ***redis.conf*** 文件，来记录redis的配置项。名字一般叫redis.conf，当然也可以使用其他名字  
redis.conf由一系列格式简单的配置项组成：  
`keyword argument1 argument2 ... argumentN`  
当参数内部有空格时，需要将参数通过单引号或者双引号括起来。  
单引号支持由反斜杠开头的转义字符，双引号则除了转义字符外，还支持使用\x十六进制编码的任意字符。  
redis的配置文件及配置项说明可以在各自的发行版中的redis.conf文件中找到
通过命令行参数传递配置项  
除了配置文件外，也可以在redis启动时，通过命令行参数来指定配置项。命令行参数支持的种类和格式与配置文件中完全一致  
`./redis-server --port 6380 --replicaof 127.0.0.1 6379`  
***需要注意的是通过命令行传递的配置项都在内存中的临时配置文件中，非持久化***
## 运行过程中改变redis配置  
服务运行中可以通过CONFIG GET/SET命令来获取/设置配置文件，不用对服务进行重启  
需要注意的是不是所有的参数都支持热更新，具体可以查看CONFIG GET/SET的命令详情  
通过这种方式所做的改变都是非持久化的，不会影响redis.conf配置文件，下次启动时依然以redis.conf中的配置项为准  
***可以通过CONFIG REWRITE命令，来更新redis.conf文件***。 注释不会改动，值为默认值的配置项不会新增，修改的配置项会自动修改，新增的会自动追加
## 将redis配置为缓存服务器  
如果将redis作为缓存服务器，可以通过设置maxmemory来指定最大缓存大小，通过指定maxmemory-policy allkeys-lru来指定内存淘汰策略  
这样一来应用程序就不需要再通过EXPIRE指定过期时间，当缓存空间不够时，redis会对所有的key执行lru算法  