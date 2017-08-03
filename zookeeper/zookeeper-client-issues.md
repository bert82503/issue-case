
# FAQ

## [Curator](http://curator.apache.org)
### 2017-08-02 19:01:52,715 ERROR [main-EventThread] o.a.c.f.i.CuratorFrameworkImpl - [] [] [] Background exception was not retry-able or retry gave up
问题根源：curator-recipes 3.1.0版本中(EnsembleTracker.java:153)存在bug(server.clientAddr = null)，会引起NPE异常；
3.2.1+版本已修复此问题，但建议使用最新的3.x.x。（结论：玩中间件要记得**版本问题**）

1、异常堆栈信息
```
2017-08-02 12:28:41,284 INFO  [main] org.apache.zookeeper.ZooKeeper - [] [] [] Client environment:os.memory.total=1973MB
2017-08-02 12:28:41,293 INFO  [main] org.apache.zookeeper.ZooKeeper - [] [] [] Initiating client connection,
connectString=zkserver1.staging.xxx.info:2181,zkserver2.staging.xxx.info:2181,zkserver3.staging.xxx.info:2181 sessionTimeout=30000 watcher=org.apache.curator.ConnectionState@47e2e487
2017-08-02 12:28:41,338 INFO  [main-SendThread(zkserver3.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Opening socket connection to server zkserver3.staging.xxx.info/ip:2181. Will not attempt to authenticate using SASL (unknown error)
2017-08-02 12:28:41,339 INFO  [main-SendThread(zkserver3.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Socket connection established, initiating session, client: /ip:64478, server: zkserver3.staging.xxx.info/ip:2181
2017-08-02 12:28:41,352 INFO  [main-SendThread(zkserver3.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Session establishment complete on server zkserver3.staging.xxx.info/ip:2181, sessionid = 0x30080abe24300f6, negotiated timeout = 30000
2017-08-02 12:28:41,361 INFO  [main-EventThread] o.a.c.f.s.ConnectionStateManager - [] [] [] State change: CONNECTED
# 配置事件接收到数据
2017-08-02 12:28:41,383 INFO  [main-EventThread] o.a.c.f.imps.EnsembleTracker - [] [] [] New config event received: ...ignore...
2017-08-02 12:28:41,396 ERROR [main-EventThread] o.a.c.f.i.CuratorFrameworkImpl - [] [] [] Background exception was not retry-able or retry gave up
java.lang.NullPointerException: null
        at org.apache.curator.framework.imps.EnsembleTracker.configToConnectionString(EnsembleTracker.java:153)
        at org.apache.curator.framework.imps.EnsembleTracker.processConfigData(EnsembleTracker.java:168)
        at org.apache.curator.framework.imps.EnsembleTracker.access$200(EnsembleTracker.java:49)
        at org.apache.curator.framework.imps.EnsembleTracker$2.processResult(EnsembleTracker.java:135)
        at org.apache.curator.framework.imps.CuratorFrameworkImpl.sendToBackgroundCallback(CuratorFrameworkImpl.java:823)
        at org.apache.curator.framework.imps.CuratorFrameworkImpl.processBackgroundOperation(CuratorFrameworkImpl.java:606)
        at org.apache.curator.framework.imps.WatcherRemovalFacade.processBackgroundOperation(WatcherRemovalFacade.java:152)
        at org.apache.curator.framework.imps.GetConfigBuilderImpl$2.processResult(GetConfigBuilderImpl.java:210)
        at org.apache.zookeeper.ClientCnxn$EventThread.processEvent(ClientCnxn.java:616)
        at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:528)
2017-08-02 12:28:41,434 INFO  [main] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
2017-08-02 12:28:41,468 INFO  [main] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
2017-08-02 12:28:45,207 INFO  [main] o.s.w.s.m.m.a.RequestMappingHandlerAdapter - [] [] [] Looking for @ControllerAdvice:
org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@470e2030:
startup date [Wed Aug 02 12:28:25 CST 2017]; root of context hierarchy
```

2、3.1.0 版本的源码：
```java
    public static String configToConnectionString(QuorumVerifier data) throws Exception
    {
        StringBuilder sb = new StringBuilder();
        for ( QuorumPeer.QuorumServer server : data.getAllMembers().values() )
        {
            if ( sb.length() != 0 )
            {
                sb.append(",");
            }
            // [bug] server.clientAddr = null
            sb.append(server.clientAddr.getAddress().getHostAddress()).append(":").append(server.clientAddr.getPort());
        }

        return sb.toString();
    }
```

