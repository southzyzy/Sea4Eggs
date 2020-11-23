###################
### Edge-Router ###
###################
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Disable Error Reporting messages
4) Syslog
5) Netflow
6) Port Security
7) TACACS
8) SSH
9) ACL

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Disable Error Reporting messages>
	|_ int g0/0
	|_ no ip unreachables
	|_ exit

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_
	|_ login on-success log
	|_ login on-failure log

<Netflow>
	|_ # Flow record settings
	|_ flow record NetflowTraffic
	|_ match ipv4 tos
	|_ match ipv4 protocol
	|_ match ipv4 source address
	|_ match ipv4 destination address
	|_ match transport source-port
	|_ match transport destination-port
	|_ match interface input
	|_ collect routing source as
	|_ collect routing destination as
	|_ collect interface output
	|_ collect counter bytes
	|_ collect counter packets
	|_ collect timestamp sys-uptime first
	|_ collect timestamp sys-uptime last
	|_ collect application name
	|_  
	|_ # Netflow record exporter settings
	|_ flow exporter NetflowExport
	|_ destination 172.16.67.2
	|_ source GigabitEthernet0/1
	|_ transport udp 9991
	|_ export-protocol netflow-v9
	|_ template data timeout 60
	|_ option application-table timeout 60
	|_ option application-attributes timeout 300
	|_  
	|_ # Netflow monitor settings
	|_ flow monitor NetflowMonitor
	|_ exporter NetflowExport
	|_ cache timeout active 60
	|_ cache timeout inactive 15
	|_ record NetflowTraffic
	|_
	|_ int g0/1
	|_ ip flow monitor NetflowMonitor input
	|_ ip flow monitor NetflowMonitor output

<Port Security>
	|_ # Shut down unused ports
	|_ int range Serial0/0/0-1
	|_ no ip address
	|_ shutdown

<TACACS>
	|_ # Setting up TACACS client on R1-MGNT-SYSLOG
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_
	|_ ip tacacs source-interface g0/1
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ local
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH Creation>
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_ 
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default

<ACL>
	|_ # Anti-Spoofing
	|_ !  Deny RFC 3330 special-use address
	|_ access-list 110 remark Deny special-use and private addresses
	|_ access-list 110 deny ip 0.0.0.0 0.0.0.0 any
	|_ access-list 110 deny ip 127.0.0.0 0.255.255.255 any
	|_ access-list 110 deny ip 192.0.2.0 0.0.0.255 any
	|_ access-list 110 deny ip 224.0.0.0 31.255.255.255 any
	|_ 
	|_ ! Filter RFC 1918 space
	|_ access-list 110 deny ip 10.0.0.0 0.255.255.255 any
	|_ access-list 110 deny ip 192.168.0.0 0.0.255.255 any
	|_ 
	|_ ! Deny your CIDR space coming in from Internet as source
	|_ access-list 110 deny ip object-group INTERNAL_NETWORK any
	|_ 
	|_ ! Deny WAN from accessing internal infrastructure addresses
	|_ access-list 110 deny ip any object-group INTERNAL_NETWORK
	|_ 
	|_ ! Permit transit traffic
	|_ access-list 110 permit ip any any
	|_
	|_ ! Apply the ACL to the WAN facing interface
	|_ int g0/0
	|_ ip access-group 110 in




####################
### ASA-Firewall ###
####################
Defenses Configured:
1) Disable HTTP Server
2) Disable SNMP
3) Syslog
4) Enable Threat Detection
5) Modify Default Inspection Policy to inspect ICMP
6) HTTP Traffic Inspection
7) Firewall Security Levels
8) TACACS
9) SSH
10) ACL

<Disable HTTP Server>
	|_ no http server enable

<Disable SNMP>
	|_ no snmp-server enable traps snmp authentication linkup linkdown coldstart warmstart

<Enable Logging>
	|_ logging trap 6
	|_ logging host R1-MGNT-SYSLOG 172.16.67.2
	|_ logging timestamp
	|_ logging enable

