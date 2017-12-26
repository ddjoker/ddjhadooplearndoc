### hadoop版本以及所在目录

1. version 2.7.5
2. path : `/data/hadoop-2.7.5`
3. 以下定义`$HADOOP_HOME`为上述路径
4. 现在已经进入`/data/hadoop-2.7.5`路径中。以下所有说明均以此为根。


## 启动之前的工作
清理临时文件（如果有必要）
```bash
rm -r hadoop-2.7.3/logs/*
rm -r /data/hadoop/storage/tmp/*
```

格式化namenode（是对namenode进行初始化）
```
cd bin
./hdfs namenode -format
```
或
```
cd bin
./hadoop namenode -format
```

## 启动

先启动HDFS
```
cd sbin
./start-dfs.sh
```
再启动YARN
```
cd sbin
./start-yarn.sh
```
启动job-historyserver
```
cd sbin
./mr-jobhistory-daemon.sh start historyserver
```


## 验证

1. 访问地址 http://192.168.81.128:50070 能正常访问即可.

> 注: `192.168.81.128`虚拟机外网地址
2. 程序验证
创建hdfs文件夹并上传测试数据
```bash
./bin/hadoop fs -mkdir /input
./bin/hadoop fs -put etc/hadoop/*.xml /input
```
执行测试程序进行测试
```
./bin/hadoop jar ../hadoop-examples-1.2.1.jar wordcount /input output
```
> 注 : output 会在 `hdfs://192.168.81.128:8032/user/eagle/` 目录下

> 注2 : 可以使用 `./bin/hadoop fs -ls /`来查看hdfs对应目录结构.

> 注3 : 执行上述语句后会显示一个错误
>  ```
>  mapred.ClientServiceDelegate: Application state is completed. FinalApplicationStatus=SUCCEEDED.
>
>ipc.Client: Retrying connect to server: eagle-dong-test/192.168.81.128:10020.
> ```
> 这个错误是因为没有启动 [jobhistory server](./hadoop\ job\ history\ server.md) 启动即可
>
>
