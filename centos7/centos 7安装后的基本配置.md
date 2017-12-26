## network

### error :" Job for network.service failed because the control process exited with error code. See "systemctl status network.service" and "journalctl -xe" for details."

1.停止networkmanager
```shell
service networkmanager stop
```
2.关闭networkmanager服务

    1. 检查开启启动状态

    ```shell
    chkconfig |grep networkmanager
    ```

    2. 关闭服务

    ``` shell
    chkconfig networkmanager off
    ```
3. 启动network

```
service network restart
```


### 关闭防火墙

#### CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙步骤。

1. 关闭firewall：

```
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
```
2. iptables防火墙（这里iptables已经安装，下面进行配置）

```
vi/etc/sysconfig/iptables #编辑防火墙配置文件
```
``` conf
# sampleconfiguration for iptables service
# you can edit thismanually or use system-config-firewall
# please do not askus to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT[0:0]
:OUTPUT ACCEPT[0:0]
-A INPUT -m state--state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -jACCEPT
-A INPUT -i lo -jACCEPT
-A INPUT -p tcp -mstate --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -jACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080-j ACCEPT
-A INPUT -j REJECT--reject-with icmp-host-prohibited
-A FORWARD -jREJECT --reject-with icmp-host-prohibited
COMMIT
```
```
:wq! #保存退出
```
备注：这里使用80和8080端口为例。***部分一般添加到“-A INPUT -p tcp -m state --state NEW -m tcp--dport 22 -j ACCEPT”行的上面或者下面，切记不要添加到最后一行，否则防火墙重启后不生效。
systemctlrestart iptables.service #最后重启防火墙使配置生效
systemctlenable iptables.service #设置防火墙开机启动

--------------------------------------分割线 --------------------------------------

iptables使用范例详解 http://www.linuxidc.com/Linux/2014-03/99159.htm

iptables—包过滤（网络层）防火墙 http://www.linuxidc.com/Linux/2013-08/88423.htm

Linux防火墙iptables详细教程 http://www.linuxidc.com/Linux/2013-07/87045.htm

iptables+L7+Squid实现完善的软件防火墙 http://www.linuxidc.com/Linux/2013-05/84802.htm

iptables的备份、恢复及防火墙脚本的基本使用 http://www.linuxidc.com/Linux/2013-08/88535.htm

Linux下防火墙iptables用法规则详解 http://www.linuxidc.com/Linux/2012-08/67952.htm

--------------------------------------分割线 --------------------------------------

更多CentOS相关信息见CentOS 专题页面 http://www.linuxidc.com/topicnews.aspx?tid=14

本文永久更新链接地址：http://www.linuxidc.com/Linux/2015-05/117473.htm
