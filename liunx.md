# liunx学习

## apt管理软件包

* source源文件

```shell
vim /etc/apt/sorces.list
```

* 更新源

  1.备份

  ```shell
  sudo mv /etc/apt/sorces.list /etc/apt/sorces.list.bak
  ```

  2.编辑文件

  ```shell
  sudo vim /etc/apt/sorces.list
  ```

  3.替换文件

  ```shell
  deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
  ##测试版源
  deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
  # 源码
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
  ##测试版源
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
  # Canonical 合作伙伴和附加
  deb http://archive.canonical.com/ubuntu/ xenial partner
  deb http://extras.ubuntu.com/ubuntu/ xenial main
  ```

  4.更新

  ```shell
  sudo apt-get update
  sudo apt-get upgrade
  ```

## ssh安装

* 安装

  ```shell
  sudo apt-get install openssh-server
  ```

* 查看状态

  ```shell
  sudo service sshd status
  ```

  ```shell
  sudo netstat -anp | more
  ```