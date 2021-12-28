# pihole

pihole installation inside a unprivileged LXC-Container

## Index

1. [create LXC-Container](#lxc-container)  
2. [installation](#installation)  
  2.1 [prerequisites](#prerequisites)  
  2.2 [installation](#installation)  
  2.3 [change pihole password](#pihole_password)  
  2.4 [cloudflare DNS over HTTPS](#cloudflare_doh)  
3. [configuration](#configuration)  
  3.1 [add lists](#adlists)  
4. [usage](#usage)  
  4.1 [browse](#browse)  
5. [errors](#errors)  
  5.1 [installation error](#error_installation)  
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
chown -R pihole:pihole /etc/pihole

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

# 3. configuration <a name="configuration"></a>  
**list tables**  
```shell
sudo sqlite3 /etc/pihole/gravity.db .tables

```

### 3.1 add lists <a name="add_lists"></a>  
**adlists from [blocklistproject](https://github.com/blocklistproject/Lists)**  
```shell
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/abuse.txt', 1, 'abuse (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/ads.txt', 1, 'ads (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/basic.txt', 1, 'basic (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/crypto.txt', 1, 'crypto (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/everything.txt', 1, 'everything (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/facebook.txt', 1, 'facebook (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/fraud.txt', 1, 'fraud (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/gambling.txt', 1, 'gambling (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/malware.txt', 1, 'malware (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/phishing.txt', 1, 'phishing (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/piracy.txt', 1, 'piracy (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/porn.txt', 1, 'porn (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/ransomware.txt', 1, 'ransomware (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/redirect.txt', 1, 'redirect (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/scam.txt', 1, 'scam (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/smart-tv.txt', 1, 'smart-tv (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/tiktok.txt', 1, 'tiktok (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/torrent.txt', 1, 'torrent (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/tracking.txt', 1, 'tracking (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/twitter.txt', 1, 'twitter (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/vaping.txt', 1, 'vaping (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/whatsapp.txt', 1, 'whatsapp (blp)');"
sudo sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('https://raw.githubusercontent.com/blocklistproject/Lists/master/youtube.txt', 1, 'youtube (blp)');"
pihole -g

```

# 4. usage <a name="usage"></a>  

### 4.1 browse <a name="browse"></a>  
**Backend**  
[https://pihole.ip/admin](https://pihole.ip/admin)  

# 5. errors <a name="errors"></a>  

### 5.1 installation error <a name="error_installation"></a>  
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
