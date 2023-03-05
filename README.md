# SD-WAN

## Test Environment

| Name        | FortiOS |        ISP1        |        ISP2        | LAN                        |
|-------------|:-------:|:-------------------:|:-------------------:|----------------------------|
| FW-HUB-01   |  7.2.3  | 100.100.100.100 & 172.22.1.11 \| wan1 | - | 10.1.30.254 \| 10.1.37.254 |
| FW-SPOKE-01 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.21.254             |
| FW-SPOKE-02 |  7.2.3  |     DHCP \| wan1    |    DHCP \| port3    | 192.168.22.254             |

| BGP AS        | 65000 |  |
|-------------|:-------:|:-------:|
| Address Range for VPNs   |  172.31.0.0/17 | /17 for 250 Spokes, /14 for 2000 Spokes |

/!\ TODO:
- SD-WAN with AZURE

### FW-HUB-01

```
config system settings
    set tcp-session-without-syn enable
end
config vpn ipsec phase1-interface
    edit "DC01-ISP1"
        set type dynamic
        set interface "wan1"
        set local-gw 172.22.1.11
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
        set ipv4-start-ip 172.31.1.10
        set ipv4-end-ip 172.31.1.253
        set ipv4-netmask 255.255.255.0
        set psksecret fortinet
        set dpd-retryinterval 60
    next
    edit "DC01-ISP2"
        set type dynamic
        set interface "wan1"
        set local-gw 172.22.1.11
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
        set ipv4-start-ip 172.31.2.10
        set ipv4-end-ip 172.31.2.253
        set ipv4-netmask 255.255.255.0
        set psksecret fortinet
        set dpd-retryinterval 60
    next
end
config vpn ipsec phase2-interface
    edit "DC01-ISP1_p2"
        set phase1name "DC01-ISP1"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
    edit "DC01-ISP2_p2"
        set phase1name "DC01-ISP2"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
end
config system interface
    edit "VPNLoop"
        set vdom "root"
        set type loopback
        set allowaccess ping
        set ip 172.31.127.254 255.255.255.255
    next
    edit "DC01-ISP1"
        set vdom "root"
        set ip 172.31.1.254 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 172.31.1.10 255.255.255.0
        set interface "wan1"
    next
    edit "DC01-ISP2"
        set vdom "root"
        set ip 172.31.2.254 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 172.31.2.10 255.255.255.0
        set interface "wan1"
    next
end
config router bgp
    set as 65000
    set ibgp-multipath enable
    set additional-path enable
    set additional-path-select 4
    config neighbor-group
        edit "DC01-ISP1"
            set advertisement-interval 1
            set link-down-failover enable
            set next-hop-self enable
            set soft-reconfiguration enable
            set interface "DC01-ISP1"
            set remote-as 65000
            set update-source "DC01-ISP1"
            set additional-path send
            set adv-additional-path 4
            set route-reflector-client enable
        next
        edit "DC01-ISP2"
            set advertisement-interval 1
            set link-down-failover enable
            set next-hop-self enable
            set soft-reconfiguration enable
            set interface "DC01-ISP2"
            set remote-as 65000
            set update-source "DC01-ISP2"
            set additional-path send
            set adv-additional-path 4
            set route-reflector-client enable
        next
    end
    config neighbor-range
        edit 0
            set prefix 172.31.1.0 255.255.255.0
            set neighbor-group "DC01-ISP1"
        next
        edit 0
            set prefix 172.31.2.0 255.255.255.0
            set neighbor-group "DC01-ISP2"
        next
    end
    config network
        edit 0
            set prefix 10.1.30.0 255.255.255.0
        next
        edit 0
            set prefix 10.1.37.0 255.255.255.0
        next
    end
end
config firewall address
    edit "RFC_1918_10"
        set subnet 10.0.0.0 255.0.0.0
    next
    edit "RFC_1918_172_16"
        set subnet 172.16.0.0 255.240.0.0
    next
    edit "RFC_1918_192_168"
        set subnet 192.168.0.0 255.255.0.0
    next
    edit "SRV-37"
        set color 9
        set subnet 10.1.37.0 255.255.255.0
    next
        edit "SRV-30"
        set color 21
        set subnet 10.1.30.0 255.255.255.0
    next
        edit "SRV-AZURE"
        set color 18
        set subnet 10.0.0.0 255.255.0.0
    next
        edit "SSLVPN_TUNNEL_ADDR1"
        set type iprange
        set start-ip 10.212.134.10
        set end-ip 10.212.134.210
    next
end
config firewall addrgrp
    edit "RFC_1918_ALL"
        set member "RFC_1918_10" "RFC_1918_172_16" "RFC_1918_192_168"
    next
    edit "RFC_1918_DC01"
        set member "SRV-37" "SRV-30" "RFC_1918_172_16" "RFC_1918_192_168" "SSLVPN_TUNNEL_ADDR1"
    next
end
config router policy
    edit 0
        set input-device "DC01-ISP1"
        set output-device "DC01-ISP1"
    next
    edit 0
        set input-device "DC01-ISP2"
        set output-device "DC01-ISP2"
    next
end
config firewall policy
    edit 0
        set name "ADVPN Spoke to Spoke"
        set srcintf "DC01-ISP1" "DC01-ISP2"
        set dstintf "DC01-ISP1" "DC01-ISP2"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
        set anti-replay disable
        set tcp-session-without-syn all        
    next
    edit 0
        set name "ADVPN Out"
        set srcintf "any"
        set dstintf "DC01-ISP1" "DC01-ISP2"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "ADVPN In"
        set srcintf "DC01-ISP1" "DC01-ISP2"
        set dstintf "any"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "ADVPN Hub HC"
        set srcintf "DC01-ISP1" "DC01-ISP2"
        set dstintf "VPNLoop"
        set srcaddr "all"
        set dstaddr "Hub-HC"
        set action accept
        set schedule "always"
        set service "ALL"
    next        
end
```

