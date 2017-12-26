## jps

1. 查看OpenJDK版本
```
rpm -qa | grep openjdk
```
可以看到安装的是 java 1.8版本

2.`yum install -y  java-1.8.0-openjdk-devel`

## 找到预安装的Java
1.
```
which java
```

result:

    /usr/bin/java

2.
```
ll /usr/bin |grep java
```
result:

    lrwxrwxrwx. 1 root root         22 12月 25 15:34 java -> /etc/alternatives/java
    lrwxrwxrwx. 1 root root         23 12月 25 15:34 javac -> /etc/alternatives/javac
    lrwxrwxrwx. 1 root root         25 12月 25 15:34 javadoc -> /etc/alternatives/javadoc
    lrwxrwxrwx. 1 root root         23 12月 25 15:34 javah -> /etc/alternatives/javah
    lrwxrwxrwx. 1 root root         23 12月 25 15:34 javap -> /etc/alternatives/javap
    lrwxrwxrwx. 1 root root         24 12月 20 16:03 javaws -> /etc/alternatives/javaws
    -rwxr-xr-x. 1 root root       2672 11月  6 2016 javaws.itweb

3.


```
ll /usr/alternatives |grep java
```

result

    lrwxrwxrwx. 1 root root 73 12月 25 15:34 java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/bin/java

4.

```
cd /usr/lib/jvm/
ll
```

result :

    lrwxrwxrwx. 1 root root  26 12月 25 15:34 java -> /etc/alternatives/java_sdk
    drwxr-xr-x. 4 root root 100 12月 20 15:58 java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64
    lrwxrwxrwx. 1 root root  32 12月 25 15:34 java-1.8.0 -> /etc/alternatives/java_sdk_1.8.0
    lrwxrwxrwx. 1 root root  40 12月 25 15:34 java-1.8.0-openjdk -> /etc/alternatives/java_sdk_1.8.0_openjdk
    drwxr-xr-x. 7 root root  68 12月 20 01:44 java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64
    lrwxrwxrwx. 1 root root  34 12月 25 15:34 java-openjdk -> /etc/alternatives/java_sdk_openjdk
    lrwxrwxrwx. 1 root root  21 12月 25 15:34 jre -> /etc/alternatives/jre
    lrwxrwxrwx. 1 root root  27 12月 20 15:59 jre-1.7.0 -> /etc/alternatives/jre_1.7.0
    lrwxrwxrwx. 1 root root  35 12月 20 15:59 jre-1.7.0-openjdk -> /etc/alternatives/jre_1.7.0_openjdk
    lrwxrwxrwx. 1 root root  51 12月 20 15:58 jre-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64 -> java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64/jre
    lrwxrwxrwx. 1 root root  27 12月 25 15:34 jre-1.8.0 -> /etc/alternatives/jre_1.8.0
    lrwxrwxrwx. 1 root root  35 12月 25 15:34 jre-1.8.0-openjdk -> /etc/alternatives/jre_1.8.0_openjdk
    lrwxrwxrwx. 1 root root  51 12月 25 15:34 jre-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 -> java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre
    lrwxrwxrwx. 1 root root  29 12月 25 15:34 jre-openjdk -> /etc/alternatives/jre_openjdk

## 配置JAVA_HOME

```
vi ~/.bashrc_profile

```

进行如下修改
```
JAVA_HOME=/usr/lib/jvm/java-1.8.0
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH:$HOME/bin
```



可以使用 `/etc/profile` `/etc/bashrc` 用于全局设置

可以使用 `~/.bashrc` `~/.bashrc_profile` 来保存用户当前变量

修改完毕后使用` source ~/.bashrc_profile`(修改哪个 就用哪个全路径)来针对于当前命令窗口使其临时起作用。下次重启后即可正常使用而不需要`source`了


附注： `~/.bash_history`保存了历史操作记录
