# pihole

pihole installation inside a unprivileged LXC-Container

## Index

1. [create LXC-Container](#lxc-container)  
2. [installation](#installation)  
  2.1 [prerequisites](#prerequisites)  
  2.2 [installation](#installation)  
  2.3 [change pihole password](#pihole_password)  
  2.4 [cloudflare DNS over HTTPS](#cloudflare_doh)  
3. [usage](#usage)  
  3.1 [browse](#browse)  
4. [errors](#errors)  
  4.1 [installation error](#error_installation)  
\# [Find Me](#findme)  
\# [License](#license)  

# 1. create LXC-Container <a name="lxc-container"></a>  
**create a LXC-Container:**  
- unprivileged = 1  
- nesting = 1  
- template = debian-11-standard  
- disk-space = 30GiB  
- cpu-cores = 1  
- ram = 512 MiB  
- swap = 0 MiB  
- IPv4 = static (set ip + gw)  
- IPv6 = SLAAC  
- dns-domain = local.lan  
- dns-server = 1.1.1.1  

# 2. installation <a name="installation"></a>  

### 2.1 prerequisites <a name="prerequisites"></a>  
```shell
apt update && apt upgrade -y
apt install curl -y

```

### 2.2 installation <a name="installation"></a>  
```shell
curl -sSL https://install.pi-hole.net | bash

```

### 2.3 change pihole password <a name="pihole_password"></a>  
```shell
pihole -a -p

```

### 2.4 cloudflare DNS over HTTPS <a name="cloudflare_doh"></a>  
```shell
cd /usr/local/bin
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
mv cloudflared-linux-amd64 cloudflared
sudo useradd -s /usr/sbin/nologin -r -M cloudflared
echo -e "# Commandline args for cloudflared" > /etc/default/cloudflared
echo -e "CLOUDFLARED_OPTS=--port 5053 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query" >> /etc/default/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
sudo chown cloudflared:cloudflared /etc/default/cloudflared
sudo chown cloudflared:cloudflared /usr/local/bin/cloudflared
wget https://raw.githubusercontent.com/3x3cut0r/proxmox/main/lxc/pihole/lib/systemd/system/cloudflared.service -O /lib/systemd/system/cloudflared.service
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared

```

**test dns**  
```shell
dig @127.0.0.1 -p 5053 google.com

```

# 3. usage <a name="usage"></a>  

### 3.1 browse <a name="browse"></a>  
**Backend**  
[https://pihole.ip/admin](https://pihole.ip/admin)  

# 4. errors <a name="errors"></a>  

### 4.1 installation error <a name="error_installation"></a>  
**[✗] Unable to build gravity tree in /etc/pihole/gravity.db_temp**  
**Error: no such table: main.gravity**  
```shell
sudo rm /etc/pihole/gravity.db
pihole -r

```

### Find Me <a name="findme"></a>

![E-Mail](https://img.shields.io/badge/E--Mail-executor55%40gmx.de-red)
* [GitHub](https://github.com/3x3cut0r)
* [DockerHub](https://hub.docker.com/u/3x3cut0r)

### License <a name="license"></a>

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) - This project is licensed under the GNU General Public License - see the [gpl-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) for details.
