# V2Ray架构

![](https://cos-1257663582.cos.ap-chengdu.myqcloud.com/Notes/V2Ray/arch.svg)

# 工作机制

## 单服务器模式

![](https://cos-1257663582.cos.ap-chengdu.myqcloud.com/Notes/V2Ray/single_server_mode.png)

## 桥接模式

![](https://cos-1257663582.cos.ap-chengdu.myqcloud.com/Notes/V2Ray/bridge_mode.png)

## 工作原理

![](https://cos-1257663582.cos.ap-chengdu.myqcloud.com/Notes/V2Ray/working_principle.png)

```
{浏览器} <--(socks)--> {V2Ray 客户端 inbound <-> V2Ray 客户端 outbound} <--(VMess)-->  {V2Ray 服务器 inbound <-> V2Ray 服务器 outbound} <--(Freedom)--> {目标网站}
```

# Linux安装

```
bash <(curl -L -s https://install.direct/go.sh)
```

此脚本会自动安装以下文件：

- `/usr/bin/v2ray/v2ray`：V2Ray 程序；
- `/usr/bin/v2ray/v2ctl`：V2Ray 工具；
- `/etc/v2ray/config.json`：配置文件；
- `/usr/bin/v2ray/geoip.dat`：IP 数据文件
- `/usr/bin/v2ray/geosite.dat`：域名数据文件

- `/etc/systemd/system/v2ray.service`: Systemd
- `/etc/init.d/v2ray`: SysV

# 命令行参数

```
v2ray [-version] [-test] [-config=config.json] [-format=json]
```

```bash
v2ctl <command> <options>
```

> ```
> command
> ```

子命令，有以下选项:

- `api`: 调用 V2Ray 进程的远程控制指令。
- `config`: 从标准输入读取 JSON 格式的配置，然后从标准输出打印 Protobuf 格式的配置。
- `cert`: 生成 TLS 证书。
- `fetch`: 抓取远程文件。
- `tlsping`: (V2Ray 4.17+) 尝试进行 TLS 握手。
- `verify`: 验证文件是否由 Project V 官方签名。
- `uuid`: 输出一个随机的 UUID。

# 配置文件格式

```
{
    //日志配置，表示 V2Ray 如何输出日志。
    "log": {
        "access": "文件地址", //访问日志的文件地址
        "error": "文件地址", //错误日志的文件地址
        "loglevel": "warning"
    },
    //内置的远程控置 API, V2Ray 中可以开放一些 API 以便远程调用。这些 API 都基于 gRPC。
    "api": {},
    //内置的 DNS 服务器，若此项不存在，则默认使用本机的 DNS 设置。
    "dns": {},
    //当此项存在时，开启统计信息。
    "stats": {},
    //V2Ray 内建了一个简单的路由功能，可以将入站数据按需求由不同的出站连接发出，以达到按需代理的目的。这一功能的常见用法是分流国内外流量，V2Ray 可以通过内部机制判断不同地区的流量，然后将它们发送到不同的出站代理。
    "routing": {
        "domainStrategy": "AsIs(default)|IPIfNonMatch|IPOnDemand",
        "rules": [
            {
                "type": "field",
                "domain": [
                    "baidu.com",
                    "qq.com",
                    "geosite:cn"
                ],
                "ip": [
                    "0.0.0.0/8",
                    "10.0.0.0/8",
                    "fc00::/7",
                    "fe80::/10",
                    "geoip:cn"
                ],
                "port": "53,443,1000-2000",
                "network": "tcp|udp|tcp,udp", //当连接方式是指定的方式时，此规则生效。
                "source": [
                    "10.0.0.1"
                ],
                "user": [
                    "love@v2ray.com"
                ],
                "inboundTag": [ //当某一元素匹配入站协议的标识时，此规则生效。
                    "tag-vmess"
                ],
                "protocol": [
                    "http",
                    "tls",
                    "bittorrent"
                ], //一个数组，数组内每一个元素表示一种协议。当某一个协议匹配当前连接的流量时，此规则生效。必须开启入站代理中的sniffing选项。
                "attrs": "attrs[':method'] == 'GET'", // 一段脚本，用于检测流量的属性值。当此脚本返回真值时，此规则生效。
                "outboundTag": "direct", //对应一个额外出站连接配置的标识。
                "balancerTag": "balancer" //对应一个负载均衡器的标识。balancerTag和outboundTag须二选一。当同时指定时，outboundTag生效。
            }
        ],
        "balancers": [
            {
                "tag": "balancer",
                "selector": []
            }
        ]
    },
    //本地策略可进行一些权限相关的配置,本地策略可以配置一些用户相关的权限，比如连接超时设置。V2Ray 处理的每一个连接，都对应到一个用户，按照这个用户的等级（level）应用不同的策略。本地策略可按照等级的不同而变化。
    "policy": {
        "levels": {
            "0": {
                "handshake": 4,
                "connIdle": 300,
                "uplinkOnly": 2,
                "downlinkOnly": 5,
                "statsUserUplink": false,
                "statsUserDownlink": false,
                "bufferSize": 10240
            }
        },
        "system": {
            "statsInboundUplink": false,
            "statsInboundDownlink": false
        }
    },
    //反向代理,可以把服务器端的流量向客户端转发，即逆向流量转发。
    "reverse": {
        "bridges": [
            {
                "tag": "bridge",
                "domain": "test.v2ray.com"
            }
        ],
        "portals": [
            {
                "tag": "portal",
                "domain": "test.v2ray.com"
            }
        ]
    },
    "inbounds": [ //一个数组，每个元素是一个入站连接配置。
        {
            "port": 1080,
            "listen": "127.0.0.1",
            "protocol": "协议名称",
            "settings": { //具体的配置内容，视协议不同而不同。详见每个协议中的InboundConfigurationObject。
            },
            "streamSettings": { //底层传输方式（transport）是当前 V2Ray 节点和其它节点对接的方式。底层传输方式提供了稳定的数据传输通道。底层传输（transport）配置分为两部分，一是全局设置(TransportObject)，二是分协议配置(StreamSettingsObject)。
                "network": "tcp(default)|kcp|ws|http|domainsocket|quic",
                "security": "none(default)|tls",
                "tlsSettings": {},
                "tcpSettings": {},
                "kcpSettings": {},
                "wsSettings": {},
                "httpSettings": {},
                "dsSettings": {},
                "quicSettings": {},
                "sockopt": {
                    "mark": 0,
                    "tcpFastOpen": false,
                    "tproxy": "off"
                }
            },
            "tag": "标识",
            "sniffing": {
                "enabled": false,
                "destOverride": [
                    "http",
                    "tls"
                ]
            },
            "allocate": {
                "strategy": "always",
                "refresh": 5,
                "concurrency": 3
            }
        }
    ],
    "outbounds": [ //一个数组，每个元素是一个出站连接配置。列表中的第一个元素作为主出站协议。当路由匹配不存在或没有匹配成功时，流量由主出站协议发出。
        {
            "sendThrough": "0.0.0.0",
            "protocol": "协议名称",
            "settings": {},
            "tag": "标识",
            "streamSettings": {},
            "proxySettings": {
                "tag": "another-outbound-tag"
            },
            "mux": { //Mux 功能是在一条 TCP 连接上分发多个 TCP 连接的数据。Mux 只需要在客户端启用，服务器端自动适配。
                "enabled": false,
                "concurrency": 8
            }
        }
    ],
    "transport": {
        "tcpSettings": {},
        "kcpSettings": {},
        "wsSettings": {},
        "httpSettings": {},
        "dsSettings": {},
        "quicSettings": {}
    }
}
```

# 协议列表

## Blackhole

- 名称: `blackhole`
- 类型: 出站协议

Blackhole（黑洞）是一个出站数据协议，它会阻碍所有数据的出站，配合路由（Routing）]一起使用，可以达到禁止访问某些网站的效果。

```javascript
{
  "response": {
    "type": "none|http"
  }
}
```

## DNS

- 名称: `dns`
- 类型: 出站协议

DNS 是一个出站协议，主要用于拦截和转发 DNS 查询。此出站协议只能接收 DNS 流量（包含基于 UDP 和 TCP 协议的查询），其它类型的流量会导致错误。

在处理 DNS 查询时，此出站协议会将 IP 查询（即 A 和 AAAA）转发给内置的 DNS 服务器。其它类型的查询流量将被转发至它们原本的目标地址。

```javascript
{
    "network": "tcp|udp",
    "address": "1.1.1.1",
    "port": 53
}
```

## Dokodemo-door

- 名称: `dokodemo-door`
- 类型: 入站协议

Dokodemo door（任意门）是一个入站数据协议，它可以监听一个本地端口，并把所有进入此端口的数据发送至指定服务器的一个端口，从而达到端口映射的效果。

```javascript
{
  "address": "8.8.8.8",
  "port": 53,
  "network": "tcp|udp|tcp,udp",
  "timeout": 0,
  "followRedirect": false,
  "userLevel": 0
}
```

## Freedom

- 名称：`freedom`
- 类型：出站协议

Freedom 是一个出站协议，可以用来向任意网络发送（正常的） TCP 或 UDP 数据。

```javascript
{
  "domainStrategy": "AsIs|UseIP|UseIPv4|UseIPv6",
  "redirect": "127.0.0.1:3366",
  "userLevel": 0
}
```

## HTTP

- 名称：`http`
- 类型：入站协议

HTTP 是一个入站数据协议，兼容 HTTP 1.x 代理。

```javascript
{
  "timeout": 0,
  "accounts": [
    {
      "user": "my-username",
      "pass": "my-password"
    }
  ],
  "allowTransparent": false,
  "userLevel": 0
}
```


在 Linux 中使用以下环境变量即可在当前 session 使用全局 HTTP 代理（很多软件都支持这一设置，也有不支持的）。

- `export http_proxy=http://127.0.0.1:8080/` (地址须改成你配置的 HTTP 入站代理地址)
- `export https_proxy=$http_proxy`

## MTProto

- 名称: `mtproto`
- 类型: 入站 / 出站

MTProto 是一个 Telegram 专用的代理协议。在 V2Ray 中可使用一组入站出站代理来完成 Telegram 数据的代理任务。

目前只支持转发到 Telegram 的 IPv4 地址。

```javascript
{
  "users": [{
    "email": "love@v2ray.com",
    "level": 0,
    "secret": "b0cbcef5a486d9636472ac27f8e11a9d"
  }]
}
```

## Shadowsocks

- 名称：`shadowsocks`
- 类型：入站 / 出站

Shadowsocks 协议，包含入站和出站两部分，兼容大部分其它版本的实现。

与官方版本的兼容性：

- 支持 TCP 和 UDP 数据包转发，其中 UDP 可选择性关闭；
- 支持OTA;
  - 客户端可选开启或关闭；
  - 服务器端可强制开启、关闭或自适应；
- 加密方式（其中AEAD加密方式在 V2Ray 3.0 中加入）：
  - aes-256-cfb
  - aes-128-cfb
  - chacha20
  - chacha20-ietf
  - aes-256-gcm
  - aes-128-gcm
  - chacha20-poly1305 或称 chacha20-ietf-poly1305
- 插件：
  - 通过 Standalone 模式支持 obfs

Shadowsocks 的配置分为两部分，`InboundConfigurationObject`和`OutboundConfigurationObject`，分别对应入站和出站协议配置中的`settings`项。

### InboundConfigurationObject

```javascript
{
  "email": "love@v2ray.com",
  "method": "aes-128-cfb",
  "password": "密码",
  "level": 0,
  "ota": true,
  "network": "tcp"
}
```

### OutboundConfigurationObject

```javascript
{
  "servers": [
    {
      "email": "love@v2ray.com",
      "address": "127.0.0.1",
      "port": 1234,
      "method": "加密方式",
      "password": "密码",
      "ota": false,
      "level": 0
    }
  ]
}
```

## Socks

- 名称：`socks`
- 类型：入站 / 出站

标准 Socks 协议实现，兼容 Socks 4、Socks 4a 和 Socks 5。

Socks 的配置分为两部分，`InboundConfigurationObject`和`OutboundConfigurationObject`，分别对应入站和出站协议配置中的`settings`项。

### InboundConfigurationObject

```javascript
{
  "auth": "noauth(default)|password",
  "accounts": [
    {
      "user": "my-username",
      "pass": "my-password"
    }
  ],
  "udp": false,
  "ip": "127.0.0.1",
  "userLevel": 0
}
```

### OutboundConfigurationObject

```javascript
{
  "servers": [{
    "address": "127.0.0.1",
    "port": 1234,
    "users": [
      {
        "user": "test user",
        "pass": "test pass",
        "level": 0
      }
    ]
  }]
}
```

## VMess

- 名称：`vmess`
- 类型：入站 / 出站

**VMess 是一个基于 TCP 的协议，所有数据使用 TCP 传输。**

VMess 是一个加密传输协议，它分为入站和出站两部分，通常作为 V2Ray 客户端和服务器之间的桥梁。

VMess 依赖于系统时间，请确保使用 V2Ray 的系统 UTC 时间误差在 90 秒之内，时区无关。在 Linux 系统中可以安装`ntp`服务来自动同步系统时间。

VMess 的配置分为两部分，`InboundConfigurationObject`和`OutboundConfigurationObject`，分别对应入站和出站协议配置中的`settings`项。

### InboundConfigurationObject

```javascript
{
  "clients": [
    {
      "id": "27848739-7e62-4138-9fd3-098a63964b6b",
      "level": 0,
      "alterId": 4,
      "email": "love@v2ray.com"
    }
  ],
  "default": {
    "level": 0,
    "alterId": 4
  },
  "detour": {
    "to": "tag_to_detour"
  },
  "disableInsecureEncryption": false
}
```

### OutboundConfigurationObject

```javascript
{
  "vnext": [
    {
      "address": "127.0.0.1",
      "port": 37192,
      "users": [
        {
          "id": "27848739-7e62-4138-9fd3-098a63964b6b",
          "alterId": 4,
          "security": "auto",
          "level": 0
        }
      ]
    }
  ]
}
```

# 传输配置

## TCP

```javascript
{
  "header": {
    "type": "none"
  }
}
```

## mKCP 

**mKCP 是一个基于 UDP 的协议，所有通讯使用 UDP 传输。**

mKCP 使用 UDP 来模拟 TCP 连接，请确定主机上的防火墙配置正确。mKCP 牺牲带宽来降低延迟。传输同样的内容，mKCP 一般比 TCP 消耗更多的流量。

```javascript
{
  "mtu": 1350,
  "tti": 20,
  "uplinkCapacity": 5,
  "downlinkCapacity": 20,
  "congestion": false,
  "readBufferSize": 1,
  "writeBufferSize": 1,
  "header": {
    "type": "none"
  }
}
```

## WebSocket

使用标准的 WebSocket 来传输数据。WebSocket 连接可以被其它 HTTP 服务器（如 NGINX）分流。

Websocket 会识别 HTTP 请求的 X-Forwarded-For 头来用做流量的源地址。

```javascript
{
  "path": "/",
  "headers": {
    "Host": "v2ray.com"
  }
}
```

## HTTP/2

V2Ray 3.17 中加入了基于 HTTP/2 的传输方式。它完整按照 HTTP/2 标准实现，可以通过其它的 HTTP 服务器（如 Nginx）进行中转。

由 HTTP/2 的建议，客户端和服务器必须同时开启 TLS 才可以正常使用这个传输方式。

V2Ray 4.20 中对服务端的TLS配置的强制条件移除，为了在特殊用途的分流部署环境中，由外部网关组件完成TLS层对话，V2Ray作为后端应用，网关和V2Ray间使用称为`h2c`的明文http/2进行通讯。

```javascript
{
  "host": ["v2ray.com"],
  "path": "/random/path"
}
```

## DomainSocket 

Domain Socket 使用标准的 Unix domain socket 来传输数据。它的优势是使用了操作系统内建的传输通道，而不会占用网络缓存。相比起本地环回网络（local loopback）来说，Domain socket 速度略快一些。

目前仅可用于支持 Unix domain socket 的平台，如 macOS 和 Linux。在 Windows 上不可用。

如果指定了 domain socket 作为传输方式，在入站出站代理中配置的端口和 IP 地址将会失效，所有的传输由 domain socket 取代。

```javascript
{
  "path": "/path/to/ds/file"
}
```

## QUIC 

QUIC 全称 Quick UDP Internet Connection，是由 Google 提出的使用 UDP 进行多路并发传输的协议。其主要优势是:

1. 减少了握手的延迟（1-RTT 或 0-RTT）
2. 多路复用，并且没有 TCP 的阻塞问题
3. 连接迁移，（主要是在客户端）当由 Wifi 转移到 4G 时，连接不会被断开。

QUIC 目前处于实验期，使用了正在标准化过程中的 IETF 实现，不能保证与最终版本的兼容性。

```javascript
{
  "security": "none",
  "key": "",
  "header": {
    "type": "none"
  }
}
```