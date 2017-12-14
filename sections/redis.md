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

http://www.cnblogs.com/guodf/p/6585657.html
```
## redis主从哨兵配置
