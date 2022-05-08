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

```sh
rpm -ivh kernel-lt-4.4.185-1.el7.elrepo.x86_64.rpm
```

4 查看系统可用内核

```sh
[root@instance-1 ~]#cat /boot/grub2/grub.cfg  | grep menuentry
if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
export menuentry_id_option
menuentry 'CentOS Linux (5.17.3-1.el7.elrepo.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-5.17.3-1.el7.elrepo.x86_64-advanced-1fc4211c-2271-43b7-92f0-3fbdbe1c2f2f' {
menuentry 'CentOS Linux (3.10.0-1160.62.1.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.62.1.el7.x86_64-advanced-1fc4211c-2271-43b7-92f0-3fbdbe1c2f2f' {
menuentry 'CentOS Linux (3.10.0-1160.59.1.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.59.1.el7.x86_64-advanced-1fc4211c-2271-43b7-92f0-3fbdbe1c2f2f' {
menuentry 'CentOS Linux (0-rescue-63dab8ca6840989080052035d6dabdff) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-63dab8ca6840989080052035d6dabdff-advanced-1fc4211c-2271-43b7-92f0-3fbdbe1c2f2f' {
```

5 设置开机从新内核启动

```sh
grub2-set-default 'CentOS Linux (5.17.3-1.el7.elrepo.x86_64) 7 (Core)'
```

6 查看内核启动项

```sh
[root@instance-1 ~]#grub2-editenv list
saved_entry=CentOS Linux (5.17.3-1.el7.elrepo.x86_64) 7 (Core)
```

7 重启系统使内核生效

```sh
[root@instance-1 ~]#reboot
```

8 启动完成确认内核版本是否更新

```sh
[root@instance-1 ~]#uname -r
5.17.3-1.el7.elrepo.x86_64
```




