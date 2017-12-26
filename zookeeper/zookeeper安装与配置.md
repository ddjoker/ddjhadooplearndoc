## 环境以及版本

1. `zookeeper`配置在3.[hadoop环境配置](./Hadoop安装教程_单机/hadoop环境配置.md)配置之后。
2. `zookeeper`版本`3.4.10`
3. `zookeeper`安装路径`/data/zookeeper-3.4.10`,以后均以此目录为根目录 代称为`$ZOOKEEPER_HOME`.
4. [ZOOKEEPER_HOMEPAGE](http://zookeeper.apache.org/)

## 配置文件修改
创建目录
```bash
mkdir data log
```
创建配置文件
```bash
cd conf
cp zoo.default.cfg zoo.cfg
```
修改配置文件
```bash
cd conf
vi zoo.cfg
```
找到并修改如下内容
```
dataDir=$ZOOKEEPER_HOME/data
```

在文件末尾添加如下内容
```
dataLogDir=$ZOOKEEPER_HOME/log
```
```
server.1=eagle-dong-test:2888:3888
```

>注: `$ZOOKEEPER_HOME`为`zookeeper`安装路径,请自行替换.

>注2 : 不知道这个`$ZOOKEEPER_HOME`是否可以使用环境变量或者相对路径.

>注3 : `eagle-dong-test`为之前配置的域名.

>注4 : `server.1`注意那个`1`.

添加myid配置
```bash
cd data
echo 1 >myid
```

## 启动
``` bash
cd bin
sh zkServer.sh start
```
查看运行状态

``` bash
cd bin
sh zkServer.sh status
```
输出内容如下
```
ZooKeeper JMX enabled by default
Using config: /data/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: standalone
```

> 注: `standalone`为单机启动的意思,集群启动为`follower`

> 注2:`/data/zookeeper-3.4.10`是安装目录
