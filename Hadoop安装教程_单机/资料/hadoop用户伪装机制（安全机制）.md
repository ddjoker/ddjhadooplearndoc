本文将从用户伪装（impersonate，翻译成“模仿”也许更好些）角度介绍Hadoop安全机制，用户伪装机制使得Hadoop支持类似于linux “sudo”的功能，即用户A以用户B的身份执行功能。该机制属于Hadoop安全机制的一部分，因此适用于Hadoop 1.0之后的版本（尽管本文标题包含“hadoop 2.0”字样）。
在Hadoop 1.0之前，用户管理方面是非常脆弱的，在MapReduce中，用户可在任意一台机器上，使用作业属性user.name设置作业的“提交用户”（默认读取linux用户），而JobTracker只凭这个属性值判断作业的拥有者，这极容易让集群处于危险状态。

自从Hadoop 1.0后，Hadoop安全机制出现了，该机制基于成熟的kerberos实现，看起来非常好。但由于Kerberos本身的复杂性，使得Hadoop 启用安全机制后将变得非常难以管理，因此目前极少公司开启该功能。

即使你没有开启Hadoop安全功能，Hadoop默认的安全机制也能够从一定程度上避免旧版本带来的问题。在Hadoop 1.0版本之后，用户作业的提交用户是客户端的作业提交的Linux用户，该用户由Hadoop客户端通过RPC发送给服务器端，用户不能通过参数胡乱修改作业提交者。当然，这样也是存在问题的，如果Hadoop客户端管理混乱，每个人均能够将自己的机器作为客户端，则可以在机器上创建任意linux用户提交作业，这将使得集群处于极大地危险中，因此，Hadoop客户端必须得到受到一定限制。

Hadoop 1.0安全机制的引入，使得Hadoop上层系统不能够将实际用户传递到Hadoop层。举例说明，对于Oozie这个作业调度系统而言，通常我们会创建一个叫“oozie”的用户启动这个服务，这样，所以从Oozie发出的作业的提交者全部变成了oozie，不管原始提交者是user1还是user2，也就是说，实际的用户被掩盖了。



为了解决该问题，Hadoop引入了安全伪装的功能，该功能允许一个超级用户代理其他用户执行作业或者命令，但对外看来执行者仍是普通用户，比如在Oozie中，我们需要达到这样的效果：用户user1提交了一个作业流，该作业流的执行者实际上是oozie，但对外看来应是user1（也可以这样理解， 用户user1  sudo 成用户oozie，执行作业流）为此，可以在jobtracker（hadoop 1.0）、主备namenode和resoucemanager（hadoop 2.0）上的core-site.xml中增加以下配置：
```xml
<property>
   <name>hadoop.proxyuser.oozie.groups</name>
   <value>group1,group2<value>
</property>
<property>
    <name>hadoop.proxyuser.oozie.hosts</name>
    <value>host1,host2<value>
</property>
```
这里，假设用户user1属于group1（注意，这里的user1和group1都是linux用户和用户组，需要在namenode和jobtracker上进行添加），此外，为了限制客户端随意部署，超级用户代理功能只支持host1和host2两个节点。经过以上配置后，在host1和host2上的客户端上，属于group1和group2的用户可以sudo成oozie用户，执行作业流。

增加以上配置后，无需重启集群，可以直接用hadoop管理员账号重新加载这两个属性值，命令为：

```bash
bin/hdfs dfsadmin –refreshSuperUserGroupsConfiguration
bin/yarn rmadmin –refreshSuperUserGroupsConfiguration
```

如果集群配置了HA，需要在为主备namenode（node000和node001）同时加载这两个属性（只加载一个不行），命令如下：
```bash
#!/usr/bin/env bash
bin/hadoop dfsadmin -fs hdfs://node000:8020 –refreshSuperUserGroupsConfiguration
bin/hadoop dfsadmin -fs hdfs://node001:8020 –refreshSuperUserGroupsConfiguration
```
如果加载成功，可以在主备namenode日志中看到如下一条日志（ResourceManager类似）：

```log
2014-03-18 17:11:05,983 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: Refreshing SuperUser proxy group mapping list
```
如果使用HDFS或者提交MapReduce作业时，出现如下错误，说明你的配置仍然存在问题，可仔细检查是否有遗漏：

```log
User: oozie is not allowed to impersonate user1
```
整个流程的实现原理借助了Java JDK中的安全库，大体流程是，客户端获取当前用户和代理用户，并通过RPC传递给服务器端，服务器端验证通过后也进行后续操作。查看代码，可看到主要实现代码是org.apache.hadoop.security. UserGroupInformation这个类，该类非常不易理解，关键点在：

（1） UserGroupInformation#getCurrentUser函数

该函数比较难理解之处在于Server端调用时，通过
```java
AccessControlContext context = AccessController.getContext();
```
获取一个AccessControlContext对象，该对象包含当前线程中所有的Subject实体，由于服务器端会启动多个handler线程并行处理客户端请求，因此每个handler内部的AccessControlContext均是不同的，它们通常与对应的客户端信息相同。很多人阅读代码时不明白AccessControlContext是怎样自动填充的？这个对象的内容在RPC 层调用UserGroupInformation#doAs函数时会自动填充，具体可了解JDK中对AccessControlContext的说明文档。

（2） UserGroupInformation#createProxyUser函数

该函数用户创建超极用户和代理用户对，一个UserGroupInformation对象中可包含一个超极用户和一个代理用户，超极用户被称为“super user”或“real user”，代理用户被称为作业提交者或“proxy user”。

最后，用户伪装机制还有一个妙用:模拟多个用户并行提交作业，方法如下：

在jobtracker（hadoop 1.0）、namenode和resoucemanager（hadoop 2.0）中增加前面的配置，然后创建一个oozie、user1、user2三个linux用户，其中，user1和user2属于group1，然后切换至oozie用户，并编写以下脚本：

# multi_user.sh

export HADOOP_PROXY_USER=$1 #Hadoop会自动识别这个环境变量
```
bin/hadoop jar hadoop-example.jar pi 10 1000 #自己根据hadoop部署位置进行修改
```
修改权限：

chmod +x multi_user.sh

之后使用oozie提交运行该作业

./multi_user.sh user1 &

./multi_user.sh user2 &

可以在界面上看到，这两个作业的提交者分别是user1和user2，而不是oozie。
