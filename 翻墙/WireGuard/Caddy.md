# Install

```
curl https://getcaddy.com | bash -s personal

setcap cap_net_bind_service=+ep /usr/local/bin/caddy

mkdir /etc/caddy
mkdir /etc/ssl/caddy
chmod 0770 /etc/ssl/caddy
touch /etc/caddy/Caddyfile
mkdir /var/www
```

# SystemD Configuration

```
vim /lib/systemd/system/caddy.service
```

```javascript
[Unit]
Description=Caddy HTTP/2 web server
Documentation=https://caddyserver.com/docs
After=network-online.target
Wants=network-online.target

[Service]
Restart=on-failure
StartLimitInterval=86400
StartLimitBurst=5

User=root
Group=root
; Letsencrypt-issued certificates will be written to this directory.
Environment=CADDYPATH=/etc/ssl/caddy

ExecStart=/usr/local/bin/caddy -log stdout -agree=true -conf=/etc/caddy/Caddyfile -root=/var/tmp
ExecReload=/bin/kill -USR1 $MAINPID

LimitNOFILE=1048576
LimitNPROC=64

PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=full
ReadWriteDirectories=/etc/ssl/caddy

; The following additional security directives only work with systemd v229 or later.
; They further retrict privileges that can be gained by caddy. Uncomment if you like.
; Note that you may have to add capabilities required by any plugins in use.
;CapabilityBoundingSet=CAP_NET_BIND_SERVICE
;AmbientCapabilities=CAP_NET_BIND_SERVICE
;NoNewPrivileges=true
[Install]
WantedBy=multi-user.target
```

```
vim /etc/caddy/Caddyfile

ai.renxingyu.ml
{
  log ./caddy.log
  proxy /ws localhost:23456 {
    websocket
    header_upstream -Origin
  }
}
```

```
systemctl enable caddy.service
systemctl start caddy.service
systemctl status caddy.service
```

