# OPNsense

all about OPNsense installation and configuration

## Index

0. [prerequisites](#prerequisites)  
    0.1 [VM installation (Proxmox)](#01_vm_installation)  
1. [OPNsense defaults](#opnsense_defaults)  
2. [VDSL2 configuration](#vdsl2_configuration)  
    2.1 [modem configuration (DrayTek Vigor 165)](#21_modem_configuration)  
    2.2 [OPNsense configuration (PPPoE)](#22_opnsense_configuration)  
3. [Dynamic DNS (DuckDNS)](#ddns)  

\# [Find Me](#findme)  
\# [License](#license)  


# 0. prerequisites <a name="prerequisites"></a>  

### 0.1 VM installation (Proxmox) <a name="01_vm_installation"></a>  
**see VM installation in the [PROXMOX.md](https://github.com/3x3cut0r/proxmox/blob/main/PROXMOX.md)**  

# 1. OPNsense defaults <a name="opnsense_defaults"></a>
**coming soon**  

# 2. VDSL2 configuration <a name="vdsl2_configuration"></a>

### 2.1 modem configuration (DrayTek Vigor 165) <a name="21_modem_configuration"></a>    
- Configure a LAN IP: (e.g.: 192.168.x.x /24)  
    if you want your modem webinterface accessible from your LAN side,   
    the LAN IP should be in the same subnet.
- **disable DHCP-Server**
- **enable Bridge-Mode**
- **disable VLAN-Tagging for ADSL/VDSL** (this is done by OPNsense)
- optional: disable HTTP-webinterface or enable the redirect to HTTPS
- optional: disable unnecessary services like ftp, telnet, ...

### 2.2 OPNsense configuration (PPPoE) <a name="22_opnsense_configuration"></a>  
**PPPoE Configuration**  
- Interfaces -> Other Types -> VLAN -> create new VLAN:
 - Parent Interface: <Interface, which is directly connected to your modem (re0)>
 - **VLAN Tag: 7**
 - Description: VDSL2
- Interfaces -> Assignments -> WAN: <VLAN interface, which we created before (vlan01)>
 - Save
- Interfaces -> WAN:
 - IPv4 Configuration Type: PPPoE
 - IPv6 Configuration Type: DHCPv6
 - PPPoE configuration:
   - Username: \<your provider login email\> (e.g.: 1und1/ui1234-567@online.de)
   - Password: \<your provider login password\>
   - Dial on Demand: CHECK
   - Idle Timeout: 00 (two "0"!)
 - DHCPv6 client configuration:
    - Request only an IPv6 prefix: CHECK
    - Prefix delegation size: 56
    - Send IPv6 prefix hint: CHECK
    - Use IPv4 connectivity: CHECK

# 2. Dynamic DNS (DuckDNS) <a name="ddns"></a>
- create an account or login into your [DuckDNS](https://www.duckdns.org/) account  
- create a token and add a domain to your account (e.g.: http://3x3cut0r.duckdns.org)  
- copy your token into your clipboard  
- Settings / Firmware / Plugins: install os-ddclient  
- Services / Dynamic DNS / Settings / Add  
 - Service: DuckDNS  
 - Username: <your DuckDNS E-Mail>  
 - Password: <your DuckDNS Token>  
 - Hostname: <your DuckDNS domain (e.g.: http://3x3cut0r.duckdns.org)>  
 - Check ip method: Interface  
 - Force SSL: Check  
 - Interface to monitor: WAN

**if WAN ist not working:**  
- ssh to your OPNsense and open a shell
 - change /usr/local/etc/ddclient.conf to: (if: WAN -> pppoe0)

 ```shell
 daemon=300
 syslog=yes                  # log update msgs to syslog
 pid=/var/run/ddclient.pid   # record PID in file.
 ssl=yes
 ...
 use=if, if=pppoe0, \
 protocol=duckdns, \
 login=<your DuckDNS E-Mail> , \
 password= <your DuckDNS Token> \
 3x3cut0r.duckdns.org

 ```  
- **you need to rechange the ddclient.conf via shell after changing anything via GUI**  

**to prevent ERROR: rebind attack dynamic dns:**  
- System / Settings / Administration / Alternate Hostnames:
 - enter your ddns domains (e.g.: 3x3cut0r.duckdns.org, space-separated list)


### Find Me <a name="findme"></a>

![E-Mail](https://img.shields.io/badge/E--Mail-executor55%40gmx.de-red)
* [GitHub](https://github.com/3x3cut0r)
* [DockerHub](https://hub.docker.com/u/3x3cut0r)

### License <a name="license"></a>

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) - This project is licensed under the GNU General Public License - see the [gpl-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) for details.
