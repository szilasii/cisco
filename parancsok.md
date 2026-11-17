#EXEC privelégizált mód engedélyezése
```cmd
enable
```
#konfigurálás
```cmd
configure terminal
```
#A default switch config ellenőrzése (show parancs exec módban)

```cmd

show running-config //megmutatja az éppen futó konfigurációt

show startup-config // megmutatja a mentett konfigurációt

show interface vlan 1

show ip interface vlan1

show version // megmutatja az IOs verziót

show interface gig1/0/6

show vlan

show flash:
dir flash:
```

#switch alap beállítások

```cmd
no ip domain-lookup
hostname S1
service password-encription
enable secret class
banner motd #  //figyelmeztető üzenet # a lezáráshoz

```
# Cisco CCNA Kibővített Konfigurációs Útmutató (v2)

Ez a dokumentum a Cisco Certified Network Associate (CCNA) vizsgához és a mindennapi hálózati üzemeltetéshez szükséges legfontosabb Switch és Router konfigurációs parancsokat tartalmazza.

---

## 1. Módváltó és Alapozó Parancsok

A Cisco IOS parancssorában (CLI) különböző jogosultsági szintek léteznek. A navigációhoz az alábbi parancsokat használjuk:

* `enable` – Belépés a kiváltságos (privileged) módba (Prompt: `Router#` vagy `Switch#`).
* `configure terminal` – Belépés a globális konfigurációs módba (Prompt: `Switch(config)#`).
* `interface [típus] [szám]` – Belépés egy adott interfész beállításaiba (Prompt: `Switch(config-if)#`).
* `exit` – Visszalépés az előző konfigurációs szintre.
* `end` – Azonnali visszatérés a kiváltságos módba.

---

## 2. Cisco Switch Konfiguráció

### Alapvető beállítások és portkezelés
```routeros
Switch(config)# hostname SW-MAIN                 ! Eszköz nevének megadása
SW-MAIN(config)# enable secret TitkosJelszo123   ! Privilegizált mód jelszava (titkosított)
SW-MAIN(config)# banner motd # Csak adminoknak! # ! Figyelmeztető üzenet belépéskor
```

### VLAN-ok és Port Módok (Access, Trunk)
```routeros
SW-MAIN(config)# vlan 10                         ! VLAN létrehozása
SW-MAIN(config-vlan)# name Admin_Network         ! VLAN elnevezése
SW-MAIN(config-vlan)# exit

SW-MAIN(config)# interface fastEthernet 0/1      ! Port kiválasztása
SW-MAIN(config-if)# switchport mode access       ! Port beállítása végberendezéshez
SW-MAIN(config-if)# switchport access vlan 10    ! Port hozzárendelése a VLAN 10-hez
SW-MAIN(config-if)# no shutdown                  ! Port aktiválása (Switchen alapból aktív, de megerősíthető)

SW-MAIN(config)# interface gigabitEthernet 0/1   ! Trunk port kiválasztása (pl. router felé)
SW-MAIN(config-if)# switchport mode trunk        ! Trunk mód bekapcsolása
SW-MAIN(config-if)# switchport trunk allowed vlan 10,20,30 ! Csak a kijelölt VLAN-ok átengedése
```
### VLAN forgalomirányítás (router on a stick)
```routeros
S2(config)# vlan 10
S2(config-vlan)# name LAN10
S2(config-vlan)# exit
S2(config)# vlan 20
S2(config-vlan)# name LAN20
S2(config-vlan)# exit
S2(config)# vlan 99
S2(config-vlan)# name Management
S2(config-vlan)# exit
S2(config)#
S2(config)# interface vlan 99
S2(config-if)# ip add 192.168.99.3 255.255.255.0
S2(config-if)# no shut
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.99.1
S2(config)# interface fa0/18
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 20
S2(config-if)# no shut
S2(config-if)# exit
S2(config)# interface fa0/1
S2(config-if)# switchport mode trunk
S2(config-if)# no shut
S2(config-if)# exit
S2(config-if)# end

R1(config)# interface G0/0/1.10
R1(config-subif)# description Default Gateway for VLAN 10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip add 192.168.10.1 255.255.255.0
R1(config-subif)# exit
R1(config)#
R1(config)# interface G0/0/1.20
R1(config-subif)# description Default Gateway for VLAN 20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip add 192.168.20.1 255.255.255.0
R1(config-subif)# exit
R1(config)#
R1(config)# interface G0/0/1.99
R1(config-subif)# description Default Gateway for VLAN 99
R1(config-subif)# encapsulation dot1Q 99
R1(config-subif)# ip add 192.168.99.1 255.255.255.0
R1(config-subif)# exit
R1(config)#
R1(config)# interface G0/0/1
R1(config-if)# description Trunk link to S1
R1(config-if)# no shut
R1(config-if)# end
R1#

```
---