<Threat Detection>
	|_ threat-detection statistics tcp-intercept rate-interval 3 burst-rate 60 average-rate 30

<Default Inspection Policy>
	|_ policy-map global_policy
	|_ class inspection_default
	|_ inspect icmp

<HTTP Traffic Inspection>
	|_ access-list DMZ_HTTP_SERVER extended permit tcp any4 object-group DMZ_SERVER eq 80
	|_
	|_ class-map DMZ_HTTP_TRAFFIC
	|_ match access-list DMZ_HTTP_SERVER
	|_
	|_ policy-map INSPECT_DMZ_HTTP_TRAFFIC
	|_ class DMZ_HTTP_TRAFFIC
	|_ set connection embryonic-conn-max 15 per-client-embryonic-max 10
	|_ set connection per-client-max 15
	|_ set connection timeout idle 0:0:10
	|_
	|_ service-policy INSPECT_DMZ_HTTP_TRAFFIC interface EDGE-ROUTER
	|_ service-policy INSPECT_DMZ_HTTP_TRAFFIC interface L3SW1
	|_ service-policy INSPECT_DMZ_HTTP_TRAFFIC interface L3SW2
	|_ service-policy INSPECT_DMZ_HTTP_TRAFFIC interface R1-MGNT-SYSLOG

<Firewall Security Levels>
	|_ # Allow traffic of same security level
	|_ same-security-traffic permit inter-interface
	|_
	|_ # Configure the respective interfaces
	|_ int g0/0
	|_ nameif EDGE-ROUTER
	|_ security-level 0
	|_
	|_ int g0/1
	|_ nameif R1-MGNT-SYSLOG
	|_ security-level 100
	|_
	|_ int g0/2
	|_ nameif R2-DMZ
	|_ security-level 50
	|_
	|_ int g0/4
	|_ nameif L3SW1
	|_ security-level 80
	|_
	|_ int g0/5
	|_ nameif L3SW2
	|_ security-level 80
	|_
	|_ int Management0/0
	|_ nameif management
	|_ security-level 100

<TACACS>
	|_ # Setup TACACS on firewall
	|_ aaa-server TACACSERVER protocol tacacs+
	|_ aaa-server TACACSERVER (management) host 172.16.68.8 Sea4EggsC0mpl3xP@ssw0rd
	|_ server-port 49
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs
	|_
	|_ # Set enable password (local authentication, backup)
	|_ enable password C0mpl3xP@ssw0rd
	|_
	|_ # Configure AAA settings
	|_ aaa authentication ssh console TACACSERVER LOCAL
	|_ aaa authentication enable console TACACSERVER LOCAL
	|_ aaa accounting ssh console TACACSERVER
	|_ aaa accounting enable console TACACSERVER
	|_ aaa accounting command TACACSERVER
	|_ aaa authorization command TACACSERVER LOCAL

<SSH>
	|_ ssh version 2
	|_ ssh timeout 30
	|_ ssh 172.16.68.0 255.255.255.240 management

