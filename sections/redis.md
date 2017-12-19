# redis安装及相关配置参考文档
* [`[Basic]`redis安装](/sections/redis.md#redis安装)
* [`[Basic]`redis基础配置](/sections/redis.md#redis基础配置)
* [`[Basic]`redis主从哨兵配置](/sections/redis.md#redis主从哨兵配置)

## redis安装
&emsp;1.下载安装包   
&emsp;```$ wget http://download.redis.io/releases/redis-4.0.2.tar.gz```  

&emsp;2.解压压缩包到/user/local/文件夹(可以是你喜欢的某个路径)   
&emsp;```$ tar xzf redis-4.0.2.tar.gz -C /usr/local/```    

&emsp;3.进入解压目录   
&emsp;```$ cd /usr/local/redis-4.0.2```   

&emsp;4.我们需要从源码编译安装，所以需要编译安装环境 build-essential 。为了测试编译产生的二进制文件，同时需要安装 tcl 包。   
&emsp;```$ apt-get update```   
&emsp;```$ apt-get install build-essential tcl```   

&emsp;5.编译安装   
&emsp;```$ make```   
&emsp;```$ make install```  

&emsp;6.编译二进制文件之后，为确保编译步骤都执行成功，我们可以运行测试用例，运行测试用户需要等待几分钟(这一步是可选的，非必做步骤)   
&emsp;```$ make test```   

&emsp;7.编译完成之后，在 /src 目录下会出现六个可执行文件
* redis-server  //is the Redis Server itself.
* redis-sentinel  //is the Redis Sentinel executable (monitoring and failover).
* redis-cli  //is the command line interface utility to talk with Redis.
* redis-benchmark //is used to check Redis performances.
* redis-check-aof and redis-check-dump // which are useful in the rare event of corrupted data files.   

&emsp;8.启动redis   
&emsp;```$ src/redis-server```或```$ src/redis-server redis.conf```,后者为制定指定配置文件启动   

## redis基础配置
&emsp;1.配置文件参数说明
&emsp;根据以上默认安装配置中参数进行解释说明
```
###################################################### 引用 ##############################################################
# include /path/to/local.conf
# include /path/to/other.conf
//不同redis server可以使用同一个模版配置作为主配置，并引用其它配置文件用于本server的个性化设置.include引入的文件不会被管理员或Redis Sentinel执行“CONFIG REWRITE”命令重写。如果有出现相同的配置选项，redis总是使用最后一次配置替换之前的。如果include进来的文件是共享配置，那么最好放在开头的位置；反之，就要放在文件的末尾。

################################################# 模块 ##################################################################
# loadmodule /usr/lib/xx.so
# loadmodule /usr/lib/yy.so
//这个配置可在redis启动的时候加载指定的模块，如果模块加载失败redis将不能启动。可以加载多个模块。

################################################# 网络 ##################################################################
bind 127.0.0.1
//指定redis只接收来自于该IP地址的请求，如果不进行设置，那么将处理所有请求,设置成bind 0.0.0.0也会接收来自所有ip的请求，通常如果不设置成127.0.0.1会使用密码 requirepass 选项

requirepass 123456
//设置客户端连接后进行任何其他指定前需要使用的密码，这里设置成123456.警告：因为redis速度相当快，所以在一台比较好的服务器下，一个外部的用户可以在一秒钟进行150K次的密码尝试，这意味着你需要指定非常非常强大的密码来防止暴力破解

protected-mode yes
//是否开启保护模式。默认开启，如果没有设置bind项的ip和redis密码的话，服务将只允许本地访问。

port 6379
//端口设置，默认为 6379，如果port设置为0 redis将不会监听tcp socket

tcp-backlog 511
//在高并发环境下需要一个高backlog值来避免慢客户端连接问题。注意Linux内核默默将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog 两个值来达到需要的效果。

timeout 0
//客户端空闲多少秒后关闭连接（0为不关闭）

tcp-keepalive 300
//设置。如果非零，则设置SO_KEEPALIVE选项来向空闲连接的客户端发送ACK，用途如下：1）能够检测无响应的对端；2）让该连接中间的网络设备知道这个连接还存活。 在Linux上，这个指定的值(单位秒)就是发送ACK的时间间隔，注意：要关闭这个连接需要两倍的这个时间值。在其他内核上这个时间间隔由内核配置决定# 从redis3.2.1开始默认值为300秒

################################################ 通用 #######################################################################
daemonize no
//是否将Redis作为守护进程运行。如果需要的话配置成'yes'，注意配置成守护进程后Redis会将进程号写入文件/var/run/redis.pid,除非有pidfile 指定了进程号写入文件，当设置pidfile时进程号将被写入pidfile指定的文件。默认no，启动时执行ctrl + c 或关闭对应的命令行窗口会中断关闭redis进程。

supervised no
//是否通过upstart或systemd管理守护进程。默认no没有服务监控，其它选项有upstart, systemd, auto

pidfile /var/run/redis_6379.pid
//pid文件在redis启动时创建，退出时删除。最佳实践为配置该项。

loglevel notice
//配置日志级别。选项有debug, verbose, notice, warning

logfile ""
//日志名称。空字符串表示标准输出。注意如果redis配置为后台进程(即daemonize yes)，标准输出中信息会发送到/dev/null。建议是设置一个路径，方便自己管理，如logfile "/var/log/redis_6379.log"

# syslog-enabled no
//是否启动系统日志记录。

# syslog-ident redis
//指定系统日志身份。

# syslog-facility local0
//指定syslog设备。必须是user或LOCAL0 ~ LOCAL7之一。

always-show-logo yes
//设置是否显示redis logo图标，在4.0之前要么在控制台，要么在日志中始终会显示这个图标，但从4.0开始我们可以选择是否显示这个图标

databases 16
//设置数据库个数。默认数据库是 DB 0.可以通过SELECT where dbid is a number between 0 and 'databases'-1为每个连接使用不同的数据库。

########################################################## 备份  ###########################################################
//下面的例子将会进行把数据写入磁盘的操作:配置redis在何时生成快照，sava有两个参数  1. 第一个参数为时间，单位为秒。 2. 第二个参数为keys的变更数，单位为个，例如：save 300 20，如果300秒有20个key发生了变更则生成快照. 禁用自动生成快照，只要注释调所有的save就行了，或者把save设置为`""`同样能达到禁止生成快照的效果
save 900 1  //900秒（15分钟）之后，且至少1次变更
save 300 10  //300秒（5分钟）之后，且至少10次变更
save 60 10000  //60秒之后，且至少10000次变更

stop-writes-on-bgsave-error yes
//生成快照失败是否停止写. 默认情下如果在启用了save且生成快照时发生错误，redis将停止写操作.如果后台保存进程重新启动工作了，redis 也将自动的允许写操作。如果有其它监控方式也可关闭。

rdbcompression yes
//  是否启用压缩数据库来节省空间. 是否在备份.rdb文件时是否用LZF压缩字符串，默认设置为yes。如果想节约cpu资源可以把它设置为no。

rdbchecksum yes
// 是否对数据进行完整性校验。

dbfilename dump.rdb
//数据库文件名

dir ./
//备份文件目录，文件名就是上面的 "dbfilename" 的值。累加文件也放这里。注意你这里指定的必须是目录，不是文件名。


########################################################## 主从同步 #######################################################
// 1) redis主从同步是异步的，但是可以配置在没有指定slave连接的情况下使master停止写入数据。
// 2) 连接中断一定时间内，slave可以执行部分数据重新同步。
// 3) 同步是自动的，slave可以自动重连且同步数据。
# slaveof <masterip> <masterport> //设置当前配置为masterip masterport的从库，例如slaveof 192.168.5.2 6379

# masterauth <master-password>
//master连接密码,主从自动切换的须后需要保持主库配置文件和从库配置文件的master-password一致。 距离： masterauth "123456789"

slave-serve-stale-data yes
//1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。
//2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。

slave-read-only yes
//你可以配置salve实例是否接受写操作。可写的slave实例可能对存储临时数据比较有用(因为写入salve# 的数据在同master同步之后将很容被删除)，但是如果客户端由于配置错误在写入时也可能产生一些问题。
//从Redis2.6默认所有的slave为只读
//注意:只读的slave不是为了暴露给互联网上不可信的客户端而设计的。它只是一个防止实例误用的保护层。

repl-diskless-sync no  //默认无盘同步
//redis有盘同步： redis同步时通常先将同步的数据生成到物理磁盘上的rdb文件中，然后在读取rdb文件发送到其它同步机。
//redis无盘同步： 省略向磁盘生成rdb的过程

repl-diskless-sync-delay 5
//如果非磁盘同步方式开启，可以配置同步延迟时间，以等待master产生子进程通过socket传输RDB数据给slave。默认值为5秒，设置为0秒则每次传输无延迟。

# repl-ping-slave-period 10
//slave根据指定的时间间隔向master发送ping请求。默认10秒。

# repl-timeout 60
//同步的超时时间
//1）slave在与master SYNC期间有大量数据传输，造成超时
//2）在slave角度，master超时，包括数据、ping等
//在master角度，slave超时，当master发送REPLCONF ACK pings 确保这个值大于指定的repl-ping-slave-period，否则在主从间流量不高时每次都会检测到超时

repl-disable-tcp-nodelay no
//是否在slave套接字发送SYNC之后禁用 TCP_NODELAY,启用后，redis将在指定时间内合并小的tcp包来减少请求次数从而减少带宽，同样，这也造成此段时间内主从数据的不一致。

# repl-backlog-size 1mb
//主从失去连接时，此缓冲区越大，失去连接的时间就可以越长。

# repl-backlog-ttl 3600
//当master在一段时间内不再与任何slave连接，backlog将会释放。以下选项配置了从最后一个slave断开开始计时多少秒后，backlog缓冲将会释放。0表示永不释放backlog

slave-priority 100
//slave的优先级是一个整数展示在Redis的Info输出中。如果master不再正常工作了，sentinel将用它来选择一个slave提升为master。优先级数字小的salve会优先考虑提升为master，所以例如有三个slave优先级分别为10，100，25，sentinel将挑选优先级最小数字为10的slave。0作为一个特殊的优先级，标识这个slave不能作为master，所以一个优先级为0的slave永远不会被sentinel挑选提升为master。默认优先级为100

# min-slaves-to-write 3
# min-slaves-max-lag 10
//设置当一个master端的可用slave少于N个，延迟时间大于M秒时，不接收写操作,如上 slave少于3个且延迟时间大于10秒时，不接受写操作。两者之一设置为0将禁用这个功能。默认 min-slaves-to-write 值是0（该功能禁用）并且 min-slaves-max-lag 值是10。

# slave-announce-ip 5.5.5.5
# slave-announce-port 1234
//slave可选配置，此配置的目的在于向master申明自己的ip和端口

########################################################## 安全 ############################################################

# requirepass foobared
//要求客户端在处理任何命令时都要验证身份和密码，默认没有密码。由于redis太快，外部用户每秒可以尝试150K次密码，因此你需要一个很难破解的密码

# rename-command CONFIG "" //禁用CONFIG这个命令
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
//它用来改变共享环境中危险命令的名字，在这个例子中 CONFIG 命令被重命名为一个难以猜解的名字。这会对内部用户的工具有效，但是对一般的客户端无效。请注意：改变命令名字被记录到AOF文件或被传送到从服务器可能产生问题。

######################################################### 限制 ###########################################################

# maxclients 10000
//设置最多同时连接的客户端数量。默认这个限制是10000个客户端，然而如果Redis服务器不能配置处理文件的限制数来满足指定的值，那么最大的客户端连接数就被设置成当前文件限制数减32（因为Redis服务器保留了一些文件描述符作为内部使用）。一旦达到这个限制，Redis会关闭所有新连接并发送错误'max number of clients reached'

# maxmemory <bytes>
//不要使用比设置的上限更多的内存。一旦内存使用达到上限，Redis会根据选定的回收策略（参见：maxmemmory-policy）删除key。如果因为删除策略Redis无法删除key，或者策略设置为 "noeviction"，Redis会回复需要更多内存的错误信息给命令。例如，SET,LPUSH等等，但是会继续响应像Get这样的只读命令。在使用Redis作为LRU缓存，或者为实例设置了硬性内存限制的时候（使用 "noeviction" 策略）的时候，这个选项通常事很有用的。
//警告：当有多个slave连上达到内存上限时，master为同步slave的输出缓冲区所需内存不计算在使用内存中。这样当移除key时，就不会因网络问题 / 重新同步事件触发移除key的循环，反过来slaves的输出缓冲区充满了key被移除的DEL命令，这将触发删除更多的key，直到这个数据库完全被清空为止。总之，如果你需要附加多个slave，建议你设置一个稍小maxmemory限制，这样系统就会有空闲的内存作为slave的输出缓存区(但是如果最大内存策略设置为"noeviction"的话就没必要了)

# maxmemory-policy noeviction
//最大内存策略：如果达到内存限制了，Redis如何选择删除key。
//volatile-lru -> 根据LRU算法删除设置过期时间的key
//allkeys-lru -> 根据LRU算法删除任何key
//volatile-random -> 随机移除设置过过期时间的key
//allkeys-random -> 随机移除任何key
//volatile-ttl -> 移除即将过期的key(minor TTL)
//noeviction -> 不移除任何key，只返回一个写错误
//注意：对所有策略来说，如果Redis找不到合适的可以删除的key都会在写操作时返回一个错误。默认策略:noeviction

# maxmemory-samples 5
//LRU和最小TTL算法的实现都不是很精确，但是很接近（为了省内存），所以你可以用样本量做检测。 例如：默认Redis会检查3个key然后取最旧的那个，你可以通过下面的配置指令来设置样本的个数。默认值为5，数字越大结果越精确但是会消耗更多CPU。

######################################################### 延迟释放 #####################################################
//redis 4.0新特性，从根本上解决Big Key(主要指定元素较多集合类型Key)删除的风险。
//lazy free可译为惰性删除或延迟释放；当删除键的时候,redis提供异步延时释放key内存的功能，把key释放操作放在bio(Background I/O)单独的子线程处理中，减少删除big key对redis主线程的阻塞。有效地避免删除big key带来的性能和可用性问题。

//Redis是single-thread程序(除少量的bio任务),当运行一个耗时较大的请求时，会导致所有请求排队等待redis不能响应其他请求，引起性能问题,甚至集群发生故障切换。
//而redis删除大的集合键时，就属于这类比较耗时的请求。通过测试来看，删除一个100万个元素的集合键，耗时约1000ms左右。在redis4.0前，没有lazy free功能；DBA只能通过取巧的方法，类似scan big key,每次删除100个元素；但在面对“被动”删除键的场景，这种取巧的删除就无能为力。
例如：我们生产Redis Cluster大集群，业务缓慢地写入一个带有TTL的2000多万个字段的Hash键，当这个键过期时，redis开始被动清理它时，导致redis被阻塞20多秒，当前分片主节点因20多秒不能处理请求，并发生主库故障切换。redis4.0有lazy free功能后，这类主动或被动的删除big key时，和一个O(1)指令的耗时一样,亚毫秒级返回； 把真正释放redis元素耗时动作交由bio后台任务执行。

//lazy free的使用分为2类：第一类是与DEL命令对应的主动删除，第二类是过期key删除、maxmemory key驱逐淘汰删除。

//1.主动删除类： UNLINK命令是与DEL一样删除key功能的lazy free实现。唯一不同时，UNLINK在删除集合类键时，如果集合键的元素个数大于64个(详细后文），会把真正的内存释放操作，给单独的bio来操作。示例：使用UNLINK命令删除一个大键mylist, 它包含200万个元素，但用时只有0.03毫秒。注意：DEL命令，还是并发阻塞的删除操作

//2.主动删除类： 通过对FLUSHALL/FLUSHDB添加ASYNC异步清理选项，redis在清理整个实例或DB时，操作都是异步的。

//3.被动删除键使用lazy free
lazyfree-lazy-eviction no //针对redis内存使用达到maxmeory，并设置有淘汰策略时；在被动淘汰键时，是否采用lazy free机制；
lazyfree-lazy-expire no //针对设置有TTL的键，达到过期后，被redis清理删除时是否采用lazy free机制；此场景建议开启，因TTL本身是自适应调整的速度。
lazyfree-lazy-server-del no //针对有些指令在处理已存在的键时，会带有一个隐式的DEL键的操作。如rename命令，当目标键已存在,redis会先删除目标键，如果这些目标键是一个big key,那就会引入阻塞删除的性能问题。 此参数设置就是解决这类问题，建议可开启。
slave-lazy-flush no //针对slave进行全量数据同步，slave在加载master的RDB文件前，会运行flushall来清理自己的数据场景，参数设置决定是否采用异常flush机制。如果内存变动不大，建议可开启。可减少全量同步耗时，从而减少主库因输出缓冲区爆涨引起的内存使用增长。

//注意：从测试来看lazy free回收内存效率还是比较高的； 但在生产环境请结合实际情况，开启被动删除的lazy free 观察redis内存使用情况。


http://www.jianshu.com/p/41f393f594e8
http://www.cnblogs.com/guodf/p/6585657.html


```
## redis主从哨兵配置
