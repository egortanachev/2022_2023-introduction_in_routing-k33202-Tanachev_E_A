University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33202

Author: Tanachev Egor Antonovich

Lab: Lab2

Date of create: 01.11.2022

Date of finished: 13.01.2023

# Отчет по лабораторной работе №2 "Установка ContainerLab и развертывание тестовой сети связи"

**Цель работы:** ознакомиться с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств.

### Ход работы

**1. Текст файла, который был использовался для развертывания тестовой сети с расширением .yaml:**

```
name: lab2


mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24


topology:
  nodes:
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.2

    R01.BRL:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.3
    
    R01.FRT:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.4

    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.5

    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.6

    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.7


  links: 
    - endpoints: ["R01.MSK:eth2", "R01.FRT:eth2"]
    - endpoints: ["R01.MSK:eth1", "R01.BRL:eth1"]
    - endpoints: ["R01.BRL:eth2", "R01.FRT:eth1"]
    - endpoints: ["R01.MSK:eth3", "PC1:eth3"]
    - endpoints: ["R01.FRT:eth3", "PC2:eth3"]
    - endpoints: ["R01.BRL:eth3", "PC3:eth3"]
```

**2. Схема связи:**

![Communication scheme](assets/communication_scheme.jpg)

**3. Текст конфигураций сетевых устройств:**

**Роутер (R01.MSK)**

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool11 ranges=192.168.10.10-192.168.10.254 
/ip dhcp-server 
add address-pool=pool11 disabled=no interface=ether4 name=dhcp11 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.1.1/30 interface=ether2 network=10.10.1.0 
add address=10.10.2.1/30 interface=ether3 network=10.10.2.0 
add address=192.168.10.1/24 interface=ether4 network=192.168.10.0 
/ip dhcp-client 
add disabled=no interface=ether1 
/ip route 
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.2.2 
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.1.2 
/system identity 
set name=R01.MSK
```

**Роутер (R01.FRT)**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool22 ranges=192.168.20.10-192.168.20.254 
/ip dhcp-server 
add address-pool=pool22 disabled=no interface=ether4 name=dhcp22 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.2.2/30 interface=ether3 network=10.10.2.0 
add address=10.10.3.1/30 interface=ether2 network=10.10.3.0 
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0 
/ip dhcp-client 
add disabled=no interface=ether1 
/ip route 
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.2.1 
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.3.2 
/system identity 
set name=R01.FRT
```

**Роутер (R01.BRL)**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool33 ranges=192.168.30.10-192.168.30.254 
/ip dhcp-server 
add address-pool=pool33 disabled=no interface=ether4 name=dhcp33 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.1.2/30 interface=ether2 network=10.10.1.0 
add address=10.10.3.2/30 interface=ether3 network=10.10.3.0 
add address=192.168.30.1/24 interface=ether4 network=192.168.30.0 
/ip dhcp-client 
add disabled=no interface=ether2 
/ip route 
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.1.1 
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.3.1 
/system identity 
set name=R01.BRL
```

**PC1**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether4
/ip route 
add distance=1 dst-address=10.10.1.0/30 gateway=192.168.10.1 
add distance=1 dst-address=10.10.2.0/30 gateway=192.168.10.1 
add distance=1 dst-address=192.168.20.0/24 gateway=192.168.10.1 
add distance=1 dst-address=192.168.30.0/24 gateway=192.168.10.1 
/system identity 
set name=PC1 
```

**PC2**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether4
/ip route 
add distance=1 dst-address=10.10.2.0/30 gateway=192.168.20.1 
add distance=1 dst-address=10.10.3.0/30 gateway=192.168.20.1 
add distance=1 dst-address=192.168.10.0/24 gateway=192.168.20.1 
add distance=1 dst-address=192.168.30.0/24 gateway=192.168.20.1 
/system identity 
set name=PC2 
```

**PC3**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether4
/ip route 
add distance=1 dst-address=10.10.1.0/30 gateway=192.168.30.1 
add distance=1 dst-address=10.10.3.0/30 gateway=192.168.30.1 
add distance=1 dst-address=192.168.10.0/24 gateway=192.168.30.1 
add distance=1 dst-address=192.168.20.0/24 gateway=192.168.30.1 
/system identity 
set name=PC3 
```

**4. Результаты пингов для проверки локальной связности**



### Вывод

В ходе работы были получены практические навыки по работе с планированием IP адресов и настройке статической маршрутизации. Результатом стала развернутая тестовая сеть связи и настроенное оборудование.