<ACL>
	|_ time-range OFFICE_HOURS
	|_ periodic weekdays 9:00 to 18:00
	|_
	|_ ----------------------- EDGE -----------------------
	|_ # Permit EDGE-ROUTER access to SYSLOG_SERVER and TACACS_SERVER
	|_ access-list EDGE_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group EDGE_ROUTER_MGNT object-group SYSLOG_SERVER
	|_ access-list EDGE_INBOUND extended permit object-group NETFLOW_TRAFFIC object-group EDGE_ROUTER_MGNT object-group SYSLOG_SERVER
	|_ access-list EDGE_INBOUND extended permit object-group TACACS_TRAFFIC object-group EDGE_ROUTER_MGNT object-group TACACS_SERVER
	|_
	|_ # Permit specified traffic to DMZ_SERVER and HONEYPOT_SERVER
	|_ access-list EDGE_INBOUND extended permit object-group DMZ_TRAFFIC any4 object-group DMZ_SERVER
	|_ access-list EDGE_INBOUND extended permit object-group HONEYPOT_TRAFFIC any4 object-group HONEYPOT_SERVER
	|_
	|_ # Explicit deny any
	|_ access-list EDGE_INBOUND extended deny ip any any
	|_
	|_ # Apply the ACL
	|_ access-group EDGE_INBOUND in interface EDGE-ROUTER
	|_
	|_ ----------------------- R1-MGNT -----------------------
	|_ # Deny access to HONEYPOT_NET
	|_ access-list R1_MGNT_INBOUND extended deny ip any object-group HONEYPOT_NET
	|_ 
	|_ access-list R1_MGNT_INBOUND extended permit object-group DNS_TRAFFIC object-group IT_MANAGEMENT object-group DNS_SERVER
	|_
	|_ # Deny SYSLOG_NET to EDGE-ROUTER (SSH), R2-DMZ and INTERNAL_NET and DMZ_NET
	|_ access-list R1_MGNT_INBOUND extended deny tcp object-group SYSLOG_NET object-group EDGE_ROUTER_MGNT eq 22
	|_ access-list R1_MGNT_INBOUND extended deny ip object-group SYSLOG_NET object-group R2_FW_UPLINK
	|_ access-list R1_MGNT_INBOUND extended deny ip object-group SYSLOG_NET object-group INTERNAL_NETWORK
	|_ access-list R1_MGNT_INBOUND extended deny ip object-group SYSLOG_NET object-group DMZ_NET
	|_ 
	|_ # Permit all other traffic (Internet Access)
	|_ access-list R1_MGNT_INBOUND extended permit ip any4 any4
	|_
	|_ # Apply the ACL
	|_ access-group R1_MGNT_INBOUND in interface R1-MGNT-SYSLOG
	|_
	|_ ----------------------- R2-DMZ -----------------------
	|_ # Permit DMZ_SERVER access to SYSLOG_SERVER
	|_ access-list R2_DMZ_INBOUND extended permit object-group DMZ_SYSLOG_TRAFFIC object-group DMZ_SERVER object-group SYSLOG_SERVER
	|_ 
	|_ # Permit R2-DMZ access to SYSLOG_SERVER and TACACS_SERVER
	|_ access-list R2_DMZ_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group R2_DMZ_MGNT object-group SYSLOG_SERVER
	|_ access-list R2_DMZ_INBOUND extended permit object-group TACACS_TRAFFIC object-group R2_DMZ_MGNT object-group TACACS_SERVER 
	|_
	|_ # Deny any access to EDGE-ROUTER (SSH), R1-MGNT-SYSLOG, INTERNAL NETWORK and IT_MANAGEMENT
	|_ access-list R2_DMZ_INBOUND extended deny tcp any object-group EDGE_ROUTER_MGNT eq 22
	|_ access-list R2_DMZ_INBOUND extended deny ip any object-group R1_FW_UPLINK
	|_ access-list R2_DMZ_INBOUND extended deny ip any object-group INTERNAL_NETWORK
	|_ access-list R2_DMZ_INBOUND extended deny ip any object-group IT_MANAGEMENT
	|_
	|_ # Permit all other traffic (Internet Access)
	|_ access-list R2_DMZ_INBOUND extended permit ip any4 any4
	|_
	|_ # Apply the ACL
	|_ access-group R2_DMZ_INBOUND in interface R2-DMZ
	|_
	|_ ----------------------- L3SW1 -----------------------
	|_ # Deny access to HONEYPOT_NET
	|_ access-list L3SW1_INBOUND extended deny ip any object-group HONEYPOT_NET
	|_ 
	|_ # Allow L3SW1 access to SYSLOG_SERVER and TACACS_SERVER
	|_ access-list L3SW1_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group L3SW1_UPLINK object-group SYSLOG_SERVER
	|_ access-list L3SW1_INBOUND extended permit object-group TACACS_TRAFFIC object-group L3SW1_UPLINK object-group TACACS_SERVER
	|_
	|_ # Allow INTERNAL_SERVER_NET access to SYSLOG_SERVER
	|_ access-list L3SW1_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group INTERNAL_SERVER_NET object-group SYSLOG_SERVER
	|_
	|_ # Permit INTERNAL_NETWORK web access (HTTP) and ICMP to DMZ_SERVER
	|_ access-list L3SW1_INBOUND extended permit icmp object-group INTERNAL_NETWORK object-group DMZ_SERVER
	|_ access-list L3SW1_INBOUND extended permit tcp object-group INTERNAL_NETWORK object-group DMZ_SERVER eq 80
	|_
	|_ ! Allow IT_SUPPORT to access Management Network Jumphost (TACACS) via SSH and SYSLOG Web UI via HTTP/HTTPS, as well as ICMP
	|_ access-list L3SW1_INBOUND extended permit icmp object-group IT_SUPPORT object-group TACACS_SERVER
	|_ access-list L3SW1_INBOUND extended permit tcp object-group IT_SUPPORT object-group TACACS_SERVER eq 22
	|_ access-list L3SW1_INBOUND extended permit icmp object-group IT_SUPPORT object-group SYSLOG_SERVER
	|_ access-list L3SW1_INBOUND extended permit tcp object-group IT_SUPPORT object-group SYSLOG_SERVER eq 80
	|_ access-list L3SW1_INBOUND extended permit tcp object-group IT_SUPPORT object-group SYSLOG_SERVER eq 443
	|_ 
	|_ # Deny any access to EDGE-ROUTER (SSH), R1-MGNT-SYSLOG, R2-DMZ, IT_MANAGEMENT and DMZ_NET
	|_ access-list L3SW1_INBOUND extended deny tcp any object-group EDGE_ROUTER_MGNT eq 22
	|_ access-list L3SW1_INBOUND extended deny ip any object-group R1_FW_UPLINK
	|_ access-list L3SW1_INBOUND extended deny ip any object-group R2_FW_UPLINK
	|_ access-list L3SW1_INBOUND extended deny ip any object-group IT_MANAGEMENT
	|_ access-list L3SW1_INBOUND extended deny ip any object-group DMZ_NET
	|_
	|_ # Deny LEGAL_SECRETARIES traffic to internet during OFFICE_HOURS
	|_ access-list L3SW1_INBOUND extended deny ip object-group LEGAL_SECRETARIES any time-range OFFICE_HOURS
	|_
	|_ # Permit all other traffic (Internet Access)
	|_ access-list L3SW1_INBOUND extended permit ip any4 any4
	|_
	|_ # Apply the ACL
	|_ access-group L3SW1_INBOUND in interface L3SW1
	|_ 
	|_ ----------------------- L3SW2 -----------------------
	|_ # Deny access to HONEYPOT_NET
	|_ access-list L3SW2_INBOUND extended deny ip any object-group HONEYPOT_NET
	|_ 
	|_ # Allow L3SW2 access to SYSLOG_SERVER and TACACS_SERVER
	|_ access-list L3SW2_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group L3SW2_UPLINK object-group SYSLOG_SERVER
	|_ access-list L3SW2_INBOUND extended permit object-group TACACS_TRAFFIC object-group L3SW2_UPLINK object-group TACACS_SERVER
	|_
	|_ # Allow INTERNAL_SERVER_NET access to SYSLOG_SERVER
	|_ access-list L3SW2_INBOUND extended permit object-group SYSLOG_TRAFFIC object-group INTERNAL_SERVER_NET object-group SYSLOG_SERVER
	|_
	|_ # Permit INTERNAL_NETWORK web access (HTTP) and ICMP to DMZ_SERVER
	|_ access-list L3SW2_INBOUND extended permit icmp object-group INTERNAL_NETWORK object-group DMZ_SERVER
	|_ access-list L3SW2_INBOUND extended permit tcp object-group INTERNAL_NETWORK object-group DMZ_SERVER eq 80
	|_
	|_ ! Allow IT_SUPPORT to access Management Network Jumphost (TACACS) via SSH and SYSLOG Web UI via HTTP/HTTPS, as well as ICMP
	|_ access-list L3SW2_INBOUND extended permit icmp object-group IT_SUPPORT object-group TACACS_SERVER
	|_ access-list L3SW2_INBOUND extended permit tcp object-group IT_SUPPORT object-group TACACS_SERVER eq 22
	|_ access-list L3SW2_INBOUND extended permit icmp object-group IT_SUPPORT object-group SYSLOG_SERVER
	|_ access-list L3SW2_INBOUND extended permit tcp object-group IT_SUPPORT object-group SYSLOG_SERVER eq 80
	|_ access-list L3SW2_INBOUND extended permit tcp object-group IT_SUPPORT object-group SYSLOG_SERVER eq 443
	|_ 
	|_ # Deny any access to EDGE-ROUTER (SSH), R1-MGNT-SYSLOG, R2-DMZ, IT_MANAGEMENT and DMZ_NET
	|_ access-list L3SW2_INBOUND extended deny tcp any object-group EDGE_ROUTER_MGNT eq 22
	|_ access-list L3SW2_INBOUND extended deny ip any object-group R1_FW_UPLINK
	|_ access-list L3SW2_INBOUND extended deny ip any object-group R2_FW_UPLINK
	|_ access-list L3SW2_INBOUND extended deny ip any object-group IT_MANAGEMENT
	|_ access-list L3SW2_INBOUND extended deny ip any object-group DMZ_NET
	|_
	|_ # Deny LEGAL_SECRETARIES traffic to internet during OFFICE_HOURS
	|_ access-list L3SW2_INBOUND extended deny ip object-group LEGAL_SECRETARIES any time-range OFFICE_HOURS
	|_
	|_ # Permit all other traffic (Internet Access)
	|_ access-list L3SW2_INBOUND extended permit ip any4 any4
	|_
	|_ # Apply the ACL
	|_ access-group L3SW2_INBOUND in interface L3SW2




