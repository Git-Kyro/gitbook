## Trojan

docker 启动 trojan

```sh
docker run -itdP -p 8443:8443 -p 880:880 --name trojan --restart=always --privileged jrohy/trojan init

```

nerdctl 启动 trojan

```sh
nerdctl run -d -e TERM=xterm --net=host --name mysql --restart=always -v /root/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD='0000!Abc' mysql:5.7.38 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci 
```

```sh
nerdctl run -d -e TERM=xterm -p 12580:443 -p 880:80 --name trojan --restart=always --privileged \
-v /etc/letsencrypt/archive/jupiterclub.tk/cert1.pem:/etc/letsencrypt/archive/jupiterclub.tk/cert1.pem \
-v /etc/letsencrypt/archive/jupiterclub.tk/privkey1.pem:/etc/letsencrypt/archive/jupiterclub.tk/privkey1.pem \
jrohy/trojan init
```

```sh
Edit config
vi /usr/local/etc/trojan/config.json
    local_port: 443
    remote: 80

7.切换为trojan

restart trojan
```

参考连接

```sh
https://github.com/Jrohy/trojan
```

MacBook clashx Pro 配置

```
# Trojan
- name: "trojan"
  type: trojan
  server: jupiterclub.tk
  port: 18680
  password: "c565e1e2-9c91-4c89-95d1-eeb8214a8f1d"
  skip-cert-verify: true
  # udp: true
  # sni: example.com # 填写伪装域名
  alpn:
     - h2
     - http/1.1
  # skip-cert-verify: true
```