## 环境定义

1. 共有三台集群机器，主机域名如下
    1. eagle-dong-test
    2. eagle01
    3. eagle02
2. HADOOP环境变量以及对应SSH权限已开启
    1. ssh默认端口为22
    2. JAVA环境已部署。
    3. 三台机器已互通
3. 此步骤在单机部署环境之后。

## 环境部署

### 配置三台机器的SSH无密码登陆

在`eagle-dong-test`执行`eagle01`登陆ssh
```
ssh eagle01
```
注：ssh默认端口为22,线上一般会更改  所以要加 -p 命令强行指定端口号。

输入密码后`exit`退出此远程方式。
依次执行以下代码
```
cd ~/.ssh/                    # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat id_rsa.pub >> authorized_keys  # 加入授权
chmod 600 ./authorized_keys    # 修改文件权限，必须修改否则无法登陆
```
用于生成秘钥文件

重复执行此动作用于 `eagle02`的SSH登陆。

在每台机器上对另外两台任意执行此上述动作，用于完成三台机器间的无密码登陆。


### 复制`eagle-dong-test`配置文件到另外两台机器`eagle01` `eagle02`

```bash
tar cvzf hadoopConfiged.tar.gz  hadoop2.7.5
scp  hadoopConfiged.tar.gz eagle@eagle01:/data/soft/
scp  hadoopConfiged.tar.gz eagle@eagle02:/data/soft/
```

注 : 一脸懵逼
