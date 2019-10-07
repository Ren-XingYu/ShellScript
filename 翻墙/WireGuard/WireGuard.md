# Install

```
add-apt-repository ppa:wireguard/wireguard
add-apt-repository -r ppa:wireguard/wireguard

apt-get update

apt-get install wireguard -y

modprobe wireguard

lsmod | grep wireguard

cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

# Server

```
vim /etc/wireguard/wg0.conf
```

```javascript
[Interface]
Address = 10.0.0.1/32
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o  ens3   -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o  ens3  -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o  ens3  -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o  ens3  -j MASQUERADE
ListenPort = 51820
PrivateKey = MK8eaitDiIsDGQP9PoQ8Wyxox6KATV4AWYVsPuvPbUk=

[Peer]
PublicKey = gNUkyAirx/T6tcnJAmK3iktycVb/pg0UhUiARi7lHhs=
AllowedIPs = 10.0.0.2/32

[Peer]
PublicKey = pm5COfCJVR7KIaLX7JXW/zSrE/XHQqYV9/RwjUNGTQM=
AllowedIPs = 10.0.0.3/32
```

```
wg-quick up wg0
wg-quick down wg0
systemctl enable wg-quick@wg0
wg show
```

# Client

```javascript
[Interface]
PrivateKey = OKJnbKGXUQVgJgkA53e85W729A5YQDc8+w4Mgn9XJE8=
ListenPort = 51820
Address = 10.0.0.2/32
DNS = 114.114.114.114

[Peer]
PublicKey = Imf961W83vCz6MSv2G9kvoVP5y/wMl3fhHcwLm5q02c=
AllowedIPs = 0.0.0.0/0
Endpoint = 144.202.13.110:51820
PersistentKeepalive = 25
```

```javascript
[Interface]
PrivateKey = uIX5y+Bs1R1hZ9yQpwQFTS6X3q/j9Phaq0fqH9kZQ0Y=
ListenPort = 51820
Address = 10.0.0.3/32
DNS = 114.114.114.114

[Peer]
PublicKey = Imf961W83vCz6MSv2G9kvoVP5y/wMl3fhHcwLm5q02c=
AllowedIPs = 0.0.0.0/0
Endpoint = 144.202.13.110:51820
PersistentKeepalive = 25
```

# forward the traffic

```
vim /etc/sysctl.conf

net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1

sysctl -p
```

