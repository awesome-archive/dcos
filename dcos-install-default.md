## 独立主机环境

### 1.硬件资源准备

| 节点 | 配置 | 操作系统 | 数量 | 备注 |
| --- | --- | --- | --- | --- |
| Bootstrap node | 2核，16G内存，60G硬盘 | CentOS 7.2 | 1 | 该节点适用于GUI安装模式，节点不在集群内（安装完成后，保留安装环境的同时，也可以加入到集群） |
| Master nodes | 2核，16G内存，128G硬盘 | CentOS 7.2 | 1 |  |
| Agent nodes | 2核，16G内存，128G硬盘 | CentOS 7.2 | 3 |  |

### 2.系统环境准备

更新系统到最新版本

```
$ sudo yum update -y
```

禁用系统防火墙

```
$ sudo systemctl stop firewalld && sudo systemctl disable firewalld
```

DC\/OS安装在\/opt\/mesosphere下，确保该目录在一个非 LVM逻辑磁盘或共享存储磁盘下。（注：实际测试时，在LVM盘下仍安装成功）。

### 3.端口协议准备

* 所有节点上的SSH服务可用

* ICMP服务在所有节点可用

* 通过Bootstrap节点可以访问到所有其他节点

* 集群内所有节点间的IP-to-IP访问不受约束

* 在Masters节点上，UDP在**53**上开放端口。Mesos的Agent节点服务使用此端口查找Master节点中的Leader节点。


### 4.软件环境准备

#### 4.1 Docker

**范围：**所有节点

**版本：**1.7.x ~ 1.11.x ** 注意：**当前DCOS（1.8.x~1.9.x)暂不支持1.12.x

**推荐：**

* 以稳定性考虑，推荐1.9.x~1.11.x

* 不要使用Docker的`loop-lvm模式下的devicemapper存储引擎，原因参考：`[`Docker and the Device Mapper storage driver`](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/)

* 推荐`OverlayFs或者direct-lvm模式下的devicemapper存储引擎`

* 通过Systemd管理Docker服务，确保Docker服务崩溃时自动重启

* 以root权限运行Docker的命令，或者运行命令的用户在docker用户组内


**参考：**

[在Centos7.2上安装Docker](/dcos-install-docker-on-centos.md)

#### 4.2 禁用sudo命令的密码输入提示

范围：所有节点

执行`%wheel ALL=(ALL) NOPASSWD: ALL`命令，或者以root用户进行SSH登录

#### 4.3 启用NTP服务

**范围：**所有节点

使用NTP服务同步时间，执行以下命令安装：

```
$ sudo yum install -y ntp
$ sudo systemctl start ntpd && sudo systemctl enable ntpd
```

通过以下命令检查服务是否存在

```
$ ntptime
```

#### 4.4 Bootstrap node

**范围：**Bootstrap node

在安装时，如果启用`exhibitor_storage_backend`: zookeeper配置，bootstrap节点将成为集群的永久组成部分。通过设置该参数，Mesos的master节点中的leader状态和leader选举都通过安装在Bootstrap节点上得Exhibitor Zookeeper来维护。当前该参数支持：`static`，`zookeeper`，`aws_s3`和`azure`，详细信息请参考

Booststrap节点必须独立于集群之外。

#### 4.5 Docker Nginx（高级安装）

**范围：**Bootstrap node

此步骤仅用于执行[高级安装模式](https://dcos.io/docs/1.8/administration/installing/custom/advanced/)

#### 4.6 压缩软件

**范围：** 集群节点

```
$ sudo yum install -y tar xz unzip curl ipset
```

#### 4.7 安全设置

**范围：**集群节点

检查是否禁用SELinux

```
$ getenforce
```

禁用SELinux或设置为permissive模式

添加**nogroup**到所有节点

重启所有节点，启用配置

```
$ sudo sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config && sudo groupadd nogroup && sudo reboot
```

#### 4.8 SSH通道配置

在Bootstrap节点上通过ssh-keygen生成公私钥对，将公钥内容添加到所有集群节点的`/root/.ssh/authorized_keys`文件（以root用户安装）中。

### 5.安装DC/OS

#### DC/OS安装文件

**范围：**Bootstrap node

下载[DC/OS安装文件](https://downloads.dcos.io/dcos/stable/dcos_generate_config.sh) 到Bootstrap节点。该文件也可以用来创建自定义的DC/OS编译文件。

```
curl -O https://downloads.dcos.io/dcos/stable/dcos_generate_config.sh
```

#### 安装模式

当前DC/OS支持**GUI**、**CLI**和**高级安装**三种模式。

