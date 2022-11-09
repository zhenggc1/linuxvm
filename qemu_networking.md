## qemu networking

#### 资源

1. 主要参考qemu的doc，wiki，以及man手册

#### 步骤

1. 这里主要介绍两种和host网络完全打通的方案，看起来就像局域网的独立主机，其实两种都是基于bridge和tap来处理，不同的是tap设备是人工创建的，还是自动的

2. 先做基础的设置，host网络先设置为bridge，假设目前有一个网络接口为enp4s0，这里网络会断开，如果是远程的ssh，需要注意，会发生链接断开，不能设置的问题，建议还是在主机前处理，另外目前以下命令都是root权限执行的，如果非root权限，在命令前加sudo
```
增加bridge
ip link add br0 type bridge
ip link set br0 up

去掉enp4s0的ip，网络地址需要替换成大家自己的
ip addr del 192.168.1.*/24 dev enp4s0

增加br0的ip，地址也需要替换
ip addr add 192.168.1.*/24 dev br0
ip route add default via 192.168.1.1 dev br0

让br0可以链接enp4s0
ip link set dev enp4s0 master br0
```

3. 第一种模式，自主添加tap接口，这步和2的内容，都可以通过其他的网络配置完成，如systemd-netword等，不需要每次写命令，这里不展开
```
增加tun tap 内核模块
modprobe tun tap

ip tuntap add dev tap0 mode tap
ip link set dev tap0 master br0

ip link set tap0 up

```

4. 这种方式下测试qemu网络
```
sudo qemu-system-i386 -netdev tap,id=tap0,script=no,downscript=no,ifname=tap0 -device virtio-net-pci,netdev=tap0

看到启动没报错，基本就没有问题了，以后想用网络，就带上这段网络的部分

```

5. 第二种模式，使用bridge，这种模式下，3和4步骤就不用做了
```

添加文件 /etc/qemu/bridge.conf，添加一行内容
allow br0
这个是说允许qemu使用br0，没有这行，会报access denied by acl file，并且给权限给到了qemu-bridge-helper，不用root也可以创建tap接口

```

6. 然后就可以直接玩了，这里可以用一般用户权限来执行，不需要root权限
```
qemu-system-i386 -netdev bridge,id=n1,helper=/usr/lib/qemu/qemu-bridge-helper -device virtio-net-pci,netdev=n1

这个启动的时候可以在host上执行ip a看看，会自动增加一个tap的接口
没啥报错也ok了

helper的位置如果在PATH下，就不需要指定了，archlinux放在非PATH下，所以要指定下，当然也可以cp下

```

7. 其它
```
在你的局域网中要有dhcp服务器，这样在guest上就可以直接通过dhcp client获取ip地址
如果你会启动多个vm，需要在device后加mac=52:54:00:12:34:56
-device virtio-net-pci,netdev=tap0,mac=52:54:00:12:34:56
动下最后几位，不要重复就可以
另外如果你要给虚拟机多加网卡，就重复上面的netdev和device，然后换id和mac地址就可以
以上三点，上述两种方式都适用的，当然如果是手工创建tap接口的，也需要手工多创建几个，综合来讲，还是选择第二种比较方便。
```