###############
### R1-MGNT ###
###############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) Port Security
5) TACACS
6) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_
	|_ login on-success log
	|_ login on-failure log

<Port Security>
	|_ # Shutdown unused ports
	|_ int Embedded-Service-Engine0/0
	|_ shutdown
	|_
	|_ int range s0/0/0-1
	|_ shutdown

<TACACS>
	|_ # Setting up TACACS client on R1-MGNT-SYSLOG
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8 
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ enable
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_ 
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default




##############
### R2_DMZ ###
##############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) Port Security
5) TACACS
6) SSH
7) ACL

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_ logging source-interface g0/0
	|_
	|_ login on-success log
	|_ login on-failure log

<Port Security>
	|_ # Shutdown unused ports
	|_ int Embedded-Service-Engine0/0
	|_ shutdown
	|_
	|_ int range s0/0/0-1
	|_ shutdown

<TACACS>
	|_ # Setting up TACACS client on DMZ-Router
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_
	|_ ip tacacs source-interface g0/0
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ enable
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_ 
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default

<ACL>
	|_ # ACL to restrict outbound honey-pot traffic
	|_ ip access-list extended RESTRICT_HONEY_OUTBOUND
	|_ deny ip object-group HONEYPOT object-group INTERNAL_NETWORK
	|_ deny ip object-group HONEYPOT object-group MANAGEMENT_NETWORK
	|_ deny ip object-group HONEYPOT object-group DMZ
	|_ deny ip object-group HONEYPOT object-group SYSLOG
	|_ permit ip object-group HONEYPOT any
	|_
	|_ # Apply HONEYPOT ACL
	|_ int g0/2
	|_ ip access-group RESTRICT_HONEY_OUTBOUND in




