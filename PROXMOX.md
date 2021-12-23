# proxmox

all about proxmox installation and configuration

## Index

1. [installation](#installation)  
2. [configuration (console)](#configuration)  
  2.1 [first steps](#first_steps)  
  2.2 [install usefull tools](#tools)  
  2.3 [setup ntp server](#ntp)  
  2.4 [setup dns server](#dns)  
  2.5 [remove subscription notice](#subscription_notice)   

4. [usage](#usage)  
  4.1 [browse](#browse)  

\# [Find Me](#findme)  
\# [License](#license)  


# 1. installation <a name="installation"></a>  
**see official documentation on [pve.proxmox.com](https://pve.proxmox.com/pve-docs/pve-admin-guide.html)**  


# 2. configuration <a name="configuration"></a>  
**login via ssh (port 22) or do the steps nativ with a keyboard**

### 2.1 first steps <a name="first_steps"></a>  
**install pve-no-subscription repository**
```shell
rm -f /etc/apt/sources.list.d/pve-enterprise.list
wget https://raw.githubusercontent.com/3x3cut0r/proxmox/main/apt/sources.list.d/pve-no-subscription.list -O /etc/apt/sources.list.d/pve-no-subscription.list

```
**update packages**
```shell
apt update && apt upgrade -y

```

### 2.2 install usefull tools <a name="tools"></a>  
**install pve-no-subscription repository**
```shell
apt install htop iperf iperf3 ncdu ntpdate

```

### 2.3 setup ntp server <a name="ntp"></a>  
**change /etc/chrony/chrony.conf**
```shell
...
# Use Debian vendor zone.
# pool 2.debian.pool.ntp.org iburst
server 0.de.pool.ntp.org iburst
server 1.de.pool.ntp.org iburst
server 2.de.pool.ntp.org iburst
...

```
**restart chrony**
```shell
systemctl restart chronyd

```

### 2.4 setup dns server <a name="dns"></a>  
**change /etc/resolv.conf**
```shell

```

### 2.5 remove subscription notice <a name="subscription_notice"></a>  
**remove the notice and restart the proxmox service**
```shell
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

```
**revert the changes if something went wrong**
```shell
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy.service

```

### Find Me <a name="findme"></a>

![E-Mail](https://img.shields.io/badge/E--Mail-executor55%40gmx.de-red)
* [GitHub](https://github.com/3x3cut0r)
* [DockerHub](https://hub.docker.com/u/3x3cut0r)

### License <a name="license"></a>

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) - This project is licensed under the GNU General Public License - see the [gpl-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) for details.
