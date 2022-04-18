## Centos upgrade kernel

查看内核版本

```sh
[root@vvuv0394 ~]# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
[root@vvuv0394 ~]# uname -r
3.10.0-327.el7.x86_64
```

升级前升级软件包(非必要)

```sh
yum update -y
```

升级内核

CentOS 6 和CentOS 7的升级方法类似，只不过选择的yum源或者rpm包不同而已，下面仅记录CentOS7升级的详细过程

方法一：添加第三方yum源进行下载安装。

Centos 6 YUM源：http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm

Centos 7 YUM源：http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

先导入elrepo的key，然后安装elrepo的yum源：

```sh
rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```

仓库启用后，你可以使用下面的命令列出可用的内核相关包，如下图：

```sh
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

```sh
[root@02-06-01-k-01 srv]#yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
Loaded plugins: fastestmirror, langpacks
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
Determining fastest mirrors
 * elrepo-kernel: hkg.mirror.rackspace.com
Available Packages
elrepo-release.noarch                   7.0-5.el7.elrepo             elrepo-kernel
kernel-lt.x86_64                        4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-devel.x86_64                  4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-doc.noarch                    4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-headers.x86_64                4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-tools.x86_64                  4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-tools-libs.x86_64             4.4.248-1.el7.elrepo         elrepo-kernel
kernel-lt-tools-libs-devel.x86_64       4.4.248-1.el7.elrepo         elrepo-kernel
kernel-ml-devel.x86_64                  5.10.0-1.el7.elrepo          elrepo-kernel
kernel-ml-doc.noarch                    5.10.0-1.el7.elrepo          elrepo-kernel
kernel-ml-headers.x86_64                5.10.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools.x86_64                  5.10.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools-libs.x86_64             5.10.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools-libs-devel.x86_64       5.10.0-1.el7.elrepo          elrepo-kernel
```

使用命令安装最新稳定内核

```sh
yum -y --enablerepo=elrepo-kernel install kernel-ml
```

方法二：通过下载kernel image的rpm包进行安装。

下载地址：

官方 Centos 6: http://elrepo.org/linux/kernel/el6/x86_64/RPMS/

官方 Centos 7: http://elrepo.org/linux/kernel/el7/x86_64/RPMS/

获取下载链接进行下载安装即可

wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-4.4.185-1.el7.elrepo.x86_64.rpm

rpm -ivh kernel-lt-4.4.185-1.el7.elrepo.x86_64.rpm

4 修改grub中默认的内核版本

内核升级完毕后，目前内核还是默认的版本，如果此时直接执行reboot命令，重启后使用的内核版本还是默认的3.10，

不会使用新的5.2.2，首先，我们可以通过命令查看默认启动顺序

```sh
[root@localhost ~]# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (5.10.0-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (4.4.182-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.21.3.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.10.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-e34fb4f1527b4f2d9fc75b77c016b6e7) 7 (Core)
```

由上面可以看出新内核(4.12.4)目前位置在0，原来的内核(3.10.0)目前位置在1，所以如果想生效最新的内核，

还需要我们修改内核的启动顺序为0：

```sh
vim /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

注：Centos 6 更改的文件相同，使用命令确定新内核位置后，然后将参数default更改为0即可。

接着运行grub2-mkconfig命令来重新创建内核配置，如下：

```sh
[root@02-06-01-k-01 srv]#grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.10.0-1.el7.elrepo.x86_64  <---------
Found initrd image: /boot/initramfs-5.10.0-1.el7.elrepo.x86_64.img <---------
Found linux image: /boot/vmlinuz-3.10.0-1160.49.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1160.49.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1062.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1062.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-fc18f1c6564848e4a67985376955cf2e
Found initrd image: /boot/initramfs-0-rescue-fc18f1c6564848e4a67985376955cf2e.img
done
```

重启系统并查看系统内核
```shell script
reboot
[root@02-06-01-k-01 srv]# uname -r
5.10.0-1.el7.elrepo.x86_64
```
