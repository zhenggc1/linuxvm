## minimum linux

#### 资源

1. 无需下载，从slackware 1.01制作
1. [镜像下载](fda.img.tar.bz2)

#### 步骤

1. 从宿主机创建一个软盘镜像
```
dd if=/dev/zero of=fda.img bs=1k count=1440
```

2. 用slackware 1.01的虚拟机，来处理这个软盘镜像，还在slackware的目录里操作
```
qemu-system-i386 -hda sw101.qcow2 -fda fda.img -boot c
```

3. 启动好后，在slackware1.01里root登录后操作，格式化软盘
```
mke2fs /dev/fd0

这里可以用minix 的文件系统，这个在启动的时候少一步在mount root的时候要回车确定

mkfs /dev/fd0 1440
```

4. 加载磁盘，开始添加文件
```
mount -t ext2 /dev/fd0 /mnt

如果用minix文件系统，-t 带 minix

cd /mnt
```

5. 后面都在/mnt目录里操作，先创建必要的目录，和拷贝必要的文件
```
mkdir bin lib etc usr proc mnt
这里还有很多可以删除的，不过dev文件小，就放着吧
cp -R /dev .

cp /zImage .
不想创建link文件，直接拷贝成目标
cp /lib/libc.so.4.4.1 lib/libc.so.4
```

6. 拷贝bin文件
```
其实这里关键只有sh，追求最小，可以只有sh和mount，其他几个只是为了稍微好玩一点
这里不用各种init等，因为在内核中会找/bin/sh的步骤，启动后就直接到shell了
更多命令的大家可以自行添加

cp /bin/bash bin/sh
cp /bin/mount bin
cp /bin/cat bin
cp /bin/ls bin
cp /bin/cp bin
```


7. 处理启动，lilo
```
在etc下建立一个文件夹lilo，这个目录可以在安装好lilo后删除，如果为了极限的大小
然后拷贝几个文件过来
cp /etc/lilo/boot.b etc/lilo
cp /etc/lilo/map etc/lilo

创建etc/lilo/config，内容如下
boot = /dev/fd0
install = /etc/lilo/boot.b
compact
image = /zImage
    ramdisk = 1440
    root = /dev/fd0
    vga = normal

然后执行lilo的安装命令，这步骤很危险，可能把你的系统搞得无法启动里，做好备份

/etc/lilo/lilo -r /mnt -m /etc/lilo/map -C /etc/lilo/config

显示Added zImage应该成功了

cd /
umount /mnt

```

8. 启动最小的linux
```
上述操作完后，可以直接关闭slackware1.01
或者用qemu的控制台命令，不然如果做的不对老是要启动slackware再做修改也是很烦
qemu-system-i386 -fda fda.img

启动后，就可以简单的浏览文件目录看看
```

9. sysvint，getty，login，passwd等
```
关键两个文件 rc和fstab

rc如下
#!/bin/sh
/bin/mount -av

设置成可执行文件
chmod u+x rc


fstab如下,如果用minix系统，这里用minix代替ext2
/dev/fd0     /    ext2    defaults
none    /proc    proc    defaults

其他的拷贝，不用啥变更 etc目录下的
getty gettydefs group init inittab login.defs passwd shadow shutdown update

bin下的
login mail

建立usr下的目录
/usr/spool/mail

然后就可以启动了，这里就可以让输入root，目前没有密码

```

10. 其他，大家可以加更多自己可以玩的命令，但是这里的软盘大小只有1440，如果为了添加更多的内容，后面就initrd之类压缩的内容了，这里没法继续展开了
