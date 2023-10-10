https://redis.io/commands/client-pause/<br>

```语法：CLIENT PAUSE timeout_milliseconds [WRITE | ALL]
Available since: 3.0.0
Time complexity: O(1)
ACL categories:@admin, @slow, @dangerous, @connection

CLIENT PAUSE命令可以在指定的时间内阻塞所有的客户端请求，该命令会产生以下几个效果：
该命令会停止处理所有符合指定模式（WRITE｜ALL）的命令，但是客户端与副本节点的交互不受影响，只有客户端尝试执行受限制的命令时才会挂起，因此不活跃的客户端不受影响。
CLIENT PAUSE自身不会暂停，它会尽快返回ok
当指定的超时时间到达后，所有阻塞的客户端会解除阻塞，并将暂停期间缓存的命令发送到服务端
CLINET PAUSE命令支持两种模式
ALL：默认模式，会阻塞所有的命令
WRITE：只阻塞WRITE命令，在write模式下，一些命令可能有特殊的行为
EVAL/ EVAL SHA：将会阻塞客户端
PUBLISH/ PFCOUNT：将会阻塞客户端
WAIT：会阻塞客户端
这个命令提供了一种可控的方式来切换clinet对不同redis实例的访问；例如当redis升级时，系统管理员可以做如下操作：
使用CLINET PAUSE命令来暂停客户端的访问
等待数秒确保副本数据与master数据已完全一致
将某个副本进行提主
让客户端访问新的master节点
从redis6.2开始推荐使用WRITE模式，这种模式下可以使用CLIENT UNPAUSE解除阻塞，会停止所有的副本流量，并且允许重新配置旧master而不同担心failover后重新接受到写请求；这同样也是cluster failover期间使用的模式
在redis6.2之前，可以在事务（MULTI/EXEC）里面来使用CLIENT PAUSE命令，这样可以搭配INFO replication命令来获取当前master的offset。这样就可以通过检测副本节点的offset来明确副本之间的同步已完成。
从redis3.2.10/ 4.0.0开始，这个命令同样会阻止key过期；这样保证数据集在阻塞期间是静止的，因此CLIENT PAUSE操作不仅仅禁止来自客户端的写操作，同样禁止内部的写操作
