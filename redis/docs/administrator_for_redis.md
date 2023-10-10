本章节为redis管理员须知
# Redis安装建议

## &emsp; 使用Linux环境运行Redis

&emsp;&emsp;Redis主要是基于Linux系统进行开发发布，虽然也有os X、FreeBSD和OpenBSD的测试版本，但是在Linux环境上是经过最严格测试的，并且大多数的生产环境也是在Linux上运行。  
&emsp;&emsp;将Linux内核的超额分配内存设置为  
&emsp;&emsp;1：vm.overcommit_memory = 1（在/etc/sysctl.conf）文件中，设置完成后重启计算机使设置生效。原因见 https://redis.io/docs/getting-started/faq/#background-saving-fails-with-a-fork-error-on-linux  
&emsp;&emsp;简单说就是用来解决内存不足导致的fork失败，linux的mmu机制会在程序申请的内存总数超过物理内存时报OOM异常，但是实际上很多程序只是申请了大块的虚拟内存，并没有真正投入使用（因此并没有实际分配物理内存），这时可以将vm.overcommit_memory设置为1来允许超额申请内存，对于redis而言，它的后台备份机制fork的子进程只是为了和父进程共享内存，并不会额外申请内存空间，但是在操作系统看来会需要跟父进程一样的内存空间，如果设置为2，就会报错。  
&emsp;&emsp;&emsp;&emsp;0 ——默认值，启发式overcommit，它允许overcommit，但太明显的overcommit会被拒绝，比如malloc一次性申请的内存大小就超过了系统总内存。  
&emsp;&emsp;&emsp;&emsp;1 ——Always overcommit. 允许overcommit，对内存申请来者不拒。  
&emsp;&emsp;&emsp;&emsp;2 ——不允许overcommit，提交给系统的总地址空间大小不允许超过。  

## &emsp;内存
&emsp;&emsp;确保开启了swap分区，并且swap分区文件的大小至少等于系统物理内存大小；如果不开启swap分区，那么当redis实例突然消耗了大量内存的话，redis可能会OOM导致redis崩溃，或者被linux的OOM killer杀掉进程，开启swap分区可以一定程度上避免这类问题，并且可以通过观测redis的操作延迟来判断是否发生了内存交换，进而去处理这类内存不足的事件。  
&emsp;&emsp;明确的设置maxmemory属性（在配置文件或使用config set设置），这样当redis使用的内存超过限制的时候，会明确的报内存错误，而不是笼统的说操作失败；需要注意的是估算最大内存时应该站在redis实例的角度上估算，即需要考虑数据内存和实例消耗的碎片内存，不能仅仅考虑要保存的数据的占用内存大小，并且还需要为操作系统留出一部分内存，因此如果有10GB的可用内存，那么将其设置到8~9GB是比较恰当的。  
&emsp;&emsp;对于写操作较多的系统，在redis保存RDB文件或者重写AOF的过程中，redis使用的内存最多可能是正常情况的两倍，额外使用的内存多少，取决于在这个过程中涉及到写的key所在的内存页所占的内存，或者简单的可以理解成所涉及到写的key的比例多少，因此要确保在使用redis的过程中考虑到了这种情况  
&emsp;&emsp;redis 出现问题时，可以参考LATENCY DOCTOR和MEMORY DOCTOR命令来查看问题建议  

## &emsp;镜像化
&emsp;&emsp;在不支持后台运行的环境下运行redis时（如docker镜像启动），此时注意将redis配置文件中的daemonize 设置为no  
&emsp;&emsp;daemonize决定redis是否以守护进程运行，如果设置no，则进入前台模式，此时会运行redis实例并进入控制台界面，关闭控制台会导致程序退出；反之如果设置为yes则会以后台模式运行，并将进程pid写入到配置文件中指定的pid文件中，此时只有系统关闭或手动kill，shutdown命令才会导致进程退出。  

