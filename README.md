# Sea4Eggs
### Network Security Team Project

---

## SSH around MGNT Network

| Device | Network |
| --- |:---:|
| EdgeRouter | 172.16.0.1 |
| R1_MGNT | 172.16.68.1 |
| TheGreatWall | 172.16.68.2 |
| MGNT_SW | 172.16.68.3 |
| L3_SW1 | 172.16.68.4 |
| L3_SW2 | 172.16.68.5 |
| SW1 | 172.16.68.6 |
| SW2 | 172.16.68.7 |
| TACACS+ | 172.16.68.8 |
| R2_DMZ | 172.16.3.2 |
| DHCP | 172.16.66.3 |
| DNS | 172.27.54.34 |

#### SSH Command

```shell
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -c aes128-cbc -l <username> <target_ip>
```

---

## Password Configurations

#### Tacacs+ Configurations

| Network | Username | Password |
| --- |:---:| :---:|
| Network Manager | network_manager | Rapes&Redwine |
| Network Engineer 1 | network_engineer1 | Dates&Redwine |
| Network Engineer 2 | network_engineer2 | Grapes&Redwine |

#### Privilege Mode Password

| Device | Password |
| --- |:---:|
| SW1 | SW1C0mpl3xP@ssw0rd |
| SW2 | SW2C0mpl3xP@ssw0rd |
| Other | C0mpl3xP@ssw0rd | 

#### Local Accounts 

| Device | Username | Password |
| --- |:---:|:---:|
| Splunk | sea4eggs-admin | 1qwer$#@!~sea4eggs |
| Database | root | rootcatdogmousehorse |
| Database | sea4eggs-admin | catdogmousehorse |

---
