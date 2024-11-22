# Архитектура сети. Проектирование сети
## Цель
В данной самостоятельной работе необходимо распланировать адресное пространство Настроить IP на всех активных портах для дальнейшей работы над проектом Адресное пространство должно быть задокументировано.

 Описание/Пошаговая инструкция выполнения домашнего задания: 

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

  1)	Разработаете и задокументируете адресное пространство для лабораторного стенда. 
  2)	Настроите ip адреса на каждом активном порту 
  3)	Настроите каждый VPC в каждом офисе в своем VLAN.
  4)	Настроите VLAN/Loopbackup interface управления для сетевых устройств 
  5)	Настроите сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано 
  6)	Используете IPv4. IPv6 по желанию

## Топология сети
![alt text](image.png)
## Сети Москвы (AS 1001)

### Таблица подсетей

| **Подсеть**     | **Маска**       | **Назначение**                            | **Пояснение**                                                             |
|------------------|-----------------|-------------------------------------------|---------------------------------------------------------------------------|
| `10.1.1.0/24`   | `255.255.255.0` | VLAN 10 — Управление                     | Используется для управления коммутаторами и маршрутизаторами.             |
| `10.1.2.0/24`   | `255.255.255.0` | VLAN 20 — Конечные устройства            | Сеть конечных устройств (например, VPC1 и VPC7) с отказоустойчивым шлюзом (VRRP). |
| `10.1.3.0/30`   | `255.255.255.252` | Соединение между R12 и R14              | Подсеть для точечного соединения между маршрутизаторами.                  |
| `10.1.3.4/30`   | `255.255.255.252` | Соединение между R12 и R15              | Аналогично другим соединениям.                                           |
| `10.1.3.8/30`   | `255.255.255.252` | Соединение между R13 и R14              | Подсеть для прямого соединения R13 ↔ R14.                                |
| `10.1.3.12/30`  | `255.255.255.252` | Соединение между R13 и R15              | Аналогично другим межмаршрутизаторным соединениям.                       |
| `10.1.3.16/30`  | `255.255.255.252` | Соединение между R14 и R19              | Точечное соединение для внутренней связи маршрутизаторов AS 1001.         |
| `10.1.3.20/30`  | `255.255.255.252` | Соединение между R15 и R20              | Аналогично соединению R15 ↔ R20.                                         |
| `10.1.255.0/24` | `255.255.255.0` | Loopback-интерфейсы маршрутизаторов      | Выделенные адреса для OSPF Router ID и BGP update-source.                |
| `198.18.0.0/30` | `255.255.255.252` | Соединение между R14 и AS 101 (R22)     | Зарезервированная сеть для тестирования взаимодействия между AS.          |
| `198.18.0.4/30` | `255.255.255.252` | Соединение между R15 и AS 301 (R21)     | Аналогично, для связи с AS 301.                                          |

---

### Пояснения
1. **VLAN 10:** Управление сетевыми устройствами для упрощения администрирования.
2. **VLAN 20:** Сеть конечных устройств с отказоустойчивым VRRP-шлюзом.
3. **Подсети /30:** Минимизируют расход адресного пространства.
4. **Loopback:** Стабильные адреса для маршрутизации и управления.
5. **Меж-AS соединения:** Используют тестовые диапазоны для эмуляции взаимодействия автономных систем.
6. Все локальные подсети Москвы (AS 1001) объединяются в одну суммарную сеть:
- **Суммарная сеть:** `10.1.0.0/16`.
- **Объединяет:**
  - VLAN 10: `10.1.1.0/24`.
  - VLAN 20: `10.1.2.0/24`.
  - Точечные соединения: `10.1.3.0/30`, `10.1.3.4/30` и так далее.
  - Loopback-интерфейсы: `10.1.255.0/24`.

**Преимущества:**
- Упрощение таблицы маршрутизации.
- Сокращение числа маршрутов, передаваемых в другие автономные системы.
- Оптимизация поиска маршрутов внутри AS 1001.

## IP адресация Москвы (AS 1001) 

- Управление через VLAN 10 (управляемые устройства и VRRP).
- Конечные устройства в VLAN 20 с отказоустойчивым шлюзом (VRRP).
- Точечные соединения между маршрутизаторами.
- Loopback-интерфейсы для маршрутизации и управления.
- Соединения с соседними автономными системами (AS 101 и AS 301) через NAT.