### VLAN forgalomirányítás 2 (router 3. rétgben)
```routeros
D1(config)# vlan 10
D1(config-vlan)# name LAN10
D1(config-vlan)# vlan 20
D1(config-vlan)# name LAN20
D1(config-vlan)# exit
D1(config)#

D1(config)# interface vlan 10
D1(config-if)# description Default Gateway SVI for 192.168.10.0/24
D1(config-if)# ip add 192.168.10.1 255.255.255.0
D1(config-if)# no shut
D1(config-if)# exit
D1(config)#
D1(config)# int vlan 20
D1(config-if)# description Default Gateway SVI for 192.168.20.0/24
D1(config-if)# ip add 192.168.20.1 255.255.255.0
D1(config-if)# no shut
D1(config-if)# exit
D1(config)#

D1(config)# interface GigabitEthernet1/0/6
D1(config-if)# description Access port to PC1
D1(config-if)# switchport mode access
D1(config-if)# switchport access vlan 10
D1(config-if)# exit
D1(config)#
D1(config)# interface GigabitEthernet1/0/18
D1(config-if)# description Access port to PC2
D1(config-if)# switchport mode access
D1(config-if)# switchport access vlan 20
D1(config-if)# exit

D1(config)# ip routing
D1(config)#


D1(config)# router ospf 10
D1(config-router)# network 192.168.10.0 0.0.0.255 area 0
D1(config-router)# network 192.168.20.0 0.0.0.255 area 0
D1(config-router)# network 10.10.10.0 0.0.0.3 area 0
D1(config-router)# ^Z
D1#
```
---

### Port Security (Biztonsági korlátozások a portokon)
Megakadályozza, hogy illetéktelen eszközök csatlakozzanak a switch portjára.
```routeros
SW-MAIN(config-if)# switchport port-security                  ! Port security bekapcsolása
SW-MAIN(config-if)# switchport port-security maximum 2        ! Max 2 MAC cím engedélyezése a porton
SW-MAIN(config-if)# switchport port-security mac-address sticky ! Az aktuálisan látott MAC címek megjegyzése
SW-MAIN(config-if)# switchport port-security violation shutdown ! Szabálysértés esetén a port automatikus lekapcsolása
```

### Switch Menedzsment IP cím (SVI)
```routeros
SW-MAIN(config)# interface vlan 1                ! Belépés a menedzsment VLAN-ba
SW-MAIN(config-if)# ip address 192.168.1.5 255.255.255.0 ! IP cím és maszk megadása
SW-MAIN(config-if)# no shutdown                  ! Az interfész bekapcsolása (KÖTELEZŐ)
SW-MAIN(config-if)# exit
SW-MAIN(config)# ip default-gateway 192.168.1.1  ! Alapértelmezett átjáró (a távoli eléréshez)
```

---

## 3. Cisco Router Konfiguráció

### Normál interfész címzés és az Interfész Állapotkezelés (`no shutdown`)
A routerek portjai gyárilag le vannak tiltva (Adminisztratívan lezárt állapot). Használat előtt kötelező aktiválni őket!

* `no shutdown` – Bekapcsolja és aktiválja az interfészt (Állapot: **UP**).
* `shutdown` – Kikapcsolja és adminisztratívan lezárja az interfészt (Állapot: **DOWN**).

```routeros
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# description Kapcsolat a LAN felé
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown                  ! Aktiválás! (Konzolüzenet: changed state to up)
```

