# rabbitmqPro

备注：b站 https://www.bilibili.com/video/BV1dE411K7MG?p=2
# 不同MQ特点
1、ActiveMQ 老牌的消息中间件 Apache出品 最流行的，能力强劲的开源消息总线，它是一个完全支持JMS规范的消息中间件

2、Kafka 不支持事务 对消息的重复 丢失 错误没有严格要求 适合产生大量数据的互联网服务的数据收集业务

3、RocketMQ 阿里开源的消息中间件 它是纯Java开发，具有高吞吐量 高可用性 适合大规模分布式系统应用的特点

4、RabbitMQ Erlang语言开发 基于AMQP协议来实现 面向消息 队列 路由 包括点对点和 发布/订阅 可靠性 安全 数据一致性 稳定性 可靠性要求很高的场景
  对性能和吞吐量的要求还在其次  对spring框架无缝连接
  
  amqp协议 高级消息队列协议
  
  生产者将消息发给server（rabbitmq节点） 发给exchange（交换机）交换机和队列进行交互  message queue
  消费者和队列进行交互
  
  下载安装 登录网站 https://www.rabbitmq.com/  点击getstarted download + installation 最新版本 3.8.11
  选择centos版本  Downloading ---->  选择centos 8 rabbitmq-server-3.8.11-1.el8.noarch.rpm
  还有相关的依赖 erlang-22.0.7-1.el7.x86_64.rpm  socat-1.7.3.2-2.el7.x86_64.rpm
  
  文件在磁盘的d盘  D:\soft\rabbitmq 三个文件
  上传到centos 的usr/local/soft 目录下 
  执行命令 
  rpm -ivh erlang-22.0.6-1.el6.x86_64.rpm
  rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
  rpm -ivh rabbitmq-server-3.7.17-1.el7.noarch.rpm
  注意提供一个Linux的搜索命令 find / -name rabbitmq.config.example
  注意：安装vim 命令为 yum install -y vim 就可以使用vim命令了
  
  需要将rabbitmq.config.example拷贝至 /etc/rabbitmq下 修改rabbitmq.config
  
  开放来宾用户
  打开{loopback_users,[]}  允许来宾用户使用，否则是没有权限登录使用的
  
  执行命令， 启动rabbitmq中的插件管理
  rabbitmq-plugins enable  rabbitmq_management  能够使用界面进行管理
  
  看rabbitmq启用情况   systemctl status rabbitmq-server
  systemctl start rabbitmq-server systemctl restart rabbitmq-server
  systemctl stop rabbitmq-server
  
  关闭防火墙
  systemctl disable firewalld
  systemctl stop firewalld
  
  启动rabbitmq ip地址+15672端口 就能看到管理页面
  用户名 密码 guest guest
  
  在安装rabbitmq时会遇到 警告和错误提示 错误：依赖检测失败 libnsl.so.1()(64bit) 被 erlang-22.0.6-1.el6.x86_64 需要
  需要安装 rpm -ivh compat-openssl10-1.0.2o-3.el8.x86_64.rpm  以及 dnf install libnsl  因为缺少库文件 文件在目录下D:\soft\rabbitmq
  
  之后就可以安装相关依赖和rabbitmq了
  
  ## centos8安装rabbitmq故障问题记录
  
  查看依赖是否存在
  $ ls /lib/x86_64-linux-gnu/libtinfo.so.*
  
  在执行如下命令,启动rabbitmq中的插件管理	rabbitmq-plugins enable rabbitmq_management时
  报错/usr/lib64/erlang/erts-10.4.4/bin/beam.smp: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
  
  此时按照 https://blog.csdn.net/qq_41571395/article/details/104693261
  根据文章中所说  根据/usr/lib/libncurses.so.5的链接方式， 在/usr/lib64下打个软链接：
  执行命令sudo ln -s /usr/lib64/libtinfo.so.6 /usr/lib64/libtinfo.so.5
  创建一个libtinfo.so.5  再去执行rabbitmq-plugins enable rabbitmq_management 就可以了。
  `当出现` 说明rabbit管理页面可以打开了
  Enabling plugins on node rabbit@kimmy:
  rabbitmq_management
  The following plugins have been configured:
    rabbitmq_management
    rabbitmq_management_agent
    rabbitmq_web_dispatch
  Applying plugin configuration to rabbit@kimmy...
  The following plugins have been enabled:
    rabbitmq_management
    rabbitmq_management_agent
    rabbitmq_web_dispatch
  
  set 3 plugins.
  Offline change; changes will take effect at broker restart.
  
  ## 一台新的机器
  执行 rpm -ivh erlang-22.0.7-1.el7.x86_64.rpm --nodeps --force 
  强制安装 否则会出现 依赖检测失败:libcrypto.so.10()(64bit) 等错误
  还有socat依赖 同样强制安装 rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm --nodeps --force
  否则会出现
  错误：依赖检测失败：
  	libcrypto.so.10()(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libcrypto.so.10(OPENSSL_1.0.1_EC)(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libcrypto.so.10(libcrypto.so.10)(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libreadline.so.6()(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libssl.so.10()(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libssl.so.10(libssl.so.10)(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	libwrap.so.0()(64bit) 被 socat-1.7.3.2-2.el7.x86_64 需要
  	
  	接着执行 yum install -y rabbitmq-server-3.7.18-1.el7.noarch.rpm
  	安装rabbtMQ
  	
  	cp /usr/share/doc/rabbitmq-server-3.7.18/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
  	查看 ls /etc/rabbitmq/rabbitmq.config
  	
  	修改配置信息 执行 vim /etc/rabbitmq/rabbitmq.config 
  	
  	 执行如下命令,启动rabbitmq中的插件管理  rabbitmq-plugins enable rabbitmq_management
  	 
  	 会出现 /usr/lib64/erlang/erts-10.4.4/bin/beam.smp: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
  	 
  	 解决方法 如上面故障问题解决
  	 
  	 启动RabbitMQ的服务
  	 systemctl start rabbitmq-server
     systemctl restart rabbitmq-server
     systemctl stop rabbitmq-server
     
     会出现错误
     Job for rabbitmq-server.service failed because the control process exited with error code.
     See "systemctl status rabbitmq-server.service" and "journalctl -xe" for details.
     和第二台出现的错误一样
     
     出现这个错误的主要原因是没有libcrypto.so.10(OPENSSL_1.0.2)(64bit)依赖
     参考文章：https://blog.csdn.net/Alphr/article/details/107931568?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control
     下载openssl-libs-1.0.2k-19.el7.x86_64.rpm 目录在 D:\360安全浏览器下载\openssl-libs-1.0.2k-19.el7.x86_64.rpm
     
     
     【  装一个 openssl-lib
         #下载
         wget ftp://ftp.pbone.net/mirror/ftp.centos.org/7.6.1810/updates/x86_64/Packages/openssl-libs-1.0.2k-16.el7_6.1.x86_64.rpm
         #安装
         rpm -ivh openssl-libs-1.0.2k-16.el7_6.1.x86_64.rpm --force
      】
     
     然后执行 rpm -ivh openssl-libs-1.0.2k-19.el7.x86_64.rpm --force
     再启动 执行 systemctl start rabbitmq-server
     访问管理界面就可以访问了
     
     
     
     关闭防火墙服务
     systemctl disable firewalld
     Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
     Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
 	systemctl stop firewalld   
 	
 	集群部署
 	修改hosts文件
 	scp /etc/hosts root@kimmy:/etc/  需要输入kimmy的密码
 	
 	:61 找到第61行
 	yum install vim -y 安装vim
 	
 	配置cookie 保持统一
 	查看每台机器的cookie  cat /var/lib/rabbitmq/.erlang.cookie
 	
 	安装scp命令  yum install openssh-clients
 	复制第二台第三台cookie的命令  
 	scp /var/lib/rabbitmq/.erlang.cookie root@kimmy:/var/lib/rabbitmq/
 	scp /var/lib/rabbitmq/.erlang.cookie root@jimmy:/var/lib/rabbitmq/
 	
 	后台启动  rabbitmq-server -detached
 	警告Warning: PID file not written; -detached was passed. 不用关心
 	
 	rabbitmqctl cluster_status
 	查看集群的状态
 	
 	开始部署集群一个为主节点 其他两个为从节点
 	在第二台机器 第三台机器执行 rabbitmqctl stop_app
 	接着加入集群    rabbitmqctl join_cluster rabbit@timmy
 	然后启动服务    rabbitmqctl start_app
 	
 	使用 rabbitmqctl cluster_status 查看集群状态
 	
 	`镜像集群`
 	
 	添加策略
    	rabbitmqctl set_policy ha-all '^hello' '{"ha-mode":"all","ha-sync-mode":"automatic"}' 
    	说明:策略正则表达式为 “^” 表示所有匹配所有队列名称  ^hello:匹配hello开头队列
    如果某一时刻主节点宕机了，其他节点依然可以进行消费；
    当之前的主节点又恢复了，此时它只能作为子节点了
    
    删除策略
    	rabbitmqctl clear_policy ha-all
    	
    	20200128今天首次启动rabbitmq集群出现的问题是无法进行集群的启动
    	出现了缺少了 socat: error while loading shared libraries: libwrap.so.0: cannot open share
    	参考配置了 https://www.cnblogs.com/bclshuai/p/7446619.html 但不知道这个有用不
    	修改/etc/profile在文件末尾加上两行： LD_LIBRARY_PATH=./ 和 export LD_LIBRARY_PATH
    	source  /etc/profile
    	
    	最后发现也不行，上面的配置也没有去掉
    	
    	发现可能是socat这个安装有问题 参考文章：https://blog.csdn.net/u013098162/article/details/106842095/
    	直接执行命令 yum install socat 进行安装
    	安装完之后
    	再执行启动rabbitmq 还是启动不了 参考文章：https://www.cnblogs.com/straycats/p/7719933.html
    	解决方案：
        
        /var/lib/rabbitmq/mnesia 目录下存在rabbit@localhost.pid、rabbit@localhost、rabbit@localhost-plugins-expand，
        删除这3项后，再使用systemctl start rabbitmq-server启动，发现不报错了。
        
        再重新启动就可以了
        
        但是在启动镜像集群时要注意
        
        可以先启动三台rabbitmq，
        第一台先关闭 再执行  rabbitmq-server -detached
        第二台 第三台 使用systemctl start rabbitmq-server  接着执行 rabbitmqctl stop_app 关闭
        第二台 第三台 接着加入集群    rabbitmqctl join_cluster rabbit@timmy
                 	然后启动服务    rabbitmqctl start_app
        之后看到管理页面每一台机器上都有三个节点了。
        执行 rabbitmqctl cluster_status 查看集群状态
        
         Cluster status of node rabbit@jimmy ...
         [{nodes,[{disc,[rabbit@jimmy,rabbit@kimmy,rabbit@timmy]}]},
         {running_nodes,[rabbit@kimmy,rabbit@timmy,rabbit@jimmy]},
         {cluster_name,<<"rabbit@timmy">>},
         {partitions,[]},
         {alarms,[{rabbit@kimmy,[]},{rabbit@timmy,[]},{rabbit@jimmy,[]}]}]
         
         配置好镜像集群后，项目就能正常启动了
         
         镜像集群需要设置同步策略 就可以实现高可用 ，如果一台机器挂了，其他机器也能使用；如果不设置同步策略，主节点机器挂了，其他子节点是不能使用，主节点正常情况下，子节点可以分担压力
         
         要指定使用哪台虚拟机 -p '主机名' 或者指定队列名，对特定队列进行同步 '^'对全部队列进行
         rabbitmqctl set_policy -p '/ems' ha-all '^QUEUE_MYSELF_SEND_MESSAGE_DEV' '{"ha-mode":"all","ha-sync-mode":"automatic"}'

    	
    	
    	
 	
  

  