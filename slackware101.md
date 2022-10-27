## slackware 1.01

#### 资源

1. [安装包下载地址](https://mirrors.slackware.com/slackware/slackware-1.01/)
1. [镜像文件下载地址](https://app.box.com/s/y4qcrcksn2j10q606z5c02sohydpv4ro)

#### 步骤

1. 创建一个空目录，就叫slackware101，然后在这个目录中创建磁盘
```
cd slackware101

创建160M的虚拟硬盘
qemu-img create sw101.qcow2 160M

```

2. 下载链接所有文件，按照原有目录放在这个目录下，叫作slackware-1.01

3. 拷贝启动软盘镜像到本目录，这里用到了下载目录中a1目录中的a1disk
```
cp slackware-1.01/a1/a1disk .

```

4. 启动虚拟机，使用qemu的命令，这里就可以进入一个单软盘的系统，供安装使用
```
测试启动盘
qemu-system-i386 -fda a1disk -hda sw101.qcow2 -boot a
```

5. 启动虚拟机对硬盘分区和格式化，不要用你的机器的fdisk做分区，老版本不认识新版本的一些设置，但是新版本兼容了老版本
```
在虚拟机内部操作
输入root登录后

做分区，需要很多细节步骤，自己看说明，m是help，p是看目前的分区情况
fdisk /dev/hda
可以平均分成两个分区，都是linux的，通过fdisk里的t修改成分区标记是83，默认是81

然后分别格式化两个分区,帮助里写着要加-c参数，但是加了后不通过，没有细看为啥
用两个分区是因为有一个分区用来放安装文件，不要一个一个做软盘这么麻烦，并且这里看了安装的shell，必须要分开两个分区，因为会分别mount
mke2fs /dev/hda1
mke2fs /dev/hda2
这里后就可以关闭虚拟机先，在宿主机上拷贝文件
```

6. 在宿主机上，拷贝安装内容到磁盘中，可以节省切换软盘的过程，比较方便
```
加载loop，可以类似磁盘一样操作文件
sudo losetup -Pf sw101.qcow2
创建mount点，临时的
mkdir tmp
mount磁盘文件的第二个分区，因为想安装在第一个分区里
sudo mount /dev/loop0p2 tmp
拷贝内容到磁盘,必须是install目录，安装脚本里写死了
sudo cp -r slackware-1.01 tmp/install

到这个目录里，从a2到a13把里面的DISKA2类似的文件全部修改成小写，不知道为啥，但是看了安装脚本是都是小写的，否则安装不了。
cd tmp/install/a2
mv DISKA2 diska2

做完后，回到slackware101目录

取消mount
sudo unmount tmp
取消loop加载
sudo losetup -D
```

7. 启动开始安装
```
qemu-system-i386 -fda a1disk -hda sw101.qcow2 -boot a

输入root登录

doinstall /dev/hda1

选择2从磁盘安装，然后输入/dev/hda2，然后输入分区类型是ext2

然后选择安装的内容，选择1，最小安装

然后会问你很多是否需要的包，按照自己的想法来选择，简单的都可以先选择n

然后会来到一个做启动盘的地方，让你做一个启动盘，只能选是，看里脚本里好多注释掉了
这个时候就是把你宿主机的a1disk安装成启动软盘，就直接用吧
以后如果要用的时候，重新从下载目录里拷贝一个出来就好了

然后选择安装lilo，选择2，测试系统就只有linux，2最简单里

然后是安装鼠标和调制解调器，都选n
```

8. 重新启动，并且只用硬盘启动
```
暴力一点直接关闭虚拟机，温柔一点就用reboot命令，等到里重启界面再关闭虚拟机，然后用以下命令启动

qemu-system-i386 -hda sw101.qcow2

```