## &emsp;副本
&emsp;&emsp;对master设置一个合理的副本积压，可以使副本同步更加容易。
slave与主节点同步时，会优先使用增量同步，增量同步数据就是保存在master的relicate backlog中，repl_backlog由两个参数控制  
&emsp;&emsp;&emsp;&emsp;repl-backlog-size：环形缓冲复制队列的大小，默认单位是byte，支持kb，mb，gb  
&emsp;&emsp;&emsp;&emsp;repl-backlog-ttl：环形缓冲复制队列的存活时长（当所有的slave都不可用时，超过这个时间会导致环形缓冲队列被销毁）
slave增量同步时，会使用psync发送自身处理的数据偏移量，如果偏移量还在replicate backlog的范围内，master只将增量数据发送给slave，否则就会转为全量同步  
&emsp;&emsp;如果开启副本，那么在没有开启无盘复制的前提下，即使没有配置持久化选项，也会在主从同步时自动生成RDB文件，如果机器没有硬盘，那么一定要确保开启无盘复制。（diskless replication）  
&emsp;&emsp;如果开启副本，那么一定要确保master的RDB文件可以得到妥善的持久化保存，或者当master崩溃时不会自动重启；因为redis的副本或保持跟master的完全同步，因此如果master重启后丢失了所有数据，副本也会将自己的数据全部擦除。  

## &emsp;安全性
&emsp;&emsp;redis默认不需要授权并且会绑定到所有网卡上，这存在极大的安全隐患，因此建议根据安全配置指引（https://redis.io/docs/management/security/）进行设置 Redis安全  

## &emsp;在EC2等云实例上运行redis
&emsp;&emsp;使用全虚拟化（HVM）实例，而非半虚拟化（PV）实例  
&emsp;&emsp;&emsp;&emsp;不要使用旧实例，例如应该使用基于HVM的m3.medium，而不要使用基于PV的m1.medium  
&emsp;&emsp;使用云盘作为redis实例的持久化存储设备时，要注意云盘可能有很高的延迟  
&emsp;&emsp;当主从同步出现问题时，考虑使用无盘复制  

## &emsp;在不停机的情况下升级或重启redis实例
&emsp;&emsp;redis作为一个长期运行服务，可以通过CONFIG SET命令在运行时修改配置，这样就可以不用重启redis；例如可以通过这种方式来切换redis持久化方式（AOF、RDB）；具体可以通过CONFIG GET查看其支持的配置项  
&emsp;&emsp;对于一些不支持通过CONFIG SET设置的属性，或者需要更新redis的版本时，依然需要重启redis进程  
&emsp;&emsp;以下措施可以尽可能的避免重启redis：  
&emsp;&emsp;&emsp;&emsp;1. 设置一个redis实例作为当前redis的副本，保证两个redis实例可以同时运行（在不同机器上，或机器内存足够）  
&emsp;&emsp;&emsp;&emsp;2. 检查副本实例日志，等待副本同步完成  
&emsp;&emsp;&emsp;&emsp;使用INFO命令查看实例信息，确保master和slave具有相同的key数量，并且可以正常响应命令  
&emsp;&emsp;&emsp;&emsp;3. 通过CONFIG SET slave-read-only no来允许副本读写  
&emsp;&emsp;&emsp;&emsp;4. 让所有的客户端都去访问副本节点，可以通过CLIENT PAUSE命令来确保没有客户端的写请求到master节点  
&emsp;&emsp;&emsp;&emsp;5. 当确保master没有任何命令之后，选举一个副本节点作为新MASTER，并使用REPLICATEOF NO ONE命令来将其提主，然后调用shutdown 命令来关停master  
&emsp;&emsp;&emsp;&emsp;6. 如果使用Redis Sentinel或Redis cluster，最简单的方式来升级redis就是挨个升级副本节点，然后通过执行failover命令来将某个已升级的节点进行提主；最后将旧master进行升级即可  