##############
### L3_SW1 ###
##############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) DHCP Snooping
5) Port Security
5) TACACS
6) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_
	|_ login on-success log
	|_ login on-failure log

<DHCP Snooping>
	|_ ip dhcp relay information trust-all

<Port Security>
	|_ # Configure unused port to access unused vlan and shutdown
	|_ int range g1/0/4-23
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport access vlan 99
	|_ shutdown
	|_
	|_ int range g1/1/1-4
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport access vlan 99
	|_ shutdown
	|_
	|_ int g1/0/1
	|_ switchport mode trunk
	|_ switchport nonegotiate
	|_ switchport trunk native vlan 99
	|_ switchport trunk allowed vlan 100,110,200,300,310,37,66
	|_
	|_ int g1/0/2
	|_ switchport nonegotiate
	|_
	|_ int g1/0/24
	|_ switchport mode trunk
	|_ switchport nonegotiate
	|_ switchport trunk native vlan 99
	|_ switchport trunk allowed vlan 100,110,200,210,300,310,37,66

<TACACS>
	|_ # Setting up TACACS client on L3_SW1
	|_ aaa new-model
	|_ aaa group server tacacs+ TACACSERVER
	|_ server-private 172.16.68.8 key Sea4EggsC0mpl3xP@ssw0rd
	|_ ip vrf forwarding Mgmt-vrf
	|_
	|_ ip tacacs source-interface g0/0
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group TACACSERVER local
	|_ aaa authentication enable default group TACACSERVER enable
	|_ aaa authorization commands 15 default group TACACSERVER local
	|_ aaa accounting commands 0 default start-stop group TACACSERVER
	|_ aaa accounting commands 15 default start-stop group TACACSERVER
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_ 
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default




