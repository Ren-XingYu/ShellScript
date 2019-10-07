# Install

```
apt update
apt install shadowsocks-libev -y
```

# Config

## Server

```
cd /etc/shadowsocks-libev
vim config.json
```

```javascript
{
    "server": "0.0.0.0",
    "mode": "tcp_and_udp",
    "server_port": 8388,
    "password": "my_ss_password",
    "timeout": 60,
    "method": "chacha20-ietf-poly1305"
}
```

```
systemctl start shadowsocks-libev
systemctl restart shadowsocks-libev
systemctl enable shadowsocks-libev
systemctl status shadowsocks-libev
```

## Client

```javascript
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_port":1080,
    "password":"my_ss_password!",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

