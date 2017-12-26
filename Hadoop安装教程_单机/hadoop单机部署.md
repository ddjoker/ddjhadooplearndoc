# hadoop安装与配置

### hadoop版本以及所在目录

1. version 2.7.5
2. path : `/data/hadoop-2.7.5`
3. 以下定义`$HADOOP_HOME`为上述路径
4. 现在已经进入`/data/hadoop-2.7.5`路径中。以下所有说明均以此为根。

## hadoop-env.sh

```
cd etc/hadoop
vim hadoop-env.sh
```
注：`etc/hadoop`前面不要加`/`，这个路径在`$HADOOP_HOME`下

添加如下内容
```
JAVA_HOME=/usr/lib/jvm/java-1.8.0
export JAVA_HOME
export CLASS_PATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
PATH=$JAVA_HOME/bin:$PATH:$HOME/bin
export HADOOP_SSH_OPTS="-p 22"
```

注：`JAVA_HOME`参考`开发环境配置.md`中JAVA配置即可。

注2：`HADOOP_SSH_OPTS`使用的就是ssh中的端口号，如果不是`22`,修改此数字一致即可。

## core-site.xml

```
cd etc/hadoop
vim core-site.xml
```
在`<configuration></configuration>`添加如下配置
```
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://eagle-dong-test:9000</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/data/hadoop/storage/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>hadoop.proxyuser.eagle.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.eagle.groups</name>
        <value>*</value>
    </property>
    <!--property>
        <name>hadoop.native.lib</name>
        <value>true</value>
    </property-->
```

1. `fs.defaultFS` :主要配置，配置hadoop启动端口号等信息，作为访问地址。
2. `hadoop.tmp.dir`:写入的临时目录。
3. `hadoop.proxyuser.eagle.hosts`,`hadoop.proxyuser.eagle.groups`:代理用户的可以代理的地址和可以代理的组。`*`所有的代理用户和组都使用此用户`eagle`来操作`hdfs`

## hdfs-site.xml
```
cd etc/hadoop
vim hdfs-site.xml
```
在`<configuration></configuration>`添加如下配置
```
 <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/data/hadoop/storage/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/data/hadoop/storage/hdfs/data</value>
    </property>
```
文件实际存储路径。

## mapred-site.xml
```
cd etc/hadoop
vim mapred-site.xml
```
如果没有这个文件，使用如下方式创建
```
cp mapred-site.xml.template mapred-site.xml
```
在`<configuration></configuration>`添加如下配置
```
    <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    </property>
    <property>
            <name>mapreduce.jobhistory.address</name>
            <value>eagle-dong-test:10020</value>
    </property>
    <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>eagle-dong-test:19888</value>
    </property>
```

## yarn-site.xml

```
cd etc/hadoop
vim yarn-site.xml
```
在`<configuration></configuration>`添加如下配置
```
    <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>eagle-dong-test</value>
    </property>
    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>
```
## slaves

```
cd etc/hadoop
vim yarn-site.xml
```

删除掉localhost

添加如下内容
```
eagle-dong-test
```

## 配置存储目录

根据上述配置`file:/data/hadoop`需要在`/data`下创建`hadoop`目录
```
cd /data
mkdir hadoop
chown eagle hadoop
chgrp eagle hadoop
```

为了安全起见
```
chown eagle hadoop-2.7.5 -R
chgrp eagle hadoop-2.7.5 -R
```
