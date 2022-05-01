## Novnc

web site remote Linux Desktop

如果想 Remote 到 Docker 里面的 Centos, 先用 Docker 启动 Centos

如果想 Romte 到 VM 忽略下面的 Docker 启动，直接安装 VNC 开始执行

```sh
docker pull centos:7.9.2009
docker run -itd --privileged --name centos7.9_vnc -p 5901:5901 -p 6901:6901 centos:7.9.2009 /usr/sbin/init
docker exec -it centos7.9_vnc bash
```

安装 VNC

```sh
curl -L https://gitee.com/panchongwen/my_scripts/raw/main/linux/centos7_vnc_install.sh -o centos7_vnc_install.sh

# 更改 noVNC 版本
# git clone https://github.com.cnpmjs.org/novnc/noVNC.git
# 改为 git clone https://github.com/novnc/noVNC.git -b v1.2.0

# git clone https://github.com.cnpmjs.org/novnc/websockify
# 改为 git clone https://github.com/novnc/websockify
bash ./centos7_vnc_install.sh  # 安装过程需要输入登陆 VNC 的密码
```

检查状态

sh```
systemctl status vncserver@:1.service
systemctl status novnc@:1.service
```

远程登陆: http://127.0.0.1:6901 # 会提示输入安装过程中的登陆 VNC 的密码

设置 vncserver 适配屏幕

```sh
vim /usr/bin/vncserver_wrapper
/usr/sbin/runuser -l "$USER" -c "/usr/bin/vncserver ${INSTANCE} -geometry 1700x1300"
```

设置访问 web 页面登陆密码


```sh
yum -y install httpd-tools
htpasswd -c /etc/htpasswd Kyro
```

通过 nginx 代理 novnc

```sh
[root@instance-2 ~]# cat /etc/nginx/conf.d/novnc.conf
server {
    server_name  novnc.localhost.com;
    root         /usr/share/nginx/html;

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/novnc.localhost.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/novnc.localhost.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    location / {
            auth_basic           "Administrator’s Area";
            auth_basic_user_file /etc/htpasswd;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_pass http://127.0.0.1:6901;
            proxy_buffering off;
            proxy_connect_timeout  3600s;
            proxy_read_timeout  3600s;
            proxy_send_timeout  3600s;
            send_timeout  3600s;
    }
}
server {
    if ($host = novnc.localhost.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
        listen       80;
        listen       [::]:80;
        server_name  novnc.localhost.com;
    return 404; # managed by Certbot
}
```