### Inter-VLAN Routing (Router-on-a-Stick)
Egyetlen fizikai interfész felosztása virtuális alinterfészekre a VLAN-ok közötti útvonalválasztáshoz.
```routeros
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# no shutdown                  ! A fizikai főportot BE kell kapcsolni IP cím nélkül!
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1.10 ! Alinterfész létrehozása a VLAN 10-nek
Router(config-subif)# encapsulation dot1Q 10     ! 802.1Q enkapszuláció hozzárendelése a VLAN 10-hez
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

### DHCP Szerver beállítása a Routeren
```routeros
Router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10 ! Kiosztásból kizárt címek (pl. szerverek, router)
Router(config)# ip dhcp pool LAN_POOL           ! DHCP pool létrehozása és elnevezése
Router(config-dhcp)# network 192.168.10.0 255.255.255.0 ! Kiosztható alhálózat
Router(config-dhcp)# default-router 192.168.10.1 ! Alapértelmezett átjáró a klienseknek
Router(config-dhcp)# dns-server 8.8.8.8         ! DNS szerver megadása
```

### Dinamikus Útvonalválasztás: OSPFv2
```routeros
Router(config)# router ospf 1                   ! OSPF folyamat indítása (Process ID: 1)
Router(config-router)# router-id 1.1.1.1        ! A router egyedi OSPF azonosítója
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0 ! Hálózat hirdetése a 0-s gerincterületen (Wildcard maszkkal!)
```

---

## 4. SSH Távoli Elérés (Biztonságos Menedzsment)

```routeros
Router(config)# ip domain-name cisco.local       ! Domain név megadása (kötelező a kulcshoz)
Router(config)# crypto key generate rsa         ! RSA titkosító kulcs generálása
! (A kérdésre adj meg legalább 1024 vagy 2048 bites kulcsméretet)

Router(config)# username admin secret AdminPass321 ! Helyi adminisztrátori felhasználó létrehozása
Router(config)# line vty 0 4                    ! Virtuális terminál vonalak (0-4) kijelölése
Router(config-line)# transport input ssh        ! Csak az SSH engedélyezése (a titkosítatlan Telnet tiltva!)
Router(config-line)# login local                ! A bejelentkezéshez a helyi felhasználói adatbázist használja
Router(config-line)# exit
```

---

## 5. ACL (Access Control List - Hozzáférési Listák)

Az ACL-ek segítségével szűrhetjük a hálózati forgalmat biztonsági okokból.

### Standard ACL (Csak forrás IP cím alapján szűr)
```routeros
Router(config)# access-list 10 permit 192.168.10.0 0.0.0.255 ! A teljes 10-es alhálózat átengedése
Router(config)# access-list 10 deny any                      ! Minden más forgalom tiltása (implicit deny)

! Alkalmazás egy interfészre (bejövő vagy kimenő irányba):
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip access-group 10 out
```

### Extended ACL (Kiterjesztett - Forrás, Cél, Port és Protokoll alapján szűr)
Szabály: Engedjük meg a `192.168.10.5` IP-jű gépnek, hogy elérje a `10.0.0.2` szervert webes (HTTP/80) porton, de mindenki mást tiltsunk le felé.
```routeros
Router(config)# access-list 101 permit tcp host 192.168.10.5 host 10.0.0.2 eq 80
Router(config)# access-list 101 deny ip any host 10.0.0.2

! Alkalmazás interfészre:
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip access-group 101 in
```

---

## 6. NAT (Network Address Translation - Címfordítás)

### Statikus NAT (Egy belső magán IP fix hozzárendelése egy külső publikus IP-hez)
```routeros
Router(config)# ip nat inside source static 192.168.10.5 203.0.113.5
```

### Dinamikus NAT Overload / PAT (Sok belső kliens internetre engedése a külső interfész IP-jével)
```routeros
! 1. Lépés: ACL létrehozása a belső hálózat kijelölésére, amit fordítani akarunk
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255

! 2. Lépés: A NAT szabály összekapcsolása az ACL-lel és a külső interfésszel (Overload = PAT)
Router(config)# ip nat inside source list 1 interface gigabitEthernet 0/2 overload

! 3. Lépés: Az interfészek irányának kijelölése (KÖTELEZŐ LÉPÉS!)
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip nat inside                 ! Belső (LAN) oldal
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/2
Router(config-if)# ip nat outside                ! Külső (WAN/Internet) oldal
```

---

## 7. Mentés és Diagnosztika (Show Parancsok)

A konfigurációt mindig el kell menteni, különben áramszünet vagy újraindítás esetén elveszik.
* `write memory` vagy `copy running-config startup-config` – Konfiguráció mentése az NVRAM-ba.

### Legfontosabb ellenőrző parancsok (Kiváltságos módban):
* `show running-config` – A memóriában futó aktuális beállítások megtekintése.
* `show ip interface brief` – Az interfészek IP címei, és hogy aktívak-e (Status: up/down, Protocol: up/down).
* `show vlan brief` – A switchen létező VLAN-ok és porttársítások ellenőrzése.
* `show mac address-table` – A switch MAC cím táblája (melyik porton milyen eszköz van).
* `show ip route` – A router útvonalválasztási (routing) táblája.
* `show ip nat translations` – Az aktív NAT címfordítások listázása.
* `show access-lists` – A létrehozott ACL listák és a szabályokra illeszkedő csomagok száma.

