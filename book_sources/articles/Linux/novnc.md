## Novnc

通过 web 页面远程连接 Linux Desktop

安装 VNC

```sh
curl -L https://gitee.com/panchongwen/my_scripts/raw/main/linux/centos7_vnc_install.sh -o centos7_vnc_install.sh

# 更改 noVNC 版本
# git clone https://github.com.cnpmjs.org/novnc/noVNC.git
# 改为 git clone https://github.com/novnc/noVNC.git -b v1.2.0

# git clone https://github.com.cnpmjs.org/novnc/websockify
# 改为 git clone https://github.com/novnc/websockify
bash ./centos7_vnc_install.sh
```

设置 vncserver 适配屏幕

```sh
vim /usr/bin/vncserver_wrapper
/usr/sbin/runuser -l "$USER" -c "/usr/bin/vncserver ${INSTANCE} -geometry 1700x1300"
```

设置访问 web 页面登陆密码
http://127.0.0.1:6901

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
    ssl_certificate /etc/letsencrypt/live/novnc.ddnsking.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/novnc.ddnsking.com/privkey.pem; # managed by Certbot
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
    if ($host = novnc.ddnsking.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
        listen       80;
        listen       [::]:80;
        server_name  novnc.ddnsking.com;
    return 404; # managed by Certbot
}
```