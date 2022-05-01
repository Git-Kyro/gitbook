## Trojan

docker 启动 trojan

```sh
docker run -itdP -p 8443:8443 -p 880:880 --name trojan --restart=always --privileged jrohy/trojan init
```

```sh
Edit config
vi /usr/local/etc/trojan/config.json
    local_port: 8443
    remote: 880

restart trojan
```

参考连接

```sh
https://github.com/Jrohy/trojan
```