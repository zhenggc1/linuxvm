## Docker 网络

#### 资源

1. 主要见docker的文档，没啥需要下载的 

#### 步骤

1. 这里主要讲一个和host网络同一网段完全打通的方法，其他很多的都可以直接见docker的网站

2. 和主机网络打通主要靠macvlan，以及一个物理的接口，假设是enp4s0，然后在docker上设置macvlan网络，首先规划网络，这里需要可以控制你自己的dhcp，不能分配和docker分配的重复
```
这里假设docker可以用到 192.168.1.128/27，也就是192.168.1.128--192.168.1.160
```

3. 现在主机上设置一个macvlan的接口，并且设置ip地址
```
ip link add vlan10 link enp4s0 type macvlan mode bridge
ip link set vlan10 up

ip addr add 192.168.1.159/24 dev vlan10
ip route add 192.168.1.128/27 via 192.168.1.159 dev vlan10

```

4. 创建docker的macvlan网络
```
docker network create -d macvlan --subnet=192.168.1.0/24 -o parent=enp4s0 \
 --gateway 192.168.1.1 --ip-range 192.168.1.128/27 --aux-address 'host=192.168.1.159' home

```

5. 就可以使用docker来创建容器了
```
docker run -it --rm --name test --network home alpine:latest

在docker中，可以
ping 192.168.1.1 看网络情况
也可以ping 宿主机的ip，159的可以，另外enp4s0的地址也可以ping通
ip a 可以看到ip地址，第一个分配的应该是192.168.1.128

在docker外的任何内网的主机上，可以ping 192.168.1.128，看是否可以ping通

在docker宿主机上，也可以ping 通128

```

6. 有这以后，感觉就是一台独立的主机了，在你自己网络的内部可以随便玩。

### 直接打通本机的网桥
1. 如果本机已经有网桥了，可以直接打通本机的网桥
2. 首先要禁止docker启动的时候启动默认网桥，iptables也不用了，在/etc/docker/daemon.json中添加以下内容，要符合json的格式，如果原来没有需要参考daemon.json的写法
```
"iptables":false,
"bridge":"none"
```
3. 重启docker服务后，可以看到原来名为bridge的网络已经没有了，这个时候创建自己的bridge网络，subnet是你的子网，iprange是docker可以使用的ip地址列表，br0是你的本机网桥的名称，home是你的docker使用的网络名称
```
docker network create --subnet=192.168.1.0/24 --gateway=192.168.1.1 \
 --ip-range 192.168.1.128/26 -o com.docker.network.bridge.name=br0 -o \
 com.docker.network.bridge.inhibit_ipv4=true home
```
