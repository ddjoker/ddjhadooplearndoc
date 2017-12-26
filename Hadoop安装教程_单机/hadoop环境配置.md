## 修改主机名
```
vi  /etc/sysconfig/network
```
添加如下内容
```
HOSTNAME=eagle-dong-test
```

强制启用此配置

```
hostname comex01-ct65
```

## 修改主机映射

```
vim /etc/hosts
```
添加如下内容
```

192.168.81.128 eagle-dong-test
```

注： `192.168.81.128`通过ifconfig 获取

注2： `eagle-dong-test` 与主机名相同即可

注3： 这个配置只是在本机强制加上域名解析，就是当本机访问`eagle-dong-test`会根据此配置文件解析到`192.168.81.128`，所以只针对于本机访问生效。详细请百度：IP与域名解析（dns服务)

注4： 线上环境可以在本地局域网配置此域名解析规则，此步骤可省略。



## 配置SSH无密码登陆

执行本机登陆ssh
```
ssh localhost
```
注：ssh默认端口为22,线上一般会更改  所以要加 -p 命令强行指定端口号。

输入密码后`exit`退出此远程方式。

```
cd ~/.ssh
ll
```
结果

```
-rw-r--r--. 1 root root 171 12月 25 16:39 known_hosts
```
制作秘钥
```
ssh-keygen -t rsa
```
全部默认即可
```
cat id_rsa.pub >> authorized_keys
ll
```
结果
```
-rw-r--r--. 1 root root  399 12月 25 16:50 authorized_keys
-rw-------. 1 root root 1679 12月 25 16:49 id_rsa
-rw-r--r--. 1 root root  399 12月 25 16:49 id_rsa.pub
-rw-r--r--. 1 root root  171 12月 25 16:39 known_hosts

```
修改权限
```
chmod 600 ./authorized_keys
```
