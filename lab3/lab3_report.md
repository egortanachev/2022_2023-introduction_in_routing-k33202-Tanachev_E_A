University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33202

Author: Tanachev Egor Antonovich

Lab: Lab3

Date of create: 15.12.2022

Date of finished: 13.01.2023

# Отчёт по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

**Цель работы:** изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

### Ход работы

**1. Текст файла, который был использовался для развертывания тестовой сети с расширением .yaml:**

```
name: lab3


mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24


topology:
  nodes:
    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.2

    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.3
    
    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.4

    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.5

    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.6

    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.7

    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.8

    SGI_Prism:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6_47_9
      mgmt_ipv4: 172.20.20.9


links: 
  - endpoints: ["R01.SPB:eth2", "R01.MSK:eth2"]
  - endpoints: ["R01.SPB:eth1", "R01.HKI:eth1"]
  - endpoints: ["R01.MSK:eth3", "R01.LBN:eth3"]
  - endpoints: ["R01.HKI:eth3", "R01.LND:eth3"]
  - endpoints: ["R01.HKI:eth2", "R01.LBN:eth2"]
  - endpoints: ["R01.LND:eth2", "R01.NY:eth2"]
  - endpoints: ["R01.LBN:eth1", "R01.NY:eth1"]
  - endpoints: ["R01.SPB:eth3", "PC1:eth3"]
  - endpoints: ["R01.NY:eth3", "SGI_Prism:eth3"]
```

**2. Схема связи:**

![Communication scheme](assets/communication_scheme.jpg)

**3. Топология ContainerLab:**

![ContainerLab scheme](assets/containerlab_scheme.jpg)

**4. Текст конфигураций сетевых устройств:**

**Роутер (R01.SPB)**

```
Текст
```

**Роутер (R01.MSK)**

```
Текст
```

**Роутер (R01.HKI)**

```
Текст
```

**Роутер (R01.LND)**

```
Текст
```

**Роутер (R01.LBN)**

```
Текст
```

**Роутер (R01.NY)**

```
Текст
```

**PC1**

```
Текст
```

**SGI_Prism**

```
Текст
```

**5. Проверка локальной связности**

**Таблицы MPLS маршрутов на роутерах**

![Check MPLS](assets/check1.jpg)

**Пинг компьютера и сервера**

![Check PC1 and SGI_Prism](assets/check2.jpg)


### Вывод

В ходе работы были получены практические навыки по развертыванию сети связи и настройке OSPF, MPLS И EoMPLS.