3、3.3.0 版本的源码：
```java
    public static String configToConnectionString(QuorumVerifier data) throws Exception
    {
        StringBuilder sb = new StringBuilder();
        for ( QuorumPeer.QuorumServer server : data.getAllMembers().values() )
        {
            if ( sb.length() != 0 )
            {
                sb.append(",");
            }
            InetSocketAddress address = Objects.firstNonNull(server.clientAddr, server.addr);
            sb.append(address.getAddress().getHostAddress()).append(":").append(address.getPort());
        }

        return sb.toString();
    }
```

4、正常日志信息
```
2017-08-03 12:55:03,497 INFO  [main] org.apache.zookeeper.ZooKeeper - [] [] [] Client environment:os.memory.total=1973MB
2017-08-03 12:55:03,508 INFO  [main] org.apache.zookeeper.ZooKeeper - [] [] [] Initiating client connection,
connectString=zkserver1.staging.xxx.info:2181,zkserver2.staging.xxx.info:2181,zkserver3.staging.xxx.info:2181 sessionTimeout=30000 watcher=org.apache.curator.ConnectionState@61001b64
2017-08-03 12:55:03,545 INFO  [main-SendThread(zkserver1.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Opening socket connection to server zkserver1.staging.xxx.info/ip:2181. Will not attempt to authenticate using SASL (unknown error)
2017-08-03 12:55:03,552 INFO  [main-SendThread(zkserver1.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Socket connection established, initiating session, client: /ip:18519, server: zkserver1.staging.xxx.info/ip:2181
2017-08-03 12:55:03,565 INFO  [main] o.a.c.f.i.CuratorFrameworkImpl - [] [] [] Default schema
2017-08-03 12:55:03,574 INFO  [main-SendThread(zkserver1.staging.xxx.info:2181)] org.apache.zookeeper.ClientCnxn - [] [] []
Session establishment complete on server zkserver1.staging.xxx.info/ip:2181, sessionid = 0x10080acc7cb010f, negotiated timeout = 30000
2017-08-03 12:55:03,600 INFO  [main-EventThread] o.a.c.f.s.ConnectionStateManager - [] [] [] State change: CONNECTED
# CONNECTION_RECONNECTED：重新连接
2017-08-03 12:55:03,641 INFO  [Curator-PathChildrenCache-0] c.w.codis.impl.RoundRobinImpl - [] [] [] zookeeper event received: type=CONNECTION_RECONNECTED
2017-08-03 12:55:03,642 INFO  [main-EventThread] o.a.c.f.imps.EnsembleTracker - [] [] [] New config event received:
{server.1=zkserver1.staging.xxx.info:2888:3888:participant, version=100000000, server.3=zkserver3.staging.xxx.info:2888:3888:participant, server.2=zkserver2.staging.xxx.info:2888:3888:participant}
2017-08-03 12:55:03,700 INFO  [main] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
# PathChildrenCache：增加子路径
2017-08-03 12:55:03,701 INFO  [Curator-PathChildrenCache-0] c.w.codis.impl.RoundRobinImpl - [] [] [] zookeeper event received:
type=CHILD_ADDED, path=/zk/codis/db_staging/proxy/staging_2, stat=4294985104,4294985108,1500021538427,1500022038554,1,0,0,216314257837588483,213,0,4294985104
2017-08-03 12:55:03,702 INFO  [Curator-PathChildrenCache-0] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
2017-08-03 12:55:03,727 INFO  [main] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
2017-08-03 12:55:03,731 INFO  [Curator-PathChildrenCache-0] c.w.codis.impl.RoundRobinImpl - [] [] [] Add new proxy: ip:19000
2017-08-03 12:55:04,567 INFO  [main-EventThread] o.a.c.f.imps.EnsembleTracker - [] [] [] New config event received:
{server.1=zkserver1.staging.xxx.info:2888:3888:participant, version=100000000, server.3=zkserver3.staging.xxx.info:2888:3888:participant, server.2=zkserver2.staging.xxx.info:2888:3888:participant}
2017-08-03 12:55:07,424 INFO  [main] o.s.w.s.m.m.a.RequestMappingHandlerAdapter - [] [] [] Looking for @ControllerAdvice:
org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@470e2030:
startup date [Thu Aug 03 12:54:50 CST 2017]; root of context hierarchy
```