##############
### L3_SW2 ###
##############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) DHCP Snooping
5) Port Security
5) TACACS
6) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_
	|_ login on-success log
	|_ login on-failure log

<DHCP Snooping>
	|_ ip dhcp relay information trust-all

<Port Security>
	|_ # Configure unused port to access unused vlan and shutdown
	|_ int range g1/0/4-23
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport access vlan 99
	|_ shutdown
	|_
	|_ int range g1/1/1-4
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport access vlan 99
	|_ shutdown
	|_ 
	|_ int g1/0/1
	|_ switchport mode trunk
	|_ switchport nonegotiate
	|_ switchport trunk native vlan 99
	|_ switchport trunk allowed vlan 100,110,200,300,310,37,66
	|_
	|_ int g1/0/2
	|_ switchport nonegotiate
	|_
	|_ int g1/0/24
	|_ switchport mode trunk
	|_ switchport nonegotiate
	|_ switchport trunk native vlan 99
	|_ switchport trunk allowed vlan 100,110,200,210,300,310,37,66

<TACACS>
	|_ # Setting up TACACS client on L3_SW2
	|_ aaa new-model
	|_ aaa group server tacacs+ TACACSERVER
	|_ server-private 172.16.68.8 key Sea4EggsC0mpl3xP@ssw0rd
	|_ ip vrf forwarding Mgmt-vrf
	|_
	|_ ip tacacs source-interface g0/0
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group TACACSERVER local
	|_ aaa authentication enable default group TACACSERVER enable
	|_ aaa authorization commands 15 default group TACACSERVER local
	|_ aaa accounting commands 0 default start-stop group TACACSERVER
	|_ aaa accounting commands 15 default start-stop group TACACSERVER
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_ 
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default




##############
# ACCESS-SW1 #
##############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) Port Security
5) Securing Trunk
6) Spanning Tree
7) DHCP Snooping
8) Dynamic ARP Inspection
9) IP Source Guard
10) TACACS
11) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_ logging source-interface vlan 68
	|_
	|_ login on-success log
	|_ login on-failure log

