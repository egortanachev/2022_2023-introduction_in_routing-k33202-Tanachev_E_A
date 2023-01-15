University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33202

Author: Tanachev Egor Antonovich

Lab: Lab4

Date of create: 6.01.2023

Date of finished: 13.01.2023

# Отчет по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

**Цель работы:** изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

### Ход работы

**1. Текст файла, который был использовался для развертывания тестовой сети с расширением .yaml:**

```
name: lab4


mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24


topology:
  nodes:
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.2

    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.3

    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.4

    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.5

    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.6

    R01.SVL:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.7

    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.8

    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.9

    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.10
  

links: 
  - endpoints: ["R01.NY:eth1","PC2:eth1"] 
  - endpoints: ["R01.NY:eth3","R01.LND:eth3"] 
  - endpoints: ["R01.LND:eth1","R01.HKI:eth2"]     
  - endpoints: ["R01.LND:eth2","R01.LBN:eth1"] 
  - endpoints: ["R01.LBN:eth2","R01.HKI:eth1"] 
  - endpoints: ["R01.LBN:eth3","R01.SVL:eth3"] 
  - endpoints: ["R01.SVL:eth1","PC3:eth1"]  
  - endpoints: ["R01.HKI:eth3","R01.SPB:eth3"]  
  - endpoints: ["R01.SPB:eth1","PC1:eth1"] 
```

**2. Схема связи:**

![Communication scheme](assets/communication_scheme.jpg)

**3. Топология ContainerLab:**

![ContainerLab scheme](assets/containerlab_scheme.jpg)

### Часть №1. Настройка L3VPN.

**4. Экспорты настроек устройств сети**

**Роутер (R01.NY)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default redistribute-connected=yes router-id=10.10.10.3
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.3 interface=Lo0 network=10.10.10.3
add address=192.168.20.10/24 interface=ether2 network=192.168.20.0
add address=10.10.3.1/30 interface=ether4 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.6 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

**Роутер (R01.LND)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.6
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.6 interface=Lo0 network=10.10.10.6
add address=10.10.4.1/30 interface=ether2 network=10.10.4.0
add address=10.10.6.2/30 interface=ether3 network=10.10.6.0
add address=10.10.3.2/30 interface=ether4 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.5 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.4 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.3 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

**Роутер (R01.HKI)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.4
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.4 interface=Lo0 network=10.10.10.4
add address=10.10.5.1/30 interface=ether2 network=10.10.5.0
add address=10.10.4.2/30 interface=ether3 network=10.10.4.0
add address=10.10.1.2/30 interface=ether4 network=10.10.1.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.6 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.5 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.1 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

**Роутер (R01.SPB)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.1
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.1 interface=Lo0 network=10.10.10.1
add address=10.10.1.1/30 interface=ether4 network=10.10.1.0
add address=192.168.10.10/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.4 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

**Роутер (R01.LBN)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.5
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.5 interface=Lo0 network=10.10.10.5
add address=10.10.6.1/30 interface=ether2 network=10.10.6.0
add address=10.10.2.2/30 interface=ether4 network=10.10.2.0
add address=10.10.5.2/30 interface=ether3 network=10.10.5.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.6 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.4 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.2 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

**Роутер (R01.SVL)**

```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.2
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.2 interface=Lo0 network=10.10.10.2
add address=10.10.2.1/30 interface=ether4 network=10.10.2.0
add address=192.168.30.10/24 interface=ether2 network=192.168.30.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.5 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```

**5. Результаты проверки VRF связанности**

![Check VRF](assets/check1.jpg)

### Часть №2. Настройка VPLS.

**6. Экспорты настроек устройств сети**

**Роутер (R01.NY)**

```
/interface bridge
add name=Lo0
add name=VPLS
/interface vpls
add disabled=no name=vpls3 remote-peer=10.10.10.1 vpls-id=10:0
add disabled=no name=vpls1 remote-peer=10.10.10.2 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.3
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.3
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=vpls1
add bridge=VPLS interface=vpls3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.3 interface=Lo0 network=10.10.10.3
add address=192.168.20.10/24 interface=ether2 network=192.168.20.0
add address=10.10.3.1/30 interface=ether4 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.6 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

**Роутер (R01.SPB)**

```
/interface bridge
add name=Lo0
add name=VPLS
/interface vpls
add disabled=no name=vpls2 remote-peer=10.10.10.3 vpls-id=10:0
add disabled=no name=vpls1 remote-peer=10.10.10.2 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.1
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.1
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=vpls1
add bridge=VPLS interface=vpls2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.1 interface=Lo0 network=10.10.10.1
add address=10.10.1.1/30 interface=ether4 network=10.10.1.0
add address=192.168.10.10/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.4 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

**Роутер (R01.SVL)**

```
/interface bridge
add name=Lo0
add name=VPLS
/interface vpls
add disabled=no name=vpls2 remote-peer=10.10.10.3 vpls-id=10:0
add disabled=no name=vpls3 remote-peer=10.10.10.1 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.2
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.2
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=vpls2
add bridge=VPLS interface=vpls3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.2 interface=Lo0 network=10.10.10.2
add address=10.10.2.1/30 interface=ether4 network=10.10.2.0
add address=192.168.30.10/24 interface=ether2 network=192.168.30.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.5 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```

**PC1**

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.1/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC1
```

**PC2**

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.2/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC2
```

**PC3**

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.3/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC3
```

**7. Результаты проверки связанности между компьютерами**

![Check PC](assets/check2.jpg)

### Вывод

В ходе работы были получены практические навыки работы с протоколами BGP, MPLS, а также были показаны правила организации L3VPN и VPLS.