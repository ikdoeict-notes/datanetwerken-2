# Herhalingsoefening

Ruben Thys - 2016-01-10

## Varia

`show controllers s0/0/0` > toon clock rate

`ip nat inside` ook voor subinterfaces

## DEEL 1

### CONFIGUREER DE SERIELE INTERFACES VAN DE 3 ROUTERS

@LONDEN

```
interface S0/0/0
ip address 10.192.16.2 255.255.255.252
no shutdown

interface s0/0/1
ip address 10.192.16.6 255.255.255.252
no shutdown

! Brussel - Londen
ping 10.192.16.1

! Parijs - Londen
ping 10.192.16.5
```

@BRUSSEL

```
interface S0/0/0
ip address 10.192.16.1 255.255.255.252
clock rate 4000000
no shutdown

interface S0/0/1
ip address 10.192.16.9 255.255.255.252
clock rate 4000000
no shutdown

! Brussel - Londen
ping 10.192.16.2

! Brussel - Parijs
ping 10.192.16.10
```

@PARIJS

```
interface S0/0/0
ip address 10.192.16.5 255.255.255.252
clock rate 4000000
no shutdown

interface S0/0/1
ip address 10.192.16.10 255.255.255.252
no shutdown

! Parijs - Londen
ping 10.192.16.6

! Brussel - Parijs
ping 10.192.16.9
```

## DEEL 2

### MAAK VLANS OP SWITCHES

```
enable
conf t
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
do show vlan br
```

### ASSIGN VLANS TO PORTS
### CREATE TRUNKS

@Access_1

```
interface range f0/1-5
switchport mode access
switchport access vlan 10
interface range f0/10-15
switchport mode access
switchport access vlan 30

interface range f0/6-9
switchport mode access
switchport access vlan 66
shutdown

interface range f0/16-23
switchport mode access
switchport access vlan 66
shutdown
do show vlan br

interface f0/24
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99

! Enable remote
interface vlan 40
ip address 172.16.40.1 255.255.255.0
no shutdown
exit
ip default-gateway 172.16.40.254

! Secure
interface f0/5
switchport mode access
switchport port-security
switchport port-security maximum 2
switchport port-security mac-address sticky
switchport port-security violation shutdown

interface f0/15
switchport mode access
switchport port-security
switchport port-security maximum 10
switchport port-security violation protect
show port-security interface f0/15
```

@Access_2

```
interface range f0/1-5
switchport mode access
switchport access vlan 20
interface range f0/10-15
switchport mode access
switchport access vlan 30

interface range f0/6-9
switchport mode access
switchport access vlan 66
shutdown

interface range f0/16-23
switchport mode access
switchport access vlan 66
shutdown
do show vlan br

interface f0/24
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99

! Enable remote
interface vlan 40
ip address 172.16.40.2 255.255.255.0
no shutdown
exit
ip default-gateway 172.16.40.254
```

@@@@ Distribution

```
interface f0/23
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
no shutdown

interface f0/24
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,40,99
no shutdown

interface g0/1
switchport mode trunk
no shutdown

! Enable remote
interface vlan 40
ip address 172.16.40.3 255.255.255.0
no shutdown
exit
ip default-gateway 172.16.40.254
```

@LONDEN

```
interface g0/0

interface g0/0.10
encapsulation dot1q 10
ip address 172.16.10.254 255.255.255.0

interface g0/0.20
encapsulation dot1q 20
ip address 172.16.20.254 255.255.255.0

interface g0/0.30
encapsulation dot1q 30
ip address 172.16.30.254 255.255.255.0

interface g0/0.40
encapsulation dot1q 40
ip address 172.16.40.254 255.255.255.0

interface g0/0.99
encapsulation dot1q 99 native
exit
interface g0/0
no shutdown
```

## DEEL 3

@BRUSSEL

```
interface g0/1
ip address 172.16.128.206 255.255.255.240
no shutdown
```

@PARIJS

```
interface g0/1
ip address 192.168.192.254 255.255.255.0
no shutdown

interface g0/0
ip address 192.168.250.254 255.255.255.128
no shutdown
```

## RIP

```
show ip protocols
show ip route
version 2 !sends subnetmasks in routing updates
```

@PARIJS

```
router rip
version 2
network 10.192.16.4
network 10.192.16.8
network 192.168.250.128
passive-interface g0/1
network 192.168.192.0
passive-interface g0/0
```

@LONDEN

```
router rip
version 2
network 10.192.16.4
network 10.192.16.0
network 172.16.10.0
passive-interface G0/0.10
network 172.16.20.0
passive-interface G0/0.20
network 172.16.30.0
passive-interface G0/0.30
network 172.16.40.0
passive-interface G0/0.40
```

@BRUSSEL

```
router rip
version 2
network 10.192.16.0
network 10.192.16.8
network 172.16.128.192
passive-interface G0/1

ip route 0.0.0.0 0.0.0.0 G0/0
router rip
default-information originate

! TELNET PC7 - LONDEN -
telnet 172.16.40.254
```

## DEEL 4

@BRUSSEL

```
interface g0/0
ip address 138.147.157.33 255.255.255.224
no shutdown
exit

ip nat inside source static 192.168.250.129 138.147.157.34
interface s0/0/1
ip nat inside
exit

interface G0/0
ip nat outside
exit

ip nat pool dynamisch 138.147.157.35 138.147.157.39 netmask 255.255.255.224
access-list 1 permit 172.16.0.0 0.0.63.255
ip nat inside source list 1 pool dynamisch

interface s0/0/0
ip nat inside
exit
```

## DEEL 4

SLAAC = Auto config ipv6

@BRUSSEL

```
ipv6 unicast-routing

interface S0/0/1
ipv6 rip RIPNG enable
ipv6 address 2001:db8:acad:a::1/64
exit

interface G0/1
ipv6 rip RIPNG enable
ipv6 address 2001:db8:acad:b::1/64
exit
```

## DEEL 5

@PARIJS
```
ipv6 unicast-routing

interface s0/0/1
ipv6 rip RIPNG enable
ipv6 address 2001:db8:acad:a::2/64
exit

interface G0/1
ipv6 rip RIPNG enable
ipv6 address 2001:db8:acad:c::1/64
```
