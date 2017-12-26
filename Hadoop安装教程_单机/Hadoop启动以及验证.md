### hadoop版本以及所在目录

1. version 2.7.5
2. path : `/data/hadoop-2.7.5`
3. 以下定义`$HADOOP_HOME`为上述路径
4. 现在已经进入`/data/hadoop-2.7.5`路径中。以下所有说明均以此为根。


## 启动之前的工作
清理临时文件（如果有必要）
```
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
#再启动YARN
```
cd sbin
./start-yarn.sh
```
