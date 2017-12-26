

## 卸载老版本的 docker 及其相关依赖
```shell
sudo yum remove docker docker-common container-selinux docker-selinux docker-engine
```

## 安装 yum-utils，它提供了 yum-config-manager，可用来管理yum源
```shell
sudo yum install -y yum-utils
```

## 添加yum源
```shell
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## 更新yum索引
```shell
sudo yum makecache fast
```

## 安装 docker-ce
```shell
sudo yum install docker-ce
```

## 启动 docker
```shell
sudo systemctl start docker
```

## 验证是否安装成功
```shell
sudo docker info
```
