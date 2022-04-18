## Centos

修改主机名

```sh
hostnamectl --static set-hostname CentOS-name   
```

主机名颜色设置 /root/.bashrc 添加

```sh
export PS1="\[\e[36;1m\][\u@\h \W]#\[\e[0m\]"
```

date 24小时显示设置

```sh
timedatectl set-timezone Asia/Shanghai
```

搜索10天之前的log日志,并删除

```sh
find ./ -mtime +10 -name "*.log*" -exec rm -f {} \;
```

磁盘清理

```sh
/var/spool/clientmqueue 是邮件暂存区  
/etc/init.d/sendmail start把sendmail服务启起来
 du -h --max-depth=1 查看单个目录大小
 -bash: /bin/rm: Argument list too long
 ls|xargs rm -f
 ```

 查看docker磁盘使用信息
 
 ```sh
 docker system df
 ```

 用于清理磁盘，删除关闭的容器、无用的数据卷和网络
 
 ```sh
 docker system prune
 ```

 清理得更加彻底，可以将没有容器使用 Docker 镜像都删掉
 
 ```sh
 docker system prune -a
```

手工清理,删除所有悬空镜像，但不会删除未使用镜像

```sh
docker rmi $(docker images -f "dangling=true" -q)
```

删除所有未使用镜像和悬空镜像。

```sh
docker rmi $(docker images-q)
```

删除所有未被任何容器关联引用的卷：

```sh
docker volume rm $(docker volume ls -qf dangling=true)
docker volume rm $(docker volume ls -q)
```

删除所有已退出的容器

```sh
docker rm -v $(docker ps -aq -f status=exited)
```

删除所有状态为 dead 的容器

```sh
docker rm -v $(docker ps -aq -f status=dead)
```

/etc/sysctl.conf

```sh
net.ipv4.ip_forward=1
fs.nr_open=100000000
fs.file-max = 1048576
net.ipv4.tcp_max_tw_buckets = 180000
net.ipv4.tcp_retries2 = 5
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
```