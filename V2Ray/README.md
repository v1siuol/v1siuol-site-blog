# 科学上网 peace &lt;SSHHHH V2Ray&gt;

> The following article will introduce how to build proxies to bypass network restrictions via V2Ray.

> Author: v1siuol
>
> Last-Modified: 2019.08.30

![v2ray_avatar](https://www.v1siuol.com/files/v2ray_avatar.png)

### Background 

- [IMPORTANT] Read the &lt; Time Flies &gt; post FIRST and follow the instructions before working  on V2Ray. Make sure the UTC time in both clients and servers are consistent, otherwise v2ray might have unexpected behaviors. 

  > V2Ray 对于时间有比较严格的要求，要求服务器和客户端时间差绝对值不能超过 2 分钟，所以一定要保证时间足够准确。但没有强制要求时区一致。

- Config the firewall, security group, etc. properly, or it might block your connection between client and V2Ray server. 


### 0. Install V2Ray via Scripts 

```bash
wget https://install.direct/go.sh
sudo bash go.sh
rm go.sh
```



### 1. Config V2Ray Server

```bash
vim /etc/v2ray/config.json
```

Default config: 

```json
{
  "log" : {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 15579,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "2fec7c9e-de29-4b90-8f02-c26989c29f99",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": ["geoip:private"],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}
```

Before we make a very basic V2Ray config, we need to generate a uuid first. There are a lot of tools out there. 

#### > [ uuid ]

```bash
cat /proc/sys/kernel/random/uuid
```

A randomly generated uuid would output and what you need to do is to make a copy of it. Make sure you need to have the same **id** (in the form of uuid) in VMess inbound and outbound. In other words,  *id* in V2Ray server and V2Ray client must be the same. 



#### > [config.json]

```
sudo chmod 777 /etc/v2ray/config.json
vim /etc/v2ray/config.json
```

In `config.json` :

```json
{
  "log" : {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 16823,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "2fec7c9e-de29-4b90-8f02-c26989c29f99",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  }
}
```

```bash
/usr/bin/v2ray/v2ray -test -config /etc/v2ray/config.json  # do the inspection
```

> V2Ray v3.35 (die Commanderin) 20180809
>
> A unified platform for anti-censorship.
>
> Configuration OK.



### 2. Start V2Ray Server

```bash
sudo systemctl start v2ray
systemctl status v2ray  # Check the status of V2Ray (active / running)
```

Till now, V2Ray server is done! Next, we will build up out V2Ray client side. 



### 3. (optional) More on V2Ray Server 

```bash
sudo systemctl stop v2ray  # stop the service
sudo systemctl restart v2ray  # restart the service
```

If you need to upgrade V2Ray, just re-do the installation scripts again (check [0. Install V2Ray via Scripts](#0. Install V2Ray via Scripts)). Scripts will do everything for you automatically. That's it. 



### 4. Install V2Ray Client

#### > Mac OS X

V2RayX (A simple GUI for V2Ray on macOS): https://github.com/Cenmrev/V2RayX

```bash
brew cask install v2rayx
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Cenmrev/V2RayX/master/compilefromsource.sh)"
```

After installation, you can make your own V2Ray client config. You can place it anywhere as long as you can find it easily. You can later import it. 



##### To Import V2Ray Client Config

```
{
  "log": {
    "loglevel": "warning",
    "access": "/your_path/v2ray_access.log",
    "error": "/your_path/v2ray_error.log"
  },
  "inbound": {
    "port": 1080,
    "protocol": "socks",
    "domainOverride": ["tls","http"],
    "settings": {
      "auth": "noauth",
      "udp": true
    }
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "123.123.123.123",
          "port": 16823,  
          "users": [
            {
              "id": "2fec7c9e-de29-4b90-8f02-c26989c29f99",
              "alterId": 64
            }
          ]
        }
      ]
    }
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "chinasites",
          "outboundTag": "direct"
        },
        {
          "type": "chinaip",
          "outboundTag": "direct"
        }
      ]
    }
  }
}
```



##### To Edit .pac File

你墙：https://github.com/gfwlist/gfwlist

用base64 decode一下就好了哦



##### Load core!

> 富强 民主 文明 和谐
>
> 自由 平等 公正 法治
>
> 爱国 敬业 诚信 友善



####  > iOS

要换成美区的号下载：[**Shadowrocket**](https://apps.apple.com/us/app/shadowrocket/id932747118)

![shadowrocket_app](https://www.v1siuol.com/files/shadowrocket_app.PNG)



不多说啦，我先去收下顺丰快递～



Thank you. 



Reference: 

- https://toutyrater.github.io/
- https://www.v2ray.com/