| **Устройство** | **Интерфейс** | **IP-адрес**   | **Маска**       | **Назначение**                                |
|----------------|---------------|----------------|-----------------|----------------------------------------------|
| **R12**        | e0/0.10       | 10.1.1.1       | 255.255.255.0   | VLAN 10 — Управление (VRRP Master)           |
|                | e0/0.20       | 10.1.2.1       | 255.255.255.0   | VLAN 20 — Конечные устройства (VRRP Master)  |
|                | e0/0.99       | -              | -               | Native                                       |
|                | e0/2          | 10.1.3.1       | 255.255.255.252 | Соединение с R14                             |
|                | e0/3          | 10.1.3.5       | 255.255.255.252 | Соединение с R15                             |
|                | Loopback0     | 10.1.255.1     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **R13**        | e0/0.10       | 10.1.1.2       | 255.255.255.0   | VLAN 10 — Управление (VRRP Backup)           |
|                | e0/0.20       | 10.1.2.2       | 255.255.255.0   | VLAN 20 — Конечные устройства (VRRP Backup)  |
|                | e0/0.99       | -              | -               | Native                                       |
|                | e0/2          | 10.1.3.9       | 255.255.255.252 | Соединение с R14                             |
|                | e0/3          | 10.1.3.13      | 255.255.255.252 | Соединение с R15                             |
|                | Loopback0     | 10.1.255.2     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **R14**        | e0/0          | 10.1.3.2       | 255.255.255.252 | Соединение с R12                             |
|                | e0/1          | 10.1.3.10      | 255.255.255.252 | Соединение с R13                             |
|                | e0/2          | 198.18.0.1     | 255.255.255.252 | Соединение с AS 101 (R22)                    |
|                | e0/3          | 10.1.3.17      | 255.255.255.252 | Соединение с R19                             |
|                | Loopback0     | 10.1.255.3     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **R15**        | e0/0          | 10.1.3.14      | 255.255.255.252 | Соединение с R13                             |
|                | e0/1          | 10.1.3.6       | 255.255.255.252 | Соединение с R12                             |
|                | e0/2          | 198.18.0.5     | 255.255.255.252 | Соединение с AS 301 (R21)                    |
|                | e0/3          | 10.1.3.21      | 255.255.255.252 | Соединение с R20                             |
|                | Loopback0     | 10.1.255.4     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **R19**        | e0/0          | 10.1.3.18      | 255.255.255.252 | Соединение с R14                             |
|                | Loopback0     | 10.1.255.5     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **R20**        | e0/0          | 10.1.3.22      | 255.255.255.252 | Соединение с R15                             |
|                | Loopback0     | 10.1.255.6     | 255.255.255.255 | Loopback для управления и маршрутизации      |
| **SW2**        | VLAN 10       | 10.1.1.3       | 255.255.255.0   | Управляемый IP для SW2                       |
| **SW3**        | VLAN 10       | 10.1.1.4       | 255.255.255.0   | Управляемый IP для SW3                       |
| **SW4**        | VLAN 10       | 10.1.1.5       | 255.255.255.0   | Управляемый IP для SW4                       |
| **SW5**        | VLAN 10       | 10.1.1.6       | 255.255.255.0   | Управляемый IP для SW5                       |
| **VPC1**       | eth0          | 10.1.2.100     | 255.255.255.0   | Конечное устройство в VLAN 20                |
| **VPC7**       | eth0          | 10.1.2.101     | 255.255.255.0   | Конечное устройство в VLAN 20                |

---

## Пояснения

### 1. VLAN 10 (10.1.1.0/24)
- Используется для управления коммутаторами и маршрутизаторами.
- **VRRP:**
  - **Master:** R12 (`10.1.1.1`).
  - **Backup:** R13 (`10.1.1.2`).
  - **Виртуальный шлюз (VRRP):** `10.1.1.254`.

### 2. VLAN 20 (10.1.2.0/24)
- Подсеть для конечных устройств (VPC1, VPC7).
- **VRRP:**
  - **Master:** R12 (`10.1.2.1`).
  - **Backup:** R13 (`10.1.2.2`).
  - **Виртуальный шлюз (VRRP):** `10.1.2.254`.

### 3. Точечные соединения (10.1.3.0/30 - 10.1.3.20/30)
- Соединяют маршрутизаторы внутри AS 1001.
- Маска `/30` минимизирует расход IP-адресов.

### 4. Loopback-интерфейсы (10.1.255.0/24)
- Выделены для стабильных IP-адресов маршрутизаторов, используемых в OSPF и BGP.

### 5. Меж-AS соединения (198.18.0.0/30 и 198.18.0.4/30)
- Используются для подключения к AS 101 и AS 301.
- Зарезервированный диапазон (RFC 2544) для тестирования и эмуляций.

## Таблица VIP-адресов VRRP

| **VLAN**         | **VIP-адрес**   | **Назначение**             | **Пояснение**                                  |
|-------------------|-----------------|----------------------------|-----------------------------------------------|
| VLAN 10           | `10.1.1.254`   | Управление                 | Виртуальный шлюз для управления сетевыми устройствами. |
| VLAN 20           | `10.1.2.254`   | Конечные устройства        | Виртуальный шлюз для конечных устройств в отказоустойчивой сети. |


Настройка локальной сети для R12 и R13.

