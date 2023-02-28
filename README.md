# SD-WAN

## Test Environment

| Name        | FortiOS |        ISP01        |        ISP02        | LAN                        |
|-------------|:-------:|:-------------------:|:-------------------:|----------------------------|
| FW-HUB-01   |  7.2.3  | 172.22.1.11 \| wan1 | 172.22.2.11 \| wan2 | 10.1.30.254 \| 10.1.37.254 |
| FW-SPOKE-01 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.21.254             |
| FW-SPOKE-02 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.22.254             |
| FW-SPOKE-03 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.23.254             |

### FW-HUB-01

```
config vpn ipsec phase1-interface
```
