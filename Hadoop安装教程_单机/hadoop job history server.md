Hadoop自带了一个历史服务器，可以通过历史服务器查看已经运行完的Mapreduce作业记录，比如用了多少个Map、用了多少个Reduce、作业提交时间、作业启动时间、作业完成时间等信息。默认情况下，Hadoop历史服务器是没有启动的，我们可以通过下面的命令来启动Hadoop历史服务器

``` bash
./sbin/mr-jobhistory-daemon.sh start historyserver
```

这样我们就可以在相应机器的`19888`端口上打开历史服务器的WEB UI界面。可以查看已经运行完的作业情况。历史服务器可以单独在一台机器上启动，主要是通过在`mapred-site.xml`文件中配置如下参数：

``` xml
<property>
<name>mapreduce.jobhistory.address</name>
<!-- 配置实际的主机名和端口-->
<value>master:10020</value>
</property>

<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>master:19888</value>
</property>
```
