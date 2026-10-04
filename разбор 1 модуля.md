Below is the full text content of the document **"Разбор ДЭ 1 модуль.docx"**, converted to text with the original structure preserved.

---

## 1. Настройка доменного имени

**Проверка.**

Команда:
```
hostname
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostname
reauq4vfsudlu
```

**Настройка.**

Команда:
```
hostnamectl set-hostname isp.au-team.irpo
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostnamectl set-hostname isp.au-team.irpo
[root@reauq4vfsudlu ~]# hostname
isp.au-team.irpo
```

---

## 2. Настройка интерфейсов

**Проверка.**

Команда:
```
ip a
```

Вывод:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:cd:3a:00:01:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 10.10.10.2/30 brd 10.10.10.3 scope global dynamic noprefixroute ens3
       valid_lft 267sec preferred_lft 267sec
    inet6 fe80::52cd:3aff:fe00:100/64 scope link
       valid_lft forever preferred_lft forever
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:cd:3a:00:01:01 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 172.16.1.1/28 brd 172.16.1.15 scope global ens4
       valid_lft forever preferred_lft forever
    inet6 fe80::52cd:3aff:fe00:101/64 scope link
       valid_lft forever preferred_lft forever
4: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:cd:3a:00:01:02 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 172.16.2.1/28 brd 172.16.2.15 scope global ens5
       valid_lft forever preferred_lft forever
    inet6 fe80::52cd:3aff:fe00:102/64 scope link
       valid_lft forever preferred_lft forever
5: ens6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:00:00:01:00:fe brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    inet 169.254.2.101/30 brd 169.254.2.103 scope global dynamic noprefixroute ens6
       valid_lft 82609sec preferred_lft 71808sec
    inet6 fe80::5200:ff:fe01:fe/64 scope link dynamic mngtmpaddr
       valid_lft 86299sec preferred_lft 14299sec
    inet6 fe80::5200:ff:fe01:fe/64 scope link
       valid_lft forever preferred_lft forever
