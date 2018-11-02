

日志格式：
```
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} [myid:%X{myid}] - %-5p [%t:%C{1}@%L] - %m%n
```

通过`zkCli.sh`连接到ZK服务器的终端日志：
```
[dannong@xxx-xxx ~]$ /usr/local/zookeeper/bin/zkCli.sh -server localhost:2181
# 连接到ZK服务器
Connecting to localhost:2181
# main线程
# 客户端的环境变量(Environment@)
## ZK版本
2018-11-03 00:24:08,219 [myid:] - INFO  [main:Environment@109] - Client environment:zookeeper.version=3.5.3-beta-8ce24f9e675cbefffb8f21a47e06b42864475a60, built on 04/03/2017 16:19 GMT
## 主机名
2018-11-03 00:24:08,227 [myid:] - INFO  [main:Environment@109] - Client environment:host.name=localhost
## Java信息(版本，主目录，类路径，库路径)
2018-11-03 00:24:08,227 [myid:] - INFO  [main:Environment@109] - Client environment:java.version=1.8.0_112
2018-11-03 00:24:08,229 [myid:] - INFO  [main:Environment@109] - Client environment:java.vendor=Oracle Corporation
2018-11-03 00:24:08,229 [myid:] - INFO  [main:Environment@109] - Client environment:java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home/jre
2018-11-03 00:24:08,229 [myid:] - INFO  [main:Environment@109] - Client environment:java.class.path=/usr/local/zookeeper/bin/../build/classes:/usr/local/zookeeper/bin/../build/lib/*.jar:/usr/local/zookeeper/bin/../lib/slf4j-log4j12-1.7.5.jar:/usr/local/zookeeper/bin/../lib/slf4j-api-1.7.5.jar:/usr/local/zookeeper/bin/../lib/netty-3.10.5.Final.jar:/usr/local/zookeeper/bin/../lib/log4j-1.2.17.jar:/usr/local/zookeeper/bin/../lib/jline-2.11.jar:/usr/local/zookeeper/bin/../lib/jetty-util-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/jetty-servlet-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/jetty-server-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/jetty-security-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/jetty-io-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/jetty-http-9.2.18.v20160721.jar:/usr/local/zookeeper/bin/../lib/javax.servlet-api-3.1.0.jar:/usr/local/zookeeper/bin/../lib/jackson-mapper-asl-1.9.11.jar:/usr/local/zookeeper/bin/../lib/jackson-core-asl-1.9.11.jar:/usr/local/zookeeper/bin/../lib/commons-cli-1.2.jar:/usr/local/zookeeper/bin/../zookeeper-3.5.3-beta.jar:/usr/local/zookeeper/bin/../src/java/lib/*.jar:/usr/local/zookeeper/bin/../conf:
2018-11-03 00:24:08,229 [myid:] - INFO  [main:Environment@109] - Client environment:java.library.path=/Users/dannong/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:java.io.tmpdir=/var/folders/0y/kzmh046n3bqb07pwdrq_zy_r0000gn/T/
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:java.compiler=<NA>
## 操作系统信息
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:os.name=Mac OS X
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:os.arch=x86_64
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:os.version=10.12
## 用户信息
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:user.name=dannong
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:user.home=/Users/dannong
2018-11-03 00:24:08,230 [myid:] - INFO  [main:Environment@109] - Client environment:user.dir=/Users/dannong
## 操作系统内存信息
2018-11-03 00:24:08,231 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.free=117MB
2018-11-03 00:24:08,232 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.max=228MB
2018-11-03 00:24:08,232 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.total=123MB
# 启动客户端连接(ZooKeeper@)，包含连接字符串(connectString)、会话超时(sessionTimeout)、事件监视者(ZooKeeperMain$MyWatcher@)
2018-11-03 00:24:08,237 [myid:] - INFO  [main:ZooKeeper@865] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@4cdf35a9
# 客户端套接字(ClientCnxnSocket@)，调整最大缓冲区(jute.maxbuffer)
2018-11-03 00:24:08,262 [myid:] - INFO  [main:ClientCnxnSocket@236] - jute.maxbuffer value is 4194304 Bytes
Welcome to ZooKeeper!
# main-SendThread(localhost:2181)，数据发送线程
# 打开与服务器的套接字连接(ClientCnxn$SendThread@)
2018-11-03 00:24:08,298 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1113] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
# 建立套接字连接，启动会话，包含客户端地址和服务器端地址(ClientCnxn$SendThread@)
2018-11-03 00:24:08,438 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@948] - Socket connection established, initiating session, client: /0:0:0:0:0:0:0:1:49726, server: localhost/0:0:0:0:0:0:0:1:2181
# 在服务器上，会话建立完成，包含会话标识(sessionid)、协商超时
2018-11-03 00:24:08,485 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1381] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x1001cc4b13c005a, negotiated timeout = 30000

# 监视者
WATCHER::

# 监视的事件，包含状态(SyncConnected-同步已连接)、类型、znode节点路径
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0]
# 罗列ZK根目录下的子节点(树形目录结构)
[zk: localhost:2181(CONNECTED) 0] ls /
[cells, dubbo, dubbo_dev, dubbo_test, zookeeper]
[zk: localhost:2181(CONNECTED) 1]
# 退出客户端
[zk: localhost:2181(CONNECTED) 1] quit
# main线程
# ZK会话结束(main:ZooKeeper@)
2018-11-03 00:24:14,856 [myid:] - INFO  [main:ZooKeeper@1326] - Session: 0x1001cc4b13c005a closed
# main-EventThread，事件处理线程
# 客户端事件线程关闭会话(ClientCnxn$EventThread@)
2018-11-03 00:24:14,856 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@513] - EventThread shut down for session: 0x1001cc4b13c005a
```

线程列表：
* main线程
* main-SendThread(ip:port)，数据发送线程
* main-EventThread，事件处理线程

