# redis安装及相关配置参考文档
* [`[Basic]`redis安装](/sections/redis.md#redis安装)
* [`[Basic]`redis基础配置](/sections/redis.md#redis基础配置)
* [`[Basic]`redis主从哨兵配置](/sections/redis.ms#redis主从哨兵配置)

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

## redis主从哨兵配置
