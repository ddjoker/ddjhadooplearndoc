在hadoop的core-site.xml中进行如下设置, 用户“super”就可以代理主机host1和host2上属于组group1和group2的所有用户。

<property>
     <name>hadoop.proxyuser.super.hosts</name>
     <value>host1,host2</value>
   </property>
   <property>
     <name>hadoop.proxyuser.super.groups</name>
     <value>group1,group2</value>
   </property>
当然，也可以进行更松弛的设置。如下所示表示用户“oozie”可以代理所有主机上的所有用户。

  <property>
    <name>hadoop.proxyuser.oozie.hosts</name>
    <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.oozie.groups</name>
    <value>*</value>
  </property>

代理机制的验证
为了搞清楚“代理”的具体含义，参考Hadoop 2.0中用户安全伪装/模仿机制实现原理，我们做如下测试。

测试1
在hadoop的core-site.xml中配置好代理用户。不过，该文件中默认有hdfs代理所有用户的配置，如下所示。那么，后续步骤就以hdfs为代理用户。

<property>
  <name>hadoop.proxyuser.hdfs.hosts</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.hdfs.groups</name>
  <value>*</value>
</property>

切换到hdfs，声明要代理的用户为ly

hdfs@lyhadoop:~$ export HADOOP_PROXY_USER=ly

提交作业

hdfs@lyhadoop:~$ hadoop jar /home/ly/wordcount.jar /user/ly/input/input.txt /user/ly/output22

查看作业界面

这里写图片描述

虽然此时的登录者为“hdfs”，但作业界面上显示的提交者为“ly”。那是因为我们在第2步设置了要被代理的用户为“ly”。

以上测试说明: hdfs提交了ly的作业，最终作业界面显示的提交者为“ly”。但hdfs提交作业的时候是使用的自己的身份还是切换到了ly，还不清楚，下面通过测试2进行验证。

测试2
设置hdfs的文件权限为ly不可访问

hdfs@lyhadoop:~$ hadoop fs -chmod 750 /user/hdfs

查看hdfs的文件

hdfs@lyhadoop:~$ hadoop fs -ls /user/hdfs/input

报错如下，

ls: Permission denied: user=ly, access=EXECUTE, inode="/user/hdfs":hdfs:supergroup:drwxr-x---

以上测试说明:hdfs提交用户ly的hadoop命令时，切换到了该用户。

代理机制的作用
以oozie为例，系统安装Oozie时，会同时创建相应用户–oozie。启动Oozie服务时，需要切换到该用户启动。以下分两种情况讨论Oozie中提交作业的流程。

不设置oozie代理
考虑到Oozie是用户oozie启动的，所以如果不设置oozie代理用户，普通用户在Oozie中提交的作业都会以oozie的身份执行，如下所示

oozie-noproxy

设置oozie代理
设置oozie代理用户后，普通用户提交作业时，oozie会切换到该用户执行作业，如下所示，

oozie-proxy

hadoop中的默认设置
hadoop的core-site.xml中，针对oozie代理用户的默认设置如下，表示oozie会代理所有主机上的所有用户。
```xml
    <property>
      <name>hadoop.proxyuser.oozie.hosts</name>
      <value>*</value>
    </property>
    <property>
      <name>hadoop.proxyuser.oozie.groups</name>
      <value>*</value>
    </property>
```
总结
以用户ops使用代理用户oozie提交作业为例，当用户ops提交作业时，oozie会接管该作业，负责作业资源的申请及监管。

但其中若遇到读取HDFS文件时，要判断是否有使用该文件的权限，此时使用的用户是ops，作业运行完后，作业列表中显示该作业的用户也是ops。当然，除此之外，剩下的工作都由oozie负责，以体现“代理”的作用。
