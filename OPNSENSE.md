# OPNsense

all about OPNsense installation and configuration

## Index

0. [prerequisites](#prerequisites)  
    0.1 [VM installation (Proxmox)](#01_vm_installation)  
1. [OPNsense defaults](#opnsense_defaults)  
    1.9 [GeoIP blocking](#19_geoip_blocking)
2. [VDSL2 configuration](#vdsl2_configuration)  
    2.1 [modem configuration (DrayTek Vigor 165)](#21_modem_configuration)  
    2.2 [OPNsense configuration (PPPoE)](#22_opnsense_configuration)  
3. [VoIP (FritzBox + 1und1)](#voip)  
    3.1 [OPNsense configuration](#31_opnsense_configuration)  
    3.2 [FritzBox configuration](#32_fritzbox_configuration)  
4. [Dynamic DNS (DuckDNS)](#ddns)  
    4.1 [DuckDNS account](#41_duckdns_account)  
    4.2 [install os-ddclient plugin](#42_install_plugin)  
    4.3 [configure os-ddclient plugin](#43_configure_plugin)  
    4.4 [ddclient via console (optional)](#44_ddclient_console)  
    4.5 [disable rebind attack warning](#45_rebind_attack)  
5. [Lets Encrypt Certificate via ACME-Client (SSL/HTTPS)](#le)  
    5.1 [install os-acme-client plugin](#51_install_plugin)  
    5.2 [configure os-acme-client plugin](#52_configure_plugin)  

\# [Find Me](#findme)  
\# [License](#license)  

# 0. prerequisites <a name="prerequisites"></a>  

### 0.1 VM installation (Proxmox) <a name="01_vm_installation"></a>  
**see VM installation in the [PROXMOX.md](https://github.com/3x3cut0r/proxmox/blob/main/PROXMOX.md)**  

# 1. OPNsense defaults <a name="opnsense_defaults"></a>  

### 1.9 GeoIP blocking <a name="19_geoip_blocking"></a>  
- for GeoIP blocking just follow the instructions [here](https://docs.opnsense.org/manual/how-tos/maxmind_geo_ip.html)  
- create an alias "GeoIP" for all countries you want to block
- create a rule to block this alias "GeoIP" on your WAN interface

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

# 3. VoIP (FritzBox - 1und1) <a name="voip"></a>

### 3.1 OPNsense configuration <a name="31_opnsense_configuration"></a>  
- Firewall / Aliases -> add:  
 - Name: FritzBox  
 - Type: Host(s)  
 - Content: <LAN IP of your FritzBox>  
 - Description: FritzBox  
- Firewall / Aliases -> add:  
 - Name: 1und1_SIP_Server  
 - Type: Host(s)  
 - Content: 212.227.124.129, 212.227.124.130  
 - Description: 1und1 SIP Server  
- Firewall / Aliases -> add:  
 - Name: 1und1_VoIP_Ports  
 - Type: Port(s)  
 - Content: 5060:5061, 7078:7109  
 - Description: 1und1 VoIP Port Range  
- Firewall / NAT / Outbound:  
 - set to Hybrid outbound NAT rule generation  
- Firewall / NAT / Outbound -> add:  
 - Interface: WAN  
 - TCP/IP Version: IPv4  
 - Protocol: UDP  
 - Source address: FritzBox (Alias)  
 - Source port: any  
 - Destination address: any  
 - Destination port: any  
 - Translation / target: WAN address  
 - Static-port: check  
 - Description: VoIP FritzBox  
- Firewall / NAT / Outbound -> add:  
 - Interface: WAN  
 - TCP/IP Version: IPv4  
 - Protocol: UDP  
 - Source address: 1und1_SIP_Server (Alias)  
 - Source port: SIP (default Alias for 5060)  
 - Destination address: This Firewall (default Alias)  
 - Destination port: any  
 - Translation / target: FritzBox (Alias)  
 - Translation / port: 5060 (SIP)  
 - Static-port: uncheck  
 - Description: 1&1 SIP Server to FritzBox7590  
- Firewall / Rules / LAN -> add:  
 - Action: Pass  
 - Quick: check  
 - Interface: LAN  
 - Direction: in  
 - TCP/IP Version: IPv4  
 - Protocol: TCP/UDP  
 - Source: FritzBox (Alias)  
 - Source port range: 1und1_VoIP_Ports (Alias) <-> 1und1_VoIP_Ports (Alias)  
 - Destination: any  
 - Destination port range: any <-> any  
 - Description: VoIP FritzBox  
- Firewall -> Settings -> Advanced:
 - Miscellaneous / Firewall Optimization: conservative  

### 3.3 FritzBox configuration <a name="32_fritzbox_configuration"></a>  
- Telephony / Telephony Numbers -> add (optional):  
 - Registrar: sip.1und1.de  
 - STUN/Proxy: stun.1und1.de  
 - Contact internet telephony: Only via IPv4  
 - Transport Protocol: Automatic  
- Telephony / Telephony Numbers / Line Settings:  
 - Telephony Connection / Kep port sharing of the internet router enabled for telephony: check  
 - Telephony Connection / Keep port forwarding enabled every: 30 sec.  

# 4. Dynamic DNS (DuckDNS) <a name="ddns"></a>

### 4.1 DuckDNS account <a name="41_duckdns_account"></a>  
- create an account or login into your [DuckDNS](https://www.duckdns.org/) account  
- create a token and add a domain to your account (e.g.: http://3x3cut0r.duckdns.org)  
- copy your token into your clipboard  

### 4.2 install os-ddclient plugin <a name="42_ddclient_plugin"></a>  
- Settings / Firmware / Plugins: install os-ddclient  
- reload site (F5) to see the plugin under Services  

### 4.3 configure os-ddclient plugin <a name="43_configure_plugin"></a>  
- Services / Dynamic DNS / Settings -> Add:  
 - Service: DuckDNS  
 - Username: <your DuckDNS E-Mail>  
 - Password: <your DuckDNS Token>  
 - Hostname: <your DuckDNS domain (e.g.: http://3x3cut0r.duckdns.org) >  
 - Check ip method: Interface  
 - Force SSL: Check  
 - Interface to monitor: WAN

### 4.4 ddclient via console (optional) <a name="44_ddclient_console"></a>  
**if WAN as 'Check ip method' is not working:**  
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

**hints:**  
- **you need to rechange the ddclient.conf via shell after changing anything via GUI**  
- **you can do more with the console as with the GUI because not all options are integrated with the GUI (like if=web as update method for ddnss.de for example)**  


### 4.5 disable rebind attack warning <a name="45_rebind_attack"></a>  
**to prevent ERROR: rebind attack dynamic dns:**  
- System / Settings / Administration / Alternate Hostnames:
 - enter your ddns domains (e.g.: '3x3cut0r.duckdns.org' - space-separated list)

**DONE**  

# 5. Lets Encrypt Certificate via ACME-Client (SSL/HTTPS) <a name="le"></a>

### 5.1 install os-acme-client plugin <a name="51_install_plugin"></a>  
- Settings / Firmware / Plugins: install os-acme-client  
- reload site (F5) to see the plugin under Services  

### 5.2 configure os-acme-client plugin (HTTP-01 - single domain)<a name="52_configure_plugin"></a>  
1. Services / ACME Client / **Settings**:  
 - **Enable Plugin: Check**  
 - Enable Auto Renewal: Check  
2. Services / ACME Client / Accounts / **Accounts -> Add**:  
 - Name: <name of your email address >  
 - Description: <name of your email address >  
 - E-Mail Address: <your email address>  
 - ACME CA: Let's Encrypt (default)  
 - Save  
3. Services / ACME Client / Challenge Types / **Challenge Types -> Add**:  
 - **Challenge HTTP-01: (single Domain)**  
 - (for other Challenges see [github.com/opnsense/plugins/pull/66](https://github.com/opnsense/plugins/pull/66))  
 - Name: LE HTTP-01 Challenge
 - Description: LE HTTP-01 Challenge  
 - Challenge Type: HTTP-01  
 - HTTP Service: OPNsense Web Service (automatic port forward)  
 - IP Auto-Discovery: Check  
 - Interface: WAN  
 - IP Addresses: < leave blank >  
 - Save  
4. Services / ACME Client / Automations / **Automations -> Add**:  
 - Enabled: Check  
 - Name: Restart OPNsense Web UI
 - Description: Restart OPNsense Web UI
 - Run Command: Restart OPNsense Web UI
 - Save
5. Services / ACME Client / Certificates / **Certificates -> Add**:  
 - Enabled: Check  
 - Common Name: <name of your domain (e.g.: 3x3cut0r.duckdns.org) >  
 - Description: LE Certificate
 - Alt Names: < you can add more domains or subdomains here >  
 - ACME Account: <your account, created on point 2. >  
 - Challenge Type: <your challenge type, created on point 3. >  
 - Auto Renewal: Check  
 - Renewal Interval: **30**  
 - Key Length: 4096 bit (default)  
 - OCSP Must Staple: Unchecked
 - Automations: < your automation created in 4. >  
 - DNS Alias Mode: Not using DNS alias mode  
 - Save  
5. Services / ACME Client / Automations / **Automations -> Add**:  
 - Enabled: Check  
 - Name: Restart OPNsense Web UI
 - Description: Restart OPNsense Web UI
 - Run Command: Restart OPNsense Web UI
 - Save
6. Services / ACME Client / Certificates / **Certificates**:  
 - **Issue/Renew All Certificates**  
7. Check if your Certifice is issued under Services / ACME Client / Log Files:  

```shell
AcmeClient: imported ACME X.509 certificate: 3x3cut0r.duckdns.org
AcmeClient: imprting ACME CA: R3
AcmeClient: successfully issued/renewed certificate: 3x3cut0r.duckdns.org
...

```
8. **System / Settings / Administration / SSL Certificate:**
 - SSL Certificate: < select your generated ACME Client Certificate >  
 - Save  
9. **DONE**  


### Find Me <a name="findme"></a>

![E-Mail](https://img.shields.io/badge/E--Mail-executor55%40gmx.de-red)
* [GitHub](https://github.com/3x3cut0r)
* [DockerHub](https://hub.docker.com/u/3x3cut0r)

### License <a name="license"></a>

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) - This project is licensed under the GNU General Public License - see the [gpl-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) for details.
