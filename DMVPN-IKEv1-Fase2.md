# DMVPN con IPsec (IKEv1 - Phase 2)

## Objetivo de la configuración

El objetivo de esta práctica es implementar una **Dynamic Multipoint VPN (DMVPN)** utilizando **IPsec con IKEv1 en fase 2**, permitiendo la comunicación segura entre múltiples sucursales (Spokes) y un router central (Hub).

DMVPN permite crear una arquitectura **hub-and-spoke dinámica**, donde los routers spokes pueden comunicarse entre sí mediante túneles GRE multipunto sin necesidad de configurar túneles punto a punto individuales.

La seguridad del túnel se proporciona mediante **IPsec en modo transport**, mientras que el descubrimiento dinámico de peers se realiza mediante **NHRP (Next Hop Resolution Protocol)**.

Además, se utiliza **EIGRP** como protocolo de enrutamiento dinámico para intercambiar rutas entre las redes remotas.

---

# Topología

![Topologia](https://github.com/user-attachments/assets/bbcb579e-5a32-4b93-920e-4d94caece3f2)

---

# Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R4 (Hub) | G1/0 | 10.0.0.1 | Conexión hacia la red WAN |
| R4 (Hub) | Tunnel0 | 172.16.0.1 /24 | Red DMVPN |
| R4 (Hub) | LAN | 192.168.100.0 /24 | Red interna |

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R2 (Spoke1) | G1/0 | 10.0.0.2 | Conexión hacia WAN |
| R2 (Spoke1) | Tunnel0 | 172.16.0.2 /24 | Red DMVPN |
| R2 (Spoke1) | LAN | 192.168.10.0 /24 | Red interna |

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R3 (Spoke2) | G1/0 | 10.0.0.3 | Conexión hacia WAN |
| R3 (Spoke2) | Tunnel0 | 172.16.0.3 /24 | Red DMVPN |
| R3 (Spoke2) | LAN | 192.168.20.0 /24 | Red interna |

---

# Parámetros utilizados

| Parámetro | Valor |
|---|---|
| Tecnología VPN | DMVPN |
| Versión IKE | IKEv1 |
| Fase DMVPN | Phase 2 |
| Protocolo de túnel | GRE Multipoint |
| Seguridad | IPsec |
| Transform-set | esp-aes 192 |
| Modo IPsec | Transport |
| Autenticación | Pre-shared Key |
| Clave compartida | pepe |
| Algoritmo de cifrado | AES 192 |
| Hash | SHA |
| Grupo Diffie-Hellman | 5 |
| Protocolo de enrutamiento | EIGRP |
| Red del túnel | 172.16.0.0 /24 |
| NHRP Network-ID | 1 |
| NHRP Authentication | CISCO |
| Tunnel Key | 1 |
| MTU del túnel | 1400 |
| MSS ajustado | 1360 |

---

# Configuración

## R4 - HUB

```bash
conf t
interface Tunnel0
 ip address 172.16.0.1 255.255.255.0
 tunnel source g1/0
 tunnel mode gre multipoint
 tunnel key 1

 ip nhrp network-id 1
 ip nhrp authentication CISCO
 ip nhrp holdtime 60
 ip nhrp map multicast dynamic
exit

router eigrp 1
 no auto-summary
 network 192.168.100.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface Tunnel0
 no ip split-horizon eigrp 1
 no ip next-hop-self eigrp 1
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto isakmp policy 1
 authentication pre-share
 encryption aes 192
 hash sha
 group 5
exit

crypto isakmp key pepe address 0.0.0.0 0.0.0.0

crypto ipsec transform-set DMVPN_transform esp-aes 192
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
exit

interface Tunnel0
 tunnel protection ipsec profile DMVPN_profile
exit
```

### R2
```bash
conf t
interface Tunnel0
 ip address 172.16.0.2 255.255.255.0
 tunnel source g1/0
 tunnel mode gre multipoint
 tunnel key 1

 ip nhrp network-id 1
 ip nhrp authentication CISCO
 ip nhrp holdtime 60
 ip nhrp map 172.16.0.1 10.0.0.1
 ip nhrp nhs 172.16.0.1
 ip nhrp map multicast 10.0.0.1
exit

router eigrp 1
 no auto-summary
 network 192.168.10.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface Tunnel0
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto isakmp policy 1
 authentication pre-share
 encryption aes 192
 hash sha
 group 5
exit

crypto isakmp key pepe address 0.0.0.0 0.0.0.0

crypto ipsec transform-set DMVPN_transform esp-aes 192
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
exit

interface Tunnel0
 tunnel protection ipsec profile DMVPN_profile
exit
```

### R3
```bash
conf t
interface Tunnel0
 ip address 172.16.0.3 255.255.255.0
 tunnel source g1/0
 tunnel mode gre multipoint
 tunnel key 1

 ip nhrp network-id 1
 ip nhrp authentication CISCO
 ip nhrp holdtime 60
 ip nhrp map 172.16.0.1 10.0.0.1
 ip nhrp nhs 172.16.0.1
 ip nhrp map multicast 10.0.0.1
exit

router eigrp 1
 no auto-summary
 network 192.168.20.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface Tunnel0
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto isakmp policy 1
 authentication pre-share
 encryption aes 192
 hash sha
 group 5
exit

crypto isakmp key pepe address 0.0.0.0 0.0.0.0

crypto ipsec transform-set DMVPN_transform esp-aes 192
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
exit

interface Tunnel0
 tunnel protection ipsec profile DMVPN_profile
exit
```

### Verificacion
```bash
show dmvpn
show ip nhrp
show interface tunnel0
show ip eigrp neighbors
show ip route
show crypto isakmp sa
show crypto ipsec sa
```