### FW-SPOKE-01

```
config vpn ipsec phase1-interface
    edit "DC01-ISP1"
        set interface "wan1"
        set ike-version 2
        set keylife 28800
        set peertype any
        set net-device enable
        set mode-cfg enable
        set localid "FW-SPOKE-01-ISP1"
        set proposal aes256-sha256 aes256gcm-prfsha384
        set add-route disable
        set dpd on-idle
        set idle-timeout enable
        set idle-timeoutinterval 5
        set auto-discovery-receiver enable
        set network-overlay enable
        set network-id 1
        set remote-gw 172.22.1.11
        set psksecret fortinet
        set dpd-retrycount 2
        set dpd-retryinterval 10
    next                    
    edit "DC01-ISP2"
        set interface "port3"
        set ike-version 2
        set keylife 28800
        set peertype any
        set net-device enable
        set mode-cfg enable
        set localid "FW-SPOKE-01-ISP2"
        set proposal aes256-sha256 aes256gcm-prfsha384
        set add-route disable
        set dpd on-idle
        set idle-timeout enable
        set idle-timeoutinterval 5
        set auto-discovery-receiver enable
        set network-overlay enable
        set network-id 2
        set remote-gw 172.22.1.11
        set psksecret fortinet
        set dpd-retrycount 2
        set dpd-retryinterval 10
    next                    
end
config vpn ipsec phase2-interface
    edit "DC01-ISP1_p2"
        set phase1name "DC01-ISP1"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
    edit "DC01-ISP2_p2"
        set phase1name "DC01-ISP2"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
end
config system interface
    edit "DC01-ISP1"
        set allowaccess ping
    next                    
    edit "DC01-ISP2"
        set allowaccess ping
    next                    
end
config router bgp
    set as 65000
    set ibgp-multipath enable
    set additional-path enable
    set additional-path-select 4
    set keepalive-timer 5
    set holdtime-timer 15
    config neighbor
        edit "172.31.1.254"
            set advertisement-interval 1
            set link-down-failover enable
            set soft-reconfiguration enable
            set interface "DC01-ISP1"
            set remote-as 65000
            set connect-timer 1
            set additional-path receive
        next
        edit "172.31.2.254"
            set advertisement-interval 1
            set link-down-failover enable
            set soft-reconfiguration enable
            set interface "DC01-ISP2"
            set remote-as 65000
            set connect-timer 1
            set additional-path receive
        next
    end
    config network
        edit 0
            set prefix 192.168.21.0 255.255.255.0
        next
    end
end
config firewall address
    edit "RFC_1918_10"
        set subnet 10.0.0.0 255.0.0.0
    next
    edit "RFC_1918_172_16"
        set subnet 172.16.0.0 255.240.0.0
    next
    edit "RFC_1918_192_168"
        set subnet 192.168.0.0 255.255.0.0
    next
    edit "SRV-37"
        set color 9
        set subnet 10.1.37.0 255.255.255.0
    next
        edit "SRV-30"
        set color 21
        set subnet 10.1.30.0 255.255.255.0
    next
        edit "SRV-AZURE"
        set color 18
        set subnet 10.0.0.0 255.255.0.0
    next
        edit "SSLVPN_TUNNEL_ADDR1"
        set type iprange
        set start-ip 10.212.134.10
        set end-ip 10.212.134.210
    next
end
config firewall addrgrp
    edit "RFC_1918_ALL"
        set member "RFC_1918_10" "RFC_1918_172_16" "RFC_1918_192_168"
    next
    edit "RFC_1918_DC01"
        set member "SRV-37" "SRV-30" "RFC_1918_172_16" "RFC_1918_192_168" "SSLVPN_TUNNEL_ADDR1"
    next
end
config system sdwan
    set status enable
    config zone
        edit "SD-WAN DC01"
        next
    end
    config zone
        edit "SD-WAN INTERNET"
        next
    end
end
config router static
    edit 1
        set distance 1
        set sdwan-zone "SD-WAN INTERNET"
    next
end
config system sdwan
    config members
        edit 0
            set interface "DC01-ISP1"
            set zone "SD-WAN DC01"
        next
        edit 0
            set interface "DC01-ISP2"
            set zone "SD-WAN DC01"
            set cost 10
            set priority 10
        next
        edit 0
            set interface "wan1"
            set zone "SD-WAN INTERNET"
            set cost 10
            set priority 10
        next
        edit 0
            set interface "port3"
            set zone "SD-WAN INTERNET"
        next
    end
    config health-check
        edit "SLA_DC01"
            set server "172.31.127.254"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 1 2
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
        edit "SLA_Google"
            set server "8.8.8.8"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 3 4
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
        edit "SLA_Cloudflare"
            set server "1.1.1.1"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 3 4
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
    end
    config service
        edit 0
            set name "DC01-Traffic"
            set mode sla
            set dst "RFC_1918_DC01"
            set src "RFC_1918_DC01"
            set hold-down-time 20
            config sla
                edit "SLA_DC01"
                    set id 1
                next
            end
            set priority-members 1 2
        next
        edit 0
            set name "Internet-Traffic"
            set mode sla
            set dst "all"
            set src "all"
            config sla
                edit "SLA_Cloudflare"
                    set id 1
                next
                edit "SLA_Google"
                    set id 1
                next
            end
            set priority-members 3 4
        next
    end
end
config firewall policy
    edit 0
        set name "ADVPN Out"
        set srcintf "any"
        set dstintf "SD-WAN DC01"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "ADVPN In"
        set srcintf "SD-WAN DC01"
        set dstintf "any"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "LAN-2-SDWAN-INTERNET"
        set srcintf "port1"
        set dstintf "SD-WAN INTERNET"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

### FW-SPOKE-02

```
config vpn ipsec phase1-interface
    edit "DC01-ISP1"
        set interface "wan1"
        set ike-version 2
        set keylife 28800
        set peertype any
        set net-device enable
        set mode-cfg enable
        set localid "FW-SPOKE-02-ISP1"
        set proposal aes256-sha256 aes256gcm-prfsha384
        set add-route disable
        set dpd on-idle
        set idle-timeout enable
        set idle-timeoutinterval 5
        set auto-discovery-receiver enable
        set network-overlay enable
        set network-id 1
        set remote-gw 172.22.1.11
        set psksecret fortinet
        set dpd-retrycount 2
        set dpd-retryinterval 10
    next                    
    edit "DC01-ISP2"
        set interface "port3"
        set ike-version 2
        set keylife 28800
        set peertype any
        set net-device enable
        set mode-cfg enable
        set localid "FW-SPOKE-02-ISP2"
        set proposal aes256-sha256 aes256gcm-prfsha384
        set add-route disable
        set dpd on-idle
        set idle-timeout enable
        set idle-timeoutinterval 5
        set auto-discovery-receiver enable
        set network-overlay enable
        set network-id 2
        set remote-gw 172.22.1.11
        set psksecret fortinet
        set dpd-retrycount 2
        set dpd-retryinterval 10
    next                    
