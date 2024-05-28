# How to create an IPsec VPN between Unifi USG and Mikrotik firewalls
* Hardware and software
    * Unifi UDM SE Unifi OS version 4.0.3
    * Unifi Network Application version 8.2.92
    * Mikrotik RB5009UG+S+IN with RouterOS v7.14.3
    
* Sources:
    * UniFi Gateway - Site-to-Site IPsec VPN with Third-Party Gateways (Advanced)
        * https://help.ui.com/hc/en-us/articles/7983431932439-UniFi-Gateway-Site-to-Site-IPsec-VPN-with-Third-Party-Gateways-Advanced  
                
## Mikrotik configuration in WebFig interface
### Select: IP -> IPsec -> Peers

| Add New | |
| - | - |
| Enabled | checked
| Name | ***UNIFI-01***
| Address |	UDM WAN address
| Port | empty
| Local Address | Mikrotik WAN address
| Auth. Method | pre shared key
| Exchange Mode | main
| Passive | uncheked
| Send INITIAL_CONTACT | checked

### Select: IP -> IPsec -> Profiles
| New Profile | |
| - | - |
| Name | ***profile-unifi***
| Hash Algorithms | sha1
| Encryption Algorithm | aes-128
| DH Group | modp2048 (=DH Group 14 to leave Unifi settings in Auto state)
| Proposal Check | obey
| Lifetime |  08:00:00
| Lifebytes | empty
| Nat Traversal | uncheck 
| DPD Interval | 60s
| DPD Maximum Failures | 5

### Select: IP -> IPsec -> Identities
| Add New | |
| - | - |
| Enabled |	checked
| Peer | ***UNIFI-01***
| Auth. Method |			pre shared key
| Secret | paste your secret
| Notrack Chain | empty
| Policy Template Group | default
| Notrack Chain | empty
| My ID Type | auto
| Remote ID Type | auto
| Match By | remote id
| Mode Configuration | empty
| Generate Policy | no

### Select: IP -> IPsec -> Proposals

|  Add New  | |
| - | - |
| Enabled | checked
| Name | ***proposal-unifi***
| Auth. Algorithms | sha1
| Encr. Algorithms | aes-128 cbc (to use Unifi in Auto)
| Lifetime | 08:00:00 (to match Unifi Auto)
| PFS Group | modp2048

### Select: IP -> IPsec -> Policies
*You don't have to Disable default

| Add New IPsec Policy | |
| - | - |
| Enabled | checked |
| Peer | ***UNIFI-01*** 
| Tunnel | checked
| Src. Address | Mikrotik internal LAN network address (the whole network e.g. 192.168.88.0/24) |
| Src. Port | empty
| Dst. Address | USG internal LAN network address
| Dst. Port | empty
| Protocol | 255 (all)
| Template | uncheck
| Action | encrypt
| Level | unique (Default was "require", but to establish connection with multiple network subnets it needs to be "unique".)
| IPsec Protocols | esp
| Proposal | ***proposal-unifi***

### Select: IP -> Firewall -> NAT
| New NAT Rule | |
| - | - |
| Chain | srcnat
| Src. Address | LAN network of Mikrotik
| Dst. Address | LAN network of USG
| Action | Accept

* Move the rule to the top of the firewall rules.

## Unifi Network Application configuration (version 8.2.92)
### Settings -> VPN -> Sit-to-Site VPN ->Create New 
| Configurations | |
| - | - |
| Name| mikrotik-unifi-vpn |
| Pre-Shared Key | secret key |
| Local IP | WAN IP of UDM  or Enter IP Address manually |
| Peer IP / Host| WAN IP of Mikrotik |
| VPN Type | Policy Based |
| Remote Networks | Mikrotik subnets |
| IPsec Profile | Customized | 
| **Advanced** | Auto (here is the default values) |
| Key Exchange Version | IKEv1
| **IKE** | |
| Encryption | AES-128
| Hash | SHA1
| DH Group | 14
| Lifetime | 28800
| **ESP** | |
| Encryption | AES-128
| Hash | SHA1
| DH Group | 14
| Lifetime | 3600
| Prefect Forward Secrecy (PFS) | checked 
| Local Authentication ID | Auto checked (WAN IP of UDM)
| Remote Authentication ID | Auto checked (WAN IP of Mikrotik)

## Mikrotik IP -> IPsec -> Installed SAs
* Something like this should show up when connection is up

| | SPI	| Src. Address | Dst. Address |Auth. Algorithm | Encr. Algorithm | Encr. Key Size | Current Bytes
| - | - | - | - | - | - | - | - |
| EH | 4cbfd50 | 62.x.y.z | 83.x.y.z | sha1 | aes cbc | 128 | 15540	
| EH | c0a27199 | 83.x.y.z | 62.x.y.z | sha1 | aes cbc | 128 | 13440

## Ping
* You should be able to ping both ways now.