<Port Security>
	|_ int range fa0/1-23
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport port-security
	|_ switchport port-security violation shutdown
	|_ switchport port-security maximum 1
	|_ switchport port-security mac-address sticky
	|_ # Shut down all ports (ports in use will be 'no shut' separately)
	|_ shut
	|_
	|_ # Configure unused port to access unused vlan
	|_ int fa0/18-23
	|_ switchport access vlan 99

<Securing Trunk>
	|_ int range g0/1-2
	|_ switchport mode trunk
	|_ switchport nonegotiate
	|_ switchport trunk native vlan 99
	|_ switchport trunk allowed vlan 37,66,100,110,200,300,310

<Spanning Tree>
	|_ # Enable bpduguard for all data ports
	|_ int range fa0/1-23
	|_ spanning-tree bpduguard enable
	|_ spanning-tree portfast

<DHCP Snooping>
	|_ ip dhcp snooping
	|_ ip dhcp snooping vlan 37,66,100,110,200,300,310
	|_ ip dhcp snooping verify mac-address
	|_
	|_ int range fa0/1-16
	|_ ip dhcp snooping limit rate 25
	|_ int range fa0/18-23
	|_ ip dhcp snooping limit rate 25
	|_
	|_ int range g0/1-2
	|_ ip dhcp snooping trust
	|_
	|_ int fa0/17
	|_ ip dhcp snooping trust

<Dynamic ARP Inspection>
	|_ ip arp inspection vlan 37,66,100,110,200,300,310
	|_ ip arp inspection validate src-mac dst-mac ip
	|_
	|_ int range fa0/1-16
	|_ ip arp inspection limit rate 100
	|_ int range fa0/18-23
	|_ ip arp inspection limit rate 100
	|_
	|_ int range g0/1-2
	|_ ip arp inspection trust
	|_
	|_ int fa0/17
	|_ ip arp inspection trust
	|_ int fa0/24
	|_ ip arp inspection trust

<IP Source Guard>
	|_ int range fa0/1-23
	|_ ip verify source port-security

<TACACS>
	|_ # Setting up TACACS client on ACCESS-SW1
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_ 
	|_ ip tacacs source-interface fa0/24
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ enable
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret SW1C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ # Configure SSH details
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default




##############
# ACCESS-SW2 #
##############
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) Port Security
5) Spanning Tree
6) DHCP Snooping
7) Dynamic ARP Inspection
8) IP Source Guard
9) TACACS
10) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_ logging source-interface vlan 68
	|_
	|_ login on-success log
	|_ login on-failure log

<Port Security>
	|_ int range fa0/1-23
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ switchport port-security
	|_ switchport port-security violation shutdown
	|_ switchport port-security maximum 1
	|_ switchport port-security mac-address sticky
	|_ # Shut down all ports (ports in use will be 'no shut' separately)
	|_ shut
	|_
	|_ # Configure unused port
	|_ int fa0/23
	|_ switchport access vlan 99

<Spanning Tree>
	|_ int range fa0/1-23
	|_ spanning-tree bpduguard enable
	|_ spanning-tree portfast

<DHCP Snooping>
	|_ ip dhcp snooping
	|_ ip dhcp snooping vlan 210
	|_ ip dhcp snooping verify mac-address
	|_
	|_ int range fa0/1-23
	|_ ip dhcp snooping limit rate 25
	|_
	|_ int range g0/1-2
	|_ ip dhcp snooping trust

<Dynamic ARP Inspection>
	|_ ip arp inspection vlan 210
	|_ ip arp inspection validate src-mac dst-mac ip
	|_
	|_ int range fa0/1-23
	|_ ip arp inspection limit rate 100
	|_
	|_ int range g0/1-2
	|_ ip arp inspection trust
	|_
	|_ int fa0/24
	|_ ip arp inspection trust

<IP Source Guard>
	|_ int range fa0/1-23
	|_ ip verify source port-security