end
config vpn ipsec phase2-interface
    edit "DC01-ISP1_p2"
        set phase1name "DC01-ISP1"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
    edit "DC01-ISP2_p2"
        set phase1name "DC01-ISP2"
        set proposal aes256-sha256 aes256gcm
        set keepalive enable
        set keylifeseconds 1800
    next
end
config system interface
    edit "DC01-ISP1"
        set allowaccess ping
    next                    
    edit "DC01-ISP2"
        set allowaccess ping
    next                    
end
config router bgp
    set as 65000
    set ibgp-multipath enable
    set additional-path enable
    set additional-path-select 4
    set keepalive-timer 5
    set holdtime-timer 15
    config neighbor
        edit "172.31.1.254"
            set advertisement-interval 1
            set link-down-failover enable
            set soft-reconfiguration enable
            set interface "DC01-ISP1"
            set remote-as 65000
            set connect-timer 1
            set additional-path receive
        next
        edit "172.31.2.254"
            set advertisement-interval 1
            set link-down-failover enable
            set soft-reconfiguration enable
            set interface "DC01-ISP2"
            set remote-as 65000
            set connect-timer 1
            set additional-path receive
        next
    end
    config network
        edit 0
            set prefix 192.168.22.0 255.255.255.0
        next
    end
end
config firewall address
    edit "RFC_1918_10"
        set subnet 10.0.0.0 255.0.0.0
    next
    edit "RFC_1918_172_16"
        set subnet 172.16.0.0 255.240.0.0
    next
    edit "RFC_1918_192_168"
        set subnet 192.168.0.0 255.255.0.0
    next
    edit "SRV-37"
        set color 9
        set subnet 10.1.37.0 255.255.255.0
    next
        edit "SRV-30"
        set color 21
        set subnet 10.1.30.0 255.255.255.0
    next
        edit "SRV-AZURE"
        set color 18
        set subnet 10.0.0.0 255.255.0.0
    next
        edit "SSLVPN_TUNNEL_ADDR1"
        set type iprange
        set start-ip 10.212.134.10
        set end-ip 10.212.134.210
    next
