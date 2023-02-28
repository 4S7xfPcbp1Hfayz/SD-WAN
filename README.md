# SD-WAN

## Test Environment

| Name        | FortiOS |        ISP01        |        ISP02        | LAN                        |
|-------------|:-------:|:-------------------:|:-------------------:|----------------------------|
| FW-HUB-01   |  7.2.3  | 172.22.1.11 \| wan1 | - | 10.1.30.254 \| 10.1.37.254 |
| FW-SPOKE-01 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.21.254             |
| FW-SPOKE-02 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.22.254             |

BGP AS: 65000

Address Range for VPNs: 172.31.0.0/17

### FW-HUB-01

```
config system settings
    set tcp-session-without-syn enable
end
config vpn ipsec phase1-interface
    edit "DC-ISP1"
        set type dynamic
        set interface "wan1"
        set ike-version 2
        set peertype any
        set net-device disable
        set mode-cfg enable
        set proposal aes256-sha256
        set add-route disable
        set dpd on-idle
        set auto-discovery-sender enable
        set network-overlay enable
        set network-id 1
        set tunnel-search nexthop
        set ipv4-start-ip 172.31.1.10
        set ipv4-end-ip 172.31.1.253
        set ipv4-netmask 255.255.255.0
        set psksecret fortinet
        set dpd-retryinterval 60
    next
    edit "DC-ISP2"
        set type dynamic
        set interface "wan1"
        set ike-version 2
        set peertype any
        set net-device disable
        set mode-cfg enable
        set proposal aes256-sha256
        set add-route disable
        set dpd on-idle
        set auto-discovery-sender enable
        set network-overlay enable
        set network-id 2
        set tunnel-search nexthop
        set ipv4-start-ip 172.31.2.10
        set ipv4-end-ip 172.31.2.253
        set ipv4-netmask 255.255.255.0
        set psksecret fortinet
        set dpd-retryinterval 60
    next
end

```

### FW-SPOKE-01

```
config vpn ipsec phase1-interface
```

### FW-SPOKE-02

```
config vpn ipsec phase1-interface
```

## Validation

### FW-HUB-01
Routing table:
```
get router info routing-table all
```
VPN establishment:
```
diagnose vpn ike gateway list
```

### FW-SPOKE-XX
SD-WAN validation:
```
diagnose sys virtual-wan-link member
diagnose sys virtual-wan-link service
diagnose sys virtual-wan-link health-check
```
Routing table:
```
get router info routing-table all
get router info route-map-address
get router info bgp route-map <route-map-name>
```
VPN establishment:
```
diagnose vpn ike gateway list
```
