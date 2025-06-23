no<div align="center">
<img src="assets/README/V2ray-Script.svg">
</div>


# V2ray-Script

+ v2ray-script package for setting up personal bridge-upstream servers
+ Following the problems of setting up servers with Docker and the problems surrounding its installation, we decided to create an easy install script for setting up multi-stage servers to remove the oppressive embargo and... for Debian, Ubuntu operating systems without the need for Docker and filtering problems. And ban, start your Linux servers with 3 command lines
+ The bridge-upstream configuration requires at least one Iranian server and one foreign server, and then your internet will be used at half price, and you will be connected in case of a disruption in the international internet!

## Language
<div align="center">

[![en](https://img.shields.io/badge/Lang-English-blue.svg)](https://github.com/4xmen/v2ray-script/blob/master/README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
[![fa](https://img.shields.io/badge/Lang-Persian-green.svg)](https://github.com/4xmen/v2ray-script/blob/master/README.fa.md)

</div>

## How To Use

To deploy this project run , First install upstream And get upstream server info

### vless - vmess (recommended) 

```shell
wget https://raw.githubusercontent.com/4xmen/v2ray-script/main/vless-upstream-installer.sh
chmod +x vless-upstream-installer.sh
./vless-upstream-installer.sh
```
Then install bridge server :
```shell
wget https://raw.githubusercontent.com/4xmen/v2ray-script/main/vmess-vless-bridge-installer.sh
chmod +x vmess-vless-bridge-installer.sh
./vmess-vless-bridge-installer.sh
```

### vmess - vmess
```shell
wget https://raw.githubusercontent.com/4xmen/v2ray-script/main/vmess-upstream-installer.sh
chmod +x vmess-upstream-installer.sh
./vmess-upstream-installer.sh
```
Then install bridge server :

```shell
wget https://raw.githubusercontent.com/4xmen/v2ray-script/main/vmess-bridge-installer.sh
chmod +x vmess-bridge-installer.sh
./vmess-bridge-installer.sh
```


## Screenshots

<div align="center">
<img src="assets/README/v2ray.png" width="600px" >
</div>

## Badges

<div align="center">

[![Github](https://img.shields.io/badge/V2ray-Script-black.svg)](https://github.com/4xmen/v2ray-script) &nbsp;&nbsp;&nbsp;
[![GPL License](https://img.shields.io/badge/License-GPL-green.svg)](https://choosealicense.com/licenses/GPL/) &nbsp;&nbsp;&nbsp;
[![Github](https://img.shields.io/badge/Github-4xmen-blue.svg)](https://Github.com/4xmen) &nbsp;&nbsp;&nbsp;

</div>

## 🔗 Links

https://Github.com/4xmen
<br>
https://Xstack.ir

## Features

- Better Config
- Support Other Linux Distros


## License

 [![GPLv3 License](https://img.shields.io/badge/License-GPL%20v3-yellow.svg)](https://opensource.org/licenses/)
<br>
 [GPL](https://www.gnu.org/licenses/gpl-3.0.en.html)    


## Report Some Bugs
Find a Bug? Please, [create an issue](https://github.com/4xmen/v2ray-script/issues) and we'll fix it together for a better template.

## Support

Share Our Project For Support Us : )

<hr>

<div align="center"> Developed With Love For Free World ! ❤️</div>


function isRoot() {
  if [ "$EUID" -ne 0 ]; then
    echo "Sorry, you need to run this as root"
    return 1
  fi
}

function tunAvailable() {
  if [ ! -e /dev/net/tun ]; then
    return 1
  fi
}
function initialCheck() {
  if ! isRoot; then
    echo "Sorry, you need to run this as root"
    exit 1
  fi
  if ! tunAvailable; then
    echo "TUN is not available"
    exit 1
  fi
}

function updateOs() {
  apt-get update
  apt-get dist-upgrade -y
}

function installRequriment() {
  apt-get install wget curl certbot git zsh unzip -y
}

function downloadV2() {
  mkdir -p /opt/vless
  mkdir -p /var/log/v2ray/
  cd /opt/vless
  FILE=/opt/vless/v2ray
  if [ -f "$FILE" ]; then
    echo "Could not download any more... :)"
  else
    wget https://github.com/v2ray/v2ray-core/releases/download/v4.28.2/v2ray-linux-64.zip
    unzip -o v2ray-linux-64.zip
    rm v2ray-linux-64.zip
  fi
}

function choosePort() {
  echo ""
  echo "What port do you want v2ray (vless) to listen to?"
  echo "   1) Default: 1935 (RTMP)"
  echo "   2) Custom"
  echo "   3) Random [49152-65535]"
  until [[ $PORT_CHOICE =~ ^[1-3]$ ]]; do
    read -rp "Port choice [1-3]: " -e -i 1 PORT_CHOICE
  done
  case $PORT_CHOICE in
  1)
    VPORT="1935"
    ;;
  2)
    until [[ $VPORT =~ ^[0-9]+$ ]] && [ "$VPORT" -ge 1 ] && [ "$VPORT" -le 65535 ]; do
      read -rp "Custom port [1-65535]: " -e -i 1935 VPORT
    done
    ;;
  3)
    # Generate random number within private ports range
    VPORT=$(shuf -i49152-65535 -n1)
    echo "Random Port: $VPORT"
    ;;
  esac
}

function makeConfig() {
  rm /opt/vless/server.json
  VPASS="991832cc-"$(shuf -i1000-9999 -n1)"-"$(shuf -i1000-9999 -n1)"-"$(shuf -i1000-9999 -n1)"-7731d533fffe"
  {
    echo '{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [


    {
      "listen": "0.0.0.0",
      "port": '$VPORT',
      "protocol": "vless",
       "settings": {
        "decryption": "none",
        "clients": [
          {
            "id": "'$VPASS'",
             "level": 0
          }
        ]
      },

      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/"
        }
      },
      "mux": {
        "enabled": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag": "freedom"
    }
  ],
  "dns": {
    "servers": [
      "8.8.8.8",
      "8.8.4.4",
      "localhost"
    ]
  }
}'
  } >>/opt/vless/server.json
}

function makeService() {
  rm /usr/bin/vless.sh
  {

    echo '#!/bin/bash
/opt/vless/v2ray -c=/opt/vless/server.json'

  } >>/usr/bin/vless.sh
  chmod +x /usr/bin/vless.sh
  rm /etc/systemd/system/vless.service
  {

    echo '[Unit]
Description=vless Peoxy
Documentation=

[Service]
Type=simple
User=root
Group=root
TimeoutStartSec=0
Restart=on-failure
RestartSec=30s
#ExecStartPre=
ExecStart=/usr/bin/vless.sh
SyslogIdentifier=Diskutilization
#ExecStop=

[Install]
WantedBy=multi-user.target'

  } >>/etc/systemd/system/vless.service

  systemctl enable vless.service
  echo 'Service created'
  systemctl restart vless.service
  echo 'Service started'
  echo 'Check service:  systemctl status vless.service'

}
function getIP() {
  VIP="$(curl https://api.ipify.org)"
}
initialCheck
updateOs
installRequriment
downloadV2
getIP
cd /opt/vless
choosePort
makeConfig
makeService

echo 'Your server secret key: '$VPASS
echo 'Your IP: '$VIP
echo 'Your port: '$VPORT
echo 'N-joy :) - xStack|4xmen Team'