end
config firewall addrgrp
    edit "RFC_1918_ALL"
        set member "RFC_1918_10" "RFC_1918_172_16" "RFC_1918_192_168"
    next
    edit "RFC_1918_DC01"
        set member "SRV-37" "SRV-30" "RFC_1918_172_16" "RFC_1918_192_168" "SSLVPN_TUNNEL_ADDR1"
    next
end
config system sdwan
    set status enable
    config zone
        edit "SD-WAN DC01"
        next
    end
    config zone
        edit "SD-WAN INTERNET"
        next
    end
end
config router static
    edit 1
        set distance 1
        set sdwan-zone "SD-WAN INTERNET"
    next
end
config system sdwan
    config members
        edit 0
            set interface "DC01-ISP1"
            set zone "SD-WAN DC01"
        next
        edit 0
            set interface "DC01-ISP2"
            set zone "SD-WAN DC01"
            set cost 10
            set priority 10
        next
        edit 0
            set interface "wan1"
            set zone "SD-WAN INTERNET"
            set cost 10
            set priority 10
        next
        edit 0
            set interface "port3"
            set zone "SD-WAN INTERNET"
        next
    end
    config health-check
        edit "SLA_DC01"
            set server "172.31.127.254"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 1 2
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
        edit "SLA_Google"
            set server "8.8.8.8"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 3 4
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
        edit "SLA_Cloudflare"
            set server "1.1.1.1"
            set sla-fail-log-period 10
            set sla-pass-log-period 10
            set members 3 4
            config sla
                edit 1
                    set latency-threshold 200
                    set jitter-threshold 20
                    set packetloss-threshold 2
                next
            end
        next
    end
    config service
        edit 0
            set name "DC01-Traffic"
            set mode sla
            set dst "RFC_1918_DC01"
            set src "RFC_1918_DC01"
            set hold-down-time 20
            config sla
                edit "SLA_DC01"
                    set id 1
                next
            end
            set priority-members 1 2
        next
        edit 0
            set name "Internet-Traffic"
            set mode sla
            set dst "all"
            set src "all"
            config sla
                edit "SLA_Cloudflare"
                    set id 1
                next
                edit "SLA_Google"
                    set id 1
                next
            end
            set priority-members 3 4
        next
    end
