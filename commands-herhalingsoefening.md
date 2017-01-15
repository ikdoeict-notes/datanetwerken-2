# Commando's herhalingsoefening

Nicolas Henrotte

2016-01-10

## Deel 1

### Londen

    hostname Londen
    no ip domain-lookup
    enable secret cisco
    service password-encryption
    line con 0
    password class
    login
    line vty 0
    password class
    login
    banner motd %Geen toegang voor onbevoegden!%
    int s0/0/0
    ip address 10.192.16.2 255.255.255.252
    no shutdown
    int s0/0/1
    ip address 10.192.16.6 255.255.255.252
    no shutdown

### Parijs

    int s0/0/0
    ip address 10.192.16.5 255.255.255.252
    no shutdown
    clock rate 4000000
    int s0/0/1
    ip address 10.192.16.10 255.255.255.252
    no shutdown

### Brussel

    int s0/0/0
    ip address 10.192.16.1 255.255.255.252
    no shutdown
    clock rate 4000000
    int s0/0/1
    ip address 10.192.16.9 255.255.255.252
    no shutdown
    clock rate 4000000

## Deel 2 (alles gebeurt in het Londen-netwerk)

### Distribution, Access_1, Access_2

    vlan 10
    name Finance
    vlan 20
    name HR
    vlan 30
    name IT
    vlan 40
    name Management
    vlan 66
    name Black_hole

### Access_1

    int range fa0/1-5
    switchport mode access
    switchport access vlan 10
    int range fa0/10-15
    switchport mode access
    switchport access vlan 30

### Access_2

    int range fa0/1-5
    switchport mode access
    switchport access vlan 20
    int range fa0/10-15
    switchport mode access
    switchport access vlan 30

### Access_1 & Access_2

    int range fa0/6-9, fa0/16-23, g0/1-2
    switchport mode access
    switchport access vlan 66
    shutdown

### Distribution (trunking) (herhaal voor Distribution fa0/24, Access_1 fa0/24, Access_2 fa0/24)

    int fa0/23
    switchport mode trunk
    switchport trunk native vlan 99
    switchport trunk allowed vlan 10,20,30,40,99

### Access_1

    int vlan 1
    no ip address
    int vlan 40
    ip address 172.16.40.1 255.255.255.0
    exit
    ip default-gateway 172.16.40.254

### Access_2

    int vlan 1
    no ip address
    int vlan 40
    ip address 172.16.40.2 255.255.255.0
    exit
    ip default-gateway 172.16.40.254

### Distribution

    int vlan 1
    no ip address
    int vlan 40
    ip address 172.16.40.3 255.255.255.0
    exit
    ip default-gateway 172.16.40.254

### PC3, 4, 5, 6

Stel de IP adressen en default gateways in volgens de bijgevoegde lijst

### Access_1

    int fa0/5
    shutdown
    switchport port-security
    switchport port-security maximum 2
    switchport port-security mac-address sticky
    int fa0/15
    shutdown
    switchport port-security
    switchport port-security maximum 10
    switchport port-security violation protect

## Deel 3


### Brussel

    int g0/0
    ip address 138.147.157.33 255.255.255.224
    no shutdown
    int g0/1
    ip address 172.16.128.206 255.255.255.240
    no shutdown
    router rip
    version 2
    network 10.192.16.0
    network 172.16.128.0
    passive-interface g0/1
    no auto-summary

### Parijs

    int g0/0
    ip address 192.168.250.129 255.255.255.128
    no shutdown
    int g0/1
    ip address 192.168.192.254 255.255.255.0
    no shutdown
    router rip
    version 2
    network 10.192.16.0
    network 192.168.250.0
    network 192.168.192.0
    passive-interface g0/1
    passive-interface g0/0
    no auto-summary

### Londen

    int g0/0.10
    encapsulation dot1Q 10
    ip address 172.16.10.254 255.255.255.0
    int g0/0.20
    encapsulation dot1Q 20
    ip address 172.16.20.254 255.255.255.0
    int g0/0.30
    encapsulation dot1Q 30
    ip address 172.16.30.254 255.255.255.0
    int g0/0.40
    encapsulation dot1Q 40
    ip address 172.16.40.254 255.255.255.0
    int g0/0
    no shutdown
    router rip
    version 2
    network 10.192.16.0
    network 172.16.0.0
    passive-interface g0/0.10
    passive-interface g0/0.20
    passive-interface g0/0.30
    passive-interface g0/0.40
    no auto-summary

### Brussel

    ip route 0.0.0.0 0.0.0.0 g0/0
    router rip
    default-information originate

### PC1, 2, 7 & Server1

Stel de IP adressen en default gateways in volgens de bijgevoegde lijst

## Deel 4


### Brussel

    int g0/0
    ip nat outside
    int s0/0/1
    ip nat inside
    ip nat inside source static 192.168.250.129 138.147.157.34
    int g0/0
    ip address 138.147.157.33 255.255.255.224
    no shutdown
    ip nat pool dynamisch 138.147.157.35 138.147.157.39 netmask 255.255.255.224
    access-list 1 permit 172.16.0.0 0.0.63.255
    ip nat inside source list 1 pool dynamisch
    int s0/0/0
    ip nat inside

## Deel 5


### Brussel

    int s0/0/1
    ipv6 address 2001:db8:acad:a::1/64
    ipv6 rip RIPNG enable
    int g0/1
    ipv6 address 2001:db8:acad:b::1/64
    ipv6 rip RIPNG enable
    exit
    ipv6 unicast-routing

### Parijs

    int s0/0/1
    ipv6 address 2001:db8:acad:a::2/64
    ipv6 rip RIPNG enable
    int g0/1
    ipv6 address 2001:db8:acad:c::1/64
    ipv6 rip RIPNG enable
    exit
    ipv6 unicast-routing
