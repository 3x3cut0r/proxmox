# pihole

pihole installation inside a unprivileged LXC-Container

## Index

1. [create LXC-Container](#lxc-container)  
2. [installation](#installation)  
  2.1 [prerequisites](#prerequisites)  
  2.2 [installation](#installation)  
3. [usage](#usage)  
  3.1 [browse](#browse)  
4. [errors](#errors)  
  4.1 [installation error](#error_installation)  
\# [Find Me](#findme)  
\# [License](#license)  

# 1. create LXC-Container <a name="lxc-container"></a>  
**create a LXC-Container:**  
- unprivileged = 1  
**- nesting = 1**  
**- template = debian-11-standard**  
**- disk-space = 30GiB**  
**- cpu-cores = 1**  
**- ram = 512 MiB**  
**- swap = 0 MiB**  
**- IPv4 = static (set ip + gw)**  
**- IPv6 = SLAAC**  
**- dns-domain = local.lan**  
**- dns-server = 1.1.1.1**  

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