R12:
```
Router>en
Router#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hos
Router(config)#hostname R12
Router(config)#no ip domain-lookup
R12(config)#interface e0/0
R12(config-if)#no shut
R12(config-if)#exit
R12(config)#interface e0/0.10
R12(config-subif)#encapsulation dot1Q 10
R12(config-subif)#ip address 10.1.1.1 255.255.255.0
R12(config-subif)#description LAN_MANAGEMENT
R12(config-subif)#interface e0/0.20
R12(config-subif)#encapsulation dot1Q 20
R12(config-subif)#description LAN_SEGMENT
R12(config-subif)#ip address 10.1.2.1 255.255.255.0
R12(config)#interface e0/0.99
R12(config-subif)#encapsulation dot1Q 99 native
R12(config-subif)#description NATIVE
R12(config-subif)#interface e0/2
R12(config-if)#ip address 10.1.3.1 255.255.255.252
R12(config-if)#description TO_R14
R12(config-if)#no shut
R12(config-if)#interface e0/3
R12(config-if)#ip address 10.1.3.5 255.255.255.252
R12(config-if)#description TO_R15
R12(config-if)#no shut
R12(config-if)#interface l0
R12(config-if)#ip address 10.1.255.1 255.255.255.255
Настройка VRRP
R12(config-if)#interface e0/0.10
R12(config-subif)#vrrp 10 ip 10.1.1.254
R12(config-subif)#vrrp 10 priority 110
R12(config-subif)#vrrp 10 preempt
R12(config-subif)#vrrp 10 authentication md5 key-string PASSWORD_MANAGEMENT
R12(config-subif)#interface e0/0.20
R12(config-subif)#vrrp 20 ip 10.1.2.254
R12(config-subif)# vrrp 20 priority 110
R12(config-subif)#vrrp 20 preempt
R12(config-subif)#vrrp 20 authentication md5 key-string PASSWORD_LAN
R12(config-subif)#do wr
Building configuration...
[OK]
```

Настройка R13:
```
en
conf t
hostname R13
no ip domain-lookup
interface e0/0
no shut
exit
interface e0/0.10
encapsulation dot1Q 10
ip address 10.1.1.2  255.255.255.0
description LAN_MANAGEMENT
interface e0/0.20
encapsulation dot1Q 20
description LAN_SEGMENT
ip address 10.1.2.2 255.255.255.0
interface e0/0.99
encapsulation dot1Q 99 native
description NATIVE
interface e0/2
ip address 10.1.3.9 255.255.255.252
description TO_R14
no shut
interface e0/3
ip address 10.1.3.13 255.255.255.252
description TO_R15
no shut
interface l0
ip address 10.1.255.2 255.255.255.255
Настройка VRRP
interface e0/0.10
vrrp 10 ip 10.1.1.254
vrrp 10 priority 100
vrrp 10 authentication md5 key-string PASSWORD_MANAGEMENT
interface e0/0.20
vrrp 20 ip 10.1.2.254
vrrp 20 priority 100
vrrp 20 authentication md5 key-string PASSWORD_LAN
do wr
```

Настройка SW4:
```
en
conf t
hostname SW4
no ip domain-lookup
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,99 root primary
interface range e1/2-3
shut
vlan 20
name USERS
vlan 10
name MANAGEMENT
vlan 99
name NATIVE
interface range e0/2-3
channel-group 1 mode active
interface range e1/0-1,e0/0-3,po1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
interface vlan 10
ip address 10.1.1.5 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 10.1.1.254
```

SW5:
```
en
conf t
hostname SW5
no ip domain-lookup
spanning-tree mode rapid-pvst
interface range e1/2-3
shut
vlan 20
name USERS
vlan 10
name MANAGEMENT
vlan 99
name NATIVE
interface range e0/2-3
channel-group 1 mode active
interface range e1/0-1,e0/0-3,po1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
interface vlan 10
ip address 10.1.1.6 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 10.1.1.254
```

SW3:
```
en
conf t
hostname SW3
no ip domain-lookup
spanning-tree mode rapid-pvst
interface e0/3
shut
interface range e1/0-3
shut
vlan 20
name USERS
vlan 10
name MANAGEMENT
vlan 99
name NATIVE
interface e0/2
switchport mode access
switchport access vlan 20
interface range e0/0-1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
interface vlan 10
ip address 10.1.1.4 255.255.255.0
no shut
exit
```
SW2:
```
en
conf t
hostname SW2
no ip domain-lookup
spanning-tree mode rapid-pvst
interface e0/3
shut
interface range e1/0-3
shut
vlan 20
name USERS
vlan 10
name MANAGEMENT
vlan 99
name NATIVE
interface e0/2
switchport mode access
switchport access vlan 20
interface range e0/0-1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
interface vlan 10
ip address 10.1.1.3 255.255.255.0
no shut
exit
```