[root@isp ~]#
```

**Настройка не требуется.**

---

## 3. Настройка динамической трансляции адресов

**Проверка.**

Команда:
```
vim /etc/net/sysctl.conf
```

Вывод:
```
# This file was formerly part of etc/sysctl.conf
### IPv4 networking options.
#
# IPv4 packet forwarding.
# This variable is special, its change resets all configuration
# parameters to their default state (RFC 1122 for hosts, RFC 1812 for
# routers).
net.ipv4.ip_forward = 1
#
# Source validation by reversed path, as specified in RFC 1812.
```

**Настройка не требуется.**

Команда:
```
iptables -t nat -L
```

Вывод:
```
[root@isp ~]# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
[root@isp ~]#
```

**Настройка.**

Команда:
```
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables
```

Вывод:
```
[root@isp ~]# iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
[root@isp ~]# iptables-save >> /etc/sysconfig/iptables
[root@isp ~]# systemctl enable --now iptables
Synchronizing state of iptables.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable iptables
Created symlink /etc/systemd/system/basic.target.wants/iptables.service → /lib/systemd/system/iptables.service.
[root@isp ~]# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  anywhere             anywhere
```

---

## 4. Настройка часового пояса

**Проверка.**

Команда:
```
timedatectl
```

Вывод:
```
[root@isp ~]# timedatectl
      Local time: Sun 2026-04-05 14:18:20 MSK
  Universal time: Sun 2026-04-05 11:18:20 UTC
        RTC time: Sun 2026-04-05 11:18:20
       Time zone: Europe/Moscow (MSK, +0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

**Настройка не требуется.**

**ISP НАСТРОЕН**

---

## Машина HQ-RTR

### 1. Настройка доменного имени

**Проверка.**

Команда:

Вывод:

**Настройка.**

Команда:

Вывод:

---

### 2. Настройка IP-адресов

**Проверка.**

Команда:
```
sh ip interface brief
```

Вывод:
```
HQ-RTR#sh ip interface brief
Interface        IP-Address        Status
ssh              169.254.2.101/30  down
isp              172.16.1.2/28     up
vl100            10.10.100.1/27    up
vl200            10.10.200.1/28    up
vl999            10.10.30.1/29     up
HQ-RTR#
```

**Настройка не требуется.**

---

### 3. Настройка пользователей

**Проверка.**

Команда:
```
sh users localdb
```

Вывод:
```
HQ-RTR# show users localdb
Roles:
User: net admin
Description:
Docker socket access: disabled
VR:
```

**Настройка не требуется.**

---

### 4. Настройка GRE

**Проверка.**

Команда:
```
sh interface tunnel.0
```

Вывод:
```
HQ-RTR#sh interface tunnel.0
HQ-RTR# если пусто, то требуется настройка
```

**Настройка.**

Команда:
```
en
conf t
interface tunnel.0
ip address 10.10.10.1/30
ip tunnel 172.16.1.2 172.16.2.2 mode gre
end
write memory
```

Вывод:
```
HQ-RTR>en
HQ-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
HQ-RTR(config)#interface tunnel.0
HQ-RTR(config-if-tunnel)#ip address 10.10.10.1/30
HQ-RTR(config-if-tunnel)#ip tunnel 172.16.1.2 172.16.2.2 mode gre
2026-04-02 10:24:35 INFO
HQ-RTR(config-if-tunnel)#end
HQ-RTR#write memory
Building configuration...
HQ-RTR#hs
HQ-RTR#show interface tunnel.0
Interface tunnel.0 is up
Snmp index: 10
Ethernet address: (port not configured)
MTU: 1476
Tunnel source: 172.16.1.2
Tunnel destination: 172.16.2.2
Tunnel mode: GRE
NAT: no
ARP Proxy: disable
ICMP redirects on, unreachables on
IP URPF is disabled
Label switching is disabled
<UP,BROADCAST,RUNNING,NOARP,MULTICAST>
inet 10.10.10.1/30 broadcast 10.10.10.3/30
total input packets 0, bytes 0
total output packets 0, bytes 0
HQ-RTR#
```

---

### 5. Настройка статической маршрутизации

**Проверка.**

Команда:
```
sh ip route
```

Вывод:
```
>en
#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.10.30.0/29 is directly connected, vl999
C       10.10.100.0/27 is directly connected, vl100
C       10.10.200.0/28 is directly connected, vl200
C       172.16.1.0/28 is directly connected, isp

Gateway of last resort is not set
```
**нет значения со *, значит маршрут по умолчанию не настроен**
**требуется настройка**

**Настройка.**

Команда:
```
conf t
ip route 0.0.0.0 0.0.0.0 172.16.1.1
end
write memory
```

Вывод:
```
HQ-RTR(config)#ip route 0.0.0.0 0.0.0.0 172.16.1.1
HQ-RTR(config)#end
HQ-RTR#write
Building configuration...
HQ-RTR#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
Gateway of last resort is 172.16.1.1 to network 0.0.0.0

S*      0.0.0.0/0 [1/0] via 172.16.1.1, isp
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.10.30.0/29 is directly connected, vl999
C       10.10.100.0/27 is directly connected, vl100
C       10.10.200.0/28 is directly connected, vl200
C       172.16.1.0/28 is directly connected, isp
HQ-RTR#
```

---

### 6. Настройка динамической маршрутизации (OSPF)

**Проверка.**

Команда:
```
sh ip route
```

Вывод:
```
>en
#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.10.30.0/29 is directly connected, vl999
C       10.10.100.0/27 is directly connected, vl100
C       10.10.200.0/28 is directly connected, vl200
C       172.16.1.0/28 is directly connected, isp

Gateway of last resort is not set
```
**если нет буквы O в таблице маршрутизации**
**необходима настройка OSPF**

**Настройка.**

Команда:
```
en
conf t
router ospf 0
network 10.10.100.0/27 area 0
network 10.10.200.0/28 area 0
network 10.10.30.0/29 area 0
network 10.10.10.0/30 area 0
passive-interface default
no passive-interface tunnel.0
area 0 authentication
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
end
write memory
```

---

### 7. Настройка динамической трансляции адресов (NAT)

Вывод:
```
HQ-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
HQ-RTR(config)#router ospf 0
HQ-RTR(config-router)#ne
neighbor network
HQ-RTR(config-router)#network 10.10.100.0/27 area 0
HQ-RTR(config-router)#network 10.10.200.0/28 area 0
HQ-RTR(config-router)#network 10.10.30.0/29 area 0
HQ-RTR(config-router)#network 10.10.10.0/30 area 0
HQ-RTR(config-router)#passive-interface default
HQ-RTR(config-router)#no passive-interface tu
HQ-RTR(config-router)#no passive-interface tunnel.0
HQ-RTR(config-router)#area au
HQ-RTR(config-router)#area aut
HQ-RTR(config-router)#area aut
HQ-RTR(config-router)#area aut
HQ-RTR(config-router)#area 0 authentication
HQ-RTR(config-router)#exi
HQ-RTR(config)#interface tunnel.0
HQ-RTR(config-if-tunnel)#ip ospf authentication message-digest
HQ-RTR(config-if-tunnel)#ip ospf message-digest-key 1 md5 P@ssw0rd
```

**Проверка.**

Команда:
```
С HQ-CLI выполнить: ping 172.16.1.1
С HQ-RTR выполнить: show ip nat translations
```

Вывод:
```
HQ-RTR#sh ip nat translations
Static translations:

Source                     Translated                 VRF

Destination                Translated                 VRF

Empty list.
Total: 0

PAT translations:

Source                     Translated                 Destination

Empty list.
Total: 0
```
**Если пусто, необходимо настроить**

Команда:
```
en
conf t
interface isp
ip nat outside
exit
interface vl100
ip nat inside
exit
interface vl200
ip nat inside
exit
interface vl999
ip nat inside
exit
ip nat pool HQ 10.10.0.1-10.10.200.254
ip nat source dynamic inside-to-outside pool HQ overload interface isp
end
write memory
```

Вывод:
```
HQ-RTR#en
HQ-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
HQ-RTR(config)#interface isp
HQ-RTR(config-if)#ip nat outside
HQ-RTR(config-if)#exit
HQ-RTR(config)#interface vl
HQ-RTR(config)#interface vl100 vl200 vl999
HQ-RTR(config)#interface vl100
HQ-RTR(config-if)#ip nat ind
HQ-RTR(config-if)#ip nat inside
HQ-RTR(config-if)#exit
HQ-RTR(config)#interface vl200
HQ-RTR(config-if)#ip nat inside
HQ-RTR(config-if)#exit
HQ-RTR(config)#interface vl999
HQ-RTR(config-if)#ip nat inside
HQ-RTR(config-if)#exit
HQ-RTR(config)#ip na
name-server nat
HQ-RTR(config)#ip nat pool HQ 10.10.0.1-10.10.200.254
HQ-RTR(config)#ip nat source dynamic inside-to-outside pool HQ overload interface isp
HQ-RTR(config)#
```

---

### 8. Настройка протокола динамической конфигурации хостов (DHCP)

Команда:
```
show dhcp-server 1 detailed
```

Вывод:
```
HQ-RTR#show dhcp-server 1 detailed
DHCP-server 1:
* Global options:
  Lease-time: 86400 sec
  Netmask: 255.255.255.0
* Static entries:
* Framed-ip pool entries:
* Pool entries:
  pool VLAN200 1
    Gateway:          10.10.200.1
    DNS-servers:      10.10.100.2
    Domain-name:      au-team.irpo
    Netmask:          255.255.255.240
```

**Настройка не требуется.**

---

### 9. Настройка часового пояса

**Проверка.**

Команда:
```
show ntp timezone
```

Вывод:
```
HQ-RTR#show ntp timezone
System Time zone:
HQ-RTR#
```

**Настройка.**

Команда:
```
en
conf t
ntp timezone utc+3
```

Вывод:
```
HQ-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
HQ-RTR(config)#ntp timezone utc+3
HQ-RTR(config)#end
HQ-RTR#show ntp timezone
System Time zone: Europe/Moscow
HQ-RTR#
```

**HQ-RTR НАСТРОЕН**

---

## Машина BR-RTR

### 1. Настройка доменного имени

**Проверка.**

Команда:
```
sh hostname
```

Вывод:

**Настройка.**

Команда:

Вывод:

---

### 2. Настройка IP-адресов

**Проверка.**

Команда:
```
sh ip interface brief
```

Вывод:
```
BR-RTR#show ip interface brief
Interface        IP-Address        Status
ssh              169.254.2.101/30  up
                 172.16.2.2/28     up
                 10.20.10.1/30     up
BR-RTR#
```

**Настройка не требуется.**

---

### 3. Настройка пользователей

**Проверка.**

Команда:
```
sh users localdb
```

Вывод:
```
# show users localdb
Roles:
User: net admin
Description:
Docker socket access: disabled
VR:
```

**Настройка не требуется.**

---

### 4. Настройка GRE

**Проверка.**

Команда:

Вывод:

**Настройка.**

Команда:

Вывод:

---

### 5. Настройка статической маршрутизации

**Проверка.**

Команда:

Вывод:
```
>en
#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.10.30.0/29 is directly connected, vl999
C       10.10.100.0/27 is directly connected, vl100
C       10.10.200.0/28 is directly connected, vl200
C       172.16.1.0/28 is directly connected, isp

Gateway of last resort is not set
```
**нет значения со *, значит маршрут по умолчанию не настроен**
**требуется настройка**

**Настройка.**

Команда:
```
conf t
ip route 0.0.0.0 0.0.0.0 172.16.2.1
end
write memory
```

Вывод:
```
BR-RTR>en
BR-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
BR-RTR(config)#ip route 0.0.0.0 0.0.0.0 172.16.2.1
BR-RTR(config)#end
BR-RTR#write memory
Building configuration...
BR-RTR#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
Gateway of last resort is 172.16.2.1 to network 0.0.0.0

S*      0.0.0.0/0 [1/0] via 172.16.2.1, isp
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.20.10.0/30 is directly connected, fw
C       169.254.2.100/30 is directly connected, ssh
C       172.16.2.0/28 is directly connected, isp
BR-RTR#
```

---

### 6. Настройка динамической маршрутизации (OSPF)

**Проверка.**

Команда:

Вывод:
```
>en
#show ip route
Codes: C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

IP Route Table for VRF "default"
C       10.10.10.0/30 is directly connected, tunnel.0
C       10.10.30.0/29 is directly connected, vl999
C       10.10.100.0/27 is directly connected, vl100
C       10.10.200.0/28 is directly connected, vl200
C       172.16.1.0/28 is directly connected, isp

Gateway of last resort is not set
```
**если нет буквы O в таблице маршрутизации**
**необходима настройка OSPF**

**Настройка.**

Команда:
```
en
conf t
router ospf 0
network 10.20.10.0/30 area 0
network 10.10.10.0/30 area 0
passive-interface default
no passive-interface tunnel.0
no passive-interface fw
area 0 authentication
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
exit
interface fw
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
end
write memory
```

---

### 7. Настройка динамической трансляции адресов (NAT)

**Проверка.**

Команда:
```
С BR-RTR выполнить: show ip nat translations
```

Вывод:
```
#sh ip nat translations
Static translations:

Source                     Translated                 VRF

Destination                Translated                 VRF

Empty list.
Total: 0

PAT translations:

Source                     Translated                 Destination

Empty list.
Total: 0
```
**Если пусто, необходимо настроить**

Команда:
```
en
conf t
interface isp
ip nat outside
exit
interface fw
ip nat inside
exit
ip nat pool BR 10.0.0.1-10.254.254.254
ip nat source dynamic inside-to-outside pool BR overload interface isp
end
write memory
```

Вывод:
```
BR-RTR#conf terminal
Enter configuration commands, one per line. End with CNTL/Z.
BR-RTR(config)#interface isp
BR-RTR(config-if)#ip nat outside
BR-RTR(config-if)#exit
BR-RTR(config)#interface fw
BR-RTR(config-if)#ip nat inside
BR-RTR(config-if)#exit
BR-RTR(config)#ip nat pool BR 10.0.0.1-10.254.254.254
BR-RTR(config)#ip na
name-server nat
BR-RTR(config)#ip nat source dynamic inside-to-outside pool BR overload interface isp
BR-RTR(config)#end
BR-RTR#write memory
Building configuration...
```

---

### 8. Настройка часового пояса

**Проверка.**

Команда:
```
show ntp timezone
```

Вывод:
```
HQ-RTR#show ntp timezone
System Time zone:
HQ-RTR#
```

**Настройка.**

Команда:
```
en
conf t
ntp timezone utc+3
```

Вывод:
```
HQ-RTR#conf t
Enter configuration commands, one per line. End with CNTL/Z.
HQ-RTR(config)#ntp timezone utc+3
HQ-RTR(config)#end
HQ-RTR#show ntp timezone
System Time zone: Europe/Moscow
HQ-RTR#
```

**BR-SRV НАСТРОЕН**

---

## Машина HQ-SW

### 1. Настройка доменного имени

**Проверка.**

Команда:
```
hostname
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostname
reauq4vfsudlu
```

Команда:
```
hostnamectl set-hostname hq-sw.au-team.irpo
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostnamectl set-hostname hq-sw.au-team.irpo
[root@reauq4vfsudlu ~]# hostname
hq-sw.au-team.irpo
[root@reauq4vfsudlu ~]#
```

---

### 2. Настройка интерфейсов

**Проверка.**

Команда:
```
ip a
```

Вывод:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master os-system state UP group default qlen 1000
    link/ether 50:62:e4:00:1e:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet6 fe80::5262:e4ff:fe00:1e00/64 scope link
       valid_lft forever preferred_lft forever
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master os-system state UP group default qlen 1000
    link/ether 50:62:e4:00:1e:01 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet6 fe80::5262:e4ff:fe00:1e01/64 scope link
       valid_lft forever preferred_lft forever
4: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master os-system state UP group default qlen 1000
    link/ether 50:62:e4:00:1e:02 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet6 fe80::5262:e4ff:fe00:1e02/64 scope link
       valid_lft forever preferred_lft forever
5: ens6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:00:00:03:00:fe brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    inet 169.254.2.101/30 brd 169.254.2.103 scope global dynamic noprefixroute ens6
       valid_lft 77534sec preferred_lft 66734sec
    inet6 fe80::5260:ff:fe00:fc64 scope site dynamic mngtmpaddr
       valid_lft 86117sec preferred_lft 14117sec
    inet6 fe80::5260:ff:fe00:fc64 scope link
       valid_lft forever preferred_lft forever
6: os-system: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ce:fb:0f:34:fb:14 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet6 fe80::0:0:0:0:0:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 10.10.30.2/29 scope global nhost
       valid_lft forever preferred_lft forever
    inet6 fe80::1c04:0bff:fed4:80:45c/64 scope link
       valid_lft forever preferred_lft forever
```

**Настройка не требуется.**

---

### 3. Настройка коммутации в сегменте HQ (VLAN)

**Проверка.**

Команда:
```
ovs-vsctl show
```

Вывод:
```
[root@hq-sw ~]# ovs-vsctl show
a4a93a7a-e450-4ce7-ad85-67474373690c
    Bridge hq-sw
        Port hq-sw
            Interface hq-sw
                type: internal
        Port ens5
            tag: 200
            Interface ens5
        Port ens3
            trunks: [100, 200, 999]
            Interface ens3
        Port MGMT
            tag: 999
            Interface MGMT
                type: internal
        Port ens4
            tag: 100
            Interface ens4
    ovs_version: "2.17.11"
[root@hq-sw ~]#
```

**Настройка не требуется.**

**HQ-SW НАСТРОЕН**

---

## Машина HQ-SRV

### 1. Настройка доменного имени

**Проверка.**

Команда:
```
hostname
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostname
reauq4vfsudlu
```

**Настройка.**

Команда:
```
hostnamectl set-hostname hq-srv.au-team.irpo
```

Вывод:
```
[root@reauq4vfsudlu ~]# hostnamectl set-hostname hq-srv.au-team.irpo
[root@reauq4vfsudlu ~]# hostname
hq-srv.au-team.irpo
[root@reauq4vfsudlu ~]#
```

---

### 2. Настройка интерфейсов

**Проверка.**

Команда:
```
ip a
```

Вывод:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 50:90:76:00:00:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet6 fe80::5290:76ff:fe00:d00/64 scope link
       valid_lft forever preferred_lft forever
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/