end
config firewall policy
    edit 0
        set name "ADVPN Out"
        set srcintf "any"
        set dstintf "SD-WAN DC01"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "ADVPN In"
        set srcintf "SD-WAN DC01"
        set dstintf "any"
        set srcaddr "RFC_1918_DC01"
        set dstaddr "RFC_1918_DC01"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 0
        set name "LAN-2-SDWAN-INTERNET"
        set srcintf "port1"
        set dstintf "SD-WAN INTERNET"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

## Validation

### FW-HUB-01
Routing table:
```
get router info routing-table static
get router info routing-table connected
get router info routing-table all
```
VPN establishment:
```
diagnose vpn ike gateway list
```

### FW-SPOKE-XX
SD-WAN validation:
```
FortiOS 6.2.X
diagnose sys virtual-wan-link member
diagnose sys virtual-wan-link service
diagnose sys virtual-wan-link health-check

FortiOS 7.2.X
diagnose sys sdwan member
diagnose sys sdwan service
diagnose sys sdwan health-check

```
Routing table:
```
get router info routing-table static
get router info routing-table connected
get router info routing-table all

get router info bgp summary
get router info routing-table bgp
```
VPN establishment:
```
diagnose vpn ike gateway list
```

PING internal server:
```
ping 10.1.37.254
ping 10.1.37.100

ping 10.1.30.254
ping 10.1.30.10
```

## DEBUG
BGP:
```
diag ip router bgp level info
diag ip router bgp all enable
diag ip router bgp show
diag debug enable

diag debug reset
diag debug disable
```

IPSEC:
```
diag vpn ike log filter name <phase1-name>
diag vpn ike log filter name MyVPN

diag debug app ike -1
diag debug enable

diag vpn ike log-filter clear
diag debug disable

diag vpn ike restart
diag vpn ike gateway clear
```

IPSEC PSK Extract plaintext:
```
https://[Address]:[port]/api/v2/cmdb/vpn.ipsec/phase1-interface?plain-text-password=1
```

ADVPN:
```
diag debug reset
diag debug application ike -1
diag debug console timestamp enable
diag debug en

looking for "adding new dynamic"
```

## TIPS
Disabling SIP ALG:
```
config system settings
    set sip-expectation disable
    set sip-nat-trace disable
    set default-voip-alg-mode kernel-helper-based
end

config system session-helper
    show 
    # find the entry for SIP
    delete XX
end

config voip profile
    edit default
    config sip
        set rtp disable
    end
end
execute reboot
```
