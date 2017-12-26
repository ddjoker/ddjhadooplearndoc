## maven在eclipse建立工程,运行出现Server IPC version 9 cannot communicate with client version 4

此配置在使得hadoop客户端版本过低
```xml
<dependency>
<groupId>org.apache.hadoop</groupId>
<artifactId>hadoop-core</artifactId>
<version>1.2.1</version>
</dependency>
```
用
```xml
<dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.5.1</version>
    </dependency>
    <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.5.1</version>
    </dependency>
    <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.5.1</version>
    </dependency>
```
替换即可

## Permission denied: user=*, access=WRITE, inode="/tmp/hadoop-yarn/staging/dong/.staging":root:supergroup:drwxr-xr-x

原因是没有权限
`/data/hadoop-2.7.3/bin/hadoop fs -chmod -R 777  /`
不能解决问题。

尝试使用修改默认权限的方式去更改
