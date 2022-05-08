## Nginx

nginx部署

依赖包

```sh
yum install -y gcc gcc-c++ autoconf automake openssl openssl-devel pcre pcre-devel bzip2 lsof zlib-devel  libtool libtool-* git unzip
```

编译

```sh
wget http://nginx.org/download/nginx-1.14.2.tar.gz
./configure --prefix=/usr/local/nginx --user=admin --group=admin    --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_realip_module   --with-http_v2_module --add-module=/srv/ngx_http_geoip2_module-master
```

--with-http_flv_module 流媒体播放支持
ngx_http_geoip2_module 屏蔽某个地区的 IP 访问，或者根据访问来源转向不同的子站实现分流

```sh
wget http://zlib.net/zlib-1.2.11.tar.gz
./configure --prefix=/usr/local/zlib
make
make install
```
 
```sh   
wget https://www.openssl.org/source/openssl-1.0.2j.tar.gz
./configure --prefix=/usr/local/openssl
make
make install
```  

```sh    
git clone --recursive https://github.com/maxmind/libmaxminddb
./configure (不要指定安装路径)
make
make install
```
    
wget https://codeload.github.com/leev/ngx_http_geoip2_module/zip/master

开机自启  

```sh
vim /etc/systemd/system/nginx.service
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/openresty/nginx/logs/nginx.pid
# Nginx will fail to start if /usr/local//nginx/logs/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

----------------------分割线-----------------------

报错

    /usr/local/nginx/sbin/nginx: error while loading shared libraries: libmaxminddb.so.0: cannot open shared object file: No such file or directory

检测

    ldd $(which /usr/local/nginx/sbin/nginx)

        libmaxminddb.so.0 => not found

做软连接

    ln -s /srv/libmaxminddb-1.3.2/src/.libs/libmaxminddb.so.0  /usr/lib64/libmaxminddb.so.0
 
----------------------分割线-----------------------

nginx浏览器浏览文本文件
          
```sh
主配置文件 nginx.conf 添加
  location / {
      root   html;
      charset utf-8,gbk;
      autoindex on;
      autoindex_exact_size off;
      autoindex_localtime on;
      autoindex_format html;
  }  
```
```sh
vim /usr/local/nginx/conf/mime.types  添加
    text/plain                                       log;
```

Using Free Let’s Encrypt SSL/TLS Certificates with NGINX

Download the Let’s Encrypt Client

```
yum -y install update  certbot python-certbot-nginx
```

Set Up NGINX

Specify your domain name (and variants, if any) with the server_name directive:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name example.com www.example.com;
}
```

Obtain the SSL/TLS Certificate

```
certbot --nginx -d example.com -d www.example.com
```