<TACACS>
	|_ # Setting up TACACS client on ACCESS-SW2
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_
	|_ ip tacacs source-interface fa0/24
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ enable
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret SW2C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ # Configure SSH details
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default




###########
# MGNT_SW #
###########
Defenses Configured:
1) Disable CDP
2) Disable HTTP Server
3) Syslog
4) Port Security
5) Spanning Tree
6) DHCP Snooping
7) Dynamic ARP Inspection
8) TACACS
9) SSH

<Disable CDP>
	|_ no cdp run

<Disable HTTP Server>
	|_ no ip http server
	|_ no ip http secure-server

<Enable logging>
	|_ logging 172.16.67.2
	|_ logging trap 6
	|_
	|_ login on-success log
	|_ login on-failure log

<Port Security>
	|_ int range g1/0/1-28
	|_ switchport mode access
	|_ switchport nonegotiate
	|_ # Shut down all ports (ports in use will be 'no shut' separately)
	|_ shut
	|_
	|_ int range g1/0/1-6
	|_ switchport port-security
	|_ switchport port-security violation shutdown
	|_ switchport port-security maximum 8
	|_ switchport port-security mac-address sticky
	|_ no shut
	|_ exit
	|_ 
	|_ int range g1/0/7-23
	|_ switchport access vlan 99
	|_ switchport port-security
	|_ switchport port-security violation shutdown
	|_ switchport port-security maximum 1
	|_ switchport port-security mac-address sticky
	|_
	|_ int range g1/0/25-28
	|_ switchport access vlan 99
	|_ switchport port-security
	|_ switchport port-security violation shutdown
	|_ switchport port-security maximum 1
	|_ switchport port-security mac-address sticky
	|_
	|_ ! Shutdown Management Port
	|_ int fa0
	|_ no ip address
	|_ no ip route-cache
	|_ shut

<Spanning Tree>
	|_ int range g1/0/7-23
	|_ spanning-tree bpduguard enable
	|_
	|_ int range g1/0/25-28
	|_ spanning-tree bpduguard enable

<DHCP Snooping>
	|_ ip dhcp snooping
	|_ ip dhcp snooping verify mac-address

<Dynamic ARP Inspection>
	|_ ip arp inspection vlan 1
	|_ ip arp inspection validate src-mac dst-mac ip
	|_
	|_ int range g1/0/7-23
	|_ ip arp inspection limit rate 100
	|_ int range g1/0/25-28
	|_ ip arp inspection limit rate 100
	|_
	|_ int range g1/0/1-6
	|_ ip arp inspection trust
	|_
	|_ int g1/0/24
	|_ ip arp inspection trust

<TACACS>
	|_ # Setting up TACACS client on ACCESS-SW2
	|_ aaa new-model
	|_ tacacs server TACACSERVER
	|_ address ipv4 172.16.68.8
	|_ key Sea4EggsC0mpl3xP@ssw0rd
	|_
	|_ ip tacacs source-interface vlan 1
	|_
	|_ # Configure AAA settings
	|_ aaa authentication login default group tacacs+ local
	|_ aaa authentication enable default group tacacs+ enable
	|_ aaa authorization commands 15 default group tacacs+ local
	|_ aaa accounting commands 0 default start-stop group tacacs+
	|_ aaa accounting commands 15 default start-stop group tacacs+
	|_
	|_ # Enable privilege mode password (local authentication, backup)
	|_ enable algorithm-type sha256 secret SW2C0mpl3xP@ssw0rd
	|_
	|_ # Create local account for local authentication (local authentication, backup)
	|_ username sea4eggs-admin privilege 15 algorithm-type sha256 secret 1qwer$#@!~sea4eggs

<SSH>
	|_ # Configure SSH details
	|_ ip domain-name intranet.sea4eggs.sitict.net
	|_ crypto key generate rsa modulus 2048
	|_ ip ssh version 2
	|_
	|_ # Enable SSH on virtual terminal
	|_ line vty 0 15
	|_ transport input ssh
	|_ login authentication default
