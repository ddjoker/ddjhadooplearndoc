故障描述：

前一阵发现Jobtracker服务器宕机，追查宕机前系统负载情况，发现很多线程被阻塞，大多上面一些cron job对hdfs目录请求响应时间较长导致的，进一步追查后发现有若干datanode被exclude掉，报错信息如下：

datanode报错

    ERROR1
    ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: DatanodeRegistration(x.x.x.x:50010, storageID=DS-1
    437963415-x.x.x.x-50010-1331795684559, infoPort=50075, ipcPort=50020):DataXceiver
    java.net.ConnectException: Connection refused
    ERROR2
    ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: DatanodeRegistration(x.x.x.x:50010, storageID=DS-1
    437963415-x.x.x.x-50010-1331795684559, infoPort=50075, ipcPort=50020):DataXceiver
    java.io.IOException: Connection reset by peer

    ERROR3
    ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: DatanodeRegistration(x.x.x.x:50010, storageID=DS-5
    34013790-x.x.x.x-50010-1331795710872, infoPort=50075, ipcPort=50020):DataXceiver
    java.io.IOException: Interrupted receiveBlock

    ERROR4
    ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: DatanodeRegistration(x.x.x.x:50010, storageID=DS-3
    46899942-x.x.x.x-50010-1331795684322, infoPort=50075, ipcPort=50020):DataXceiver
    java.io.EOFException: while trying to read 65557 bytes
    at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.readToBuf(BlockReceiver.java:290)
    at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.readNextPacket(BlockReceiver.java:334)
    at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.receivePacket(BlockReceiver.java:398)
    at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.receiveBlock(BlockReceiver.java:577)
    at org.apache.hadoop.hdfs.server.datanode.DataXceiver.writeBlock(DataXceiver.java:480)
    at org.apache.hadoop.hdfs.server.datanode.DataXceiver.run(DataXceiver.java:171)
仔细排查所有ERROR，发现在datanode exclude后，之前部署的metric统计脚本发现，部分datanode内存耗尽，推测datanode节点存在内存瓶颈。
java.lang.OutOfMemoryError: Java heap space
2012-09-12 04:37:38,311 ERROR org.mortbay.log: Error for /metrics

之前线上hadoop集群遇到过几次大量由于Datanode内存溢出，被强制下线后，导致线上集群不稳定的情况，故针对如上内存问题进行JVM GC的优化：

1.原FGC影响时间较长：
优化前：


      S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
      0.00  96.36  60.06  89.27  87.69   1926  387.419    93  589.308  976.728
      0.00  96.36  60.07  89.27  87.69   1926  387.419    93  589.308  976.728
      0.00  96.36  60.07  89.27  87.69   1926  387.419    93  589.308  976.728
      0.00  96.36  60.18  89.27  87.69   1926  387.419    93  589.308  976.728
      0.00  96.36  60.18  89.27  87.69   1926  387.419    93  589.308  976.728
优化后：

      41.12   0.00   2.57  79.72  59.78    302  209.850   146   49.716  259.566
      41.12   0.00   3.88  79.72  59.78    302  209.850   146   49.716  259.566
      41.12   0.00   4.18  79.72  59.78    302  209.850   146   49.716  259.566
      41.12   0.00   4.39  79.72  59.78    302  209.850   146   49.716  259.566
      41.12   0.00   4.72  79.72  59.78    302  209.850   146   49.716  259.566
平均影响时间从6.33ms降至0.34ms
2.原Old Generation占用内存量巨大，可能产生内存溢出危险：


优化前

    PS Old Generation
    capacity = 2863333376 (2730.6875MB)
    used     = 2556763896 (2438.3200607299805MB)
    free     = 306569480 (292.36743927001953MB)
    89.29326628294085% used
优化后：

    PS Old Generation
    capacity = 3221225472 (3072.0MB)
    used     = 2118978904 (2020.8157577514648MB)
    free     = 1102246568 (1051.1842422485352MB)
    65.78176294763882% used
    
占用比例从 89%->65%
之后再不存在Datanode由于GC缓慢导致RPC超时引发的一连串问题。
总结：

优化最重要花最长时间的是定位问题，所以帮助快速定位问题的工具很重要，ganglia及datanode、jobtracker 日志都给了我很大帮助，本次主要优化思路是在保证GC影响时间较短情况下，让Old Generation提早进行FGC,来保证有足够内存容量应对突发内存请求，详细参数如下：


      export HADOOP_DATANODE_OPTS=”-server -XX:+UseConcMarkSweepGC -XX:SurvivorRatio=3 -XX:MaxTenuringThreshold=20 -XX:CMSInitiatingOccupancyFraction=80 -XX:+ExplicitGCInvokesConcurrent -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime ${HADOOP_DATANODE_OPTS}”
