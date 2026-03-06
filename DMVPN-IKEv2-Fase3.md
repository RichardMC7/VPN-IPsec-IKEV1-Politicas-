# DMVPN con IPsec (IKEv2 - Phase 3)

## Objetivo de la configuración

El objetivo de esta práctica es implementar **DMVPN (Dynamic Multipoint VPN) utilizando IPsec con IKEv2 en Phase 3**, permitiendo la comunicación segura entre múltiples sucursales (Spokes) y un router central (Hub).

DMVPN Phase 3 mejora el funcionamiento de las fases anteriores permitiendo que los **routers Spokes establezcan túneles directos entre ellos**, evitando que el tráfico tenga que pasar siempre por el Hub.

Esto se logra mediante:

- **NHRP Redirect** en el Hub
- **NHRP Shortcut** en los Spokes

La seguridad del túnel se implementa mediante **IPsec en modo transport**, mientras que **EIGRP** se utiliza como protocolo de enrutamiento dinámico para intercambiar rutas entre las redes remotas.

---

# Topología

![Topologia](https://github.com/user-attachments/assets/d9bddd6d-40a7-4448-9e2d-7593047e47d4)

---

# Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R4 (Hub) | G1/0 | 10.0.0.1 | Conexión hacia la red WAN |
| R4 (Hub) | Tunnel0 | 172.16.0.1 /24 | Red DMVPN |
| R4 (Hub) | LAN | 192.168.100.0 /24 | Red interna |

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R2 (Spoke1) | G1/0 | 10.0.0.2 | Conexión WAN |
| R2 (Spoke1) | Tunnel0 | 172.16.0.2 /24 | Red DMVPN |
| R2 (Spoke1) | LAN | 192.168.10.0 /24 | Red interna |

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| R3 (Spoke2) | G1/0 | 10.0.0.3 | Conexión WAN |
| R3 (Spoke2) | Tunnel0 | 172.16.0.3 /24 | Red DMVPN |
| R3 (Spoke2) | LAN | 192.168.20.0 /24 | Red interna |

---

# Parámetros utilizados

| Parámetro | Valor |
|---|---|
| Tecnología VPN | DMVPN |
| Fase DMVPN | Phase 3 |
| Versión IKE | IKEv2 |
| Protocolo de túnel | GRE Multipoint |
| Seguridad | IPsec |
| Transform-set | esp-aes 192 esp-sha-hmac |
| Modo IPsec | Transport |
| Autenticación | Pre-shared Key |
| Clave compartida | pepe |
| Cifrado IKE | AES-CBC 192 |
| Integridad | SHA1 |
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

interface tunnel0
 ip address 172.16.0.1 255.255.255.0
 tunnel source g1/0
 tunnel mode gre multipoint
 tunnel key 1

 ip nhrp network-id 1
 ip nhrp authentication CISCO
 ip nhrp holdtime 60
 ip nhrp map multicast dynamic
 ip nhrp redirect
exit

router eigrp 1
 no auto-summary
 network 192.168.100.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface tunnel0
 no ip split-horizon eigrp 1
 no ip next-hop-self eigrp 1
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto ikev2 proposal DMVPN-PROP
 encryption aes-cbc-192
 integrity sha1
 group 5
exit

crypto ikev2 policy DMVPN-POLICY
 proposal DMVPN-PROP
exit

crypto ikev2 keyring DMVPN-KEY
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key pepe
 exit
exit

crypto ikev2 profile DMVPN-IKEV2
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEY
exit

crypto ipsec transform-set DMVPN_transform esp-aes 192 esp-sha-hmac
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
 set ikev2-profile DMVPN-IKEV2
exit

interface tunnel0
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
 ip nhrp shortcut
exit

router eigrp 1
 no auto-summary
 network 192.168.10.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface tunnel0
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto ikev2 proposal DMVPN-PROP
 encryption aes-cbc-192
 integrity sha1
 group 5
exit

crypto ikev2 policy DMVPN-POLICY
 proposal DMVPN-PROP
exit

crypto ikev2 keyring DMVPN-KEY
 peer HUB
  address 10.0.0.1
  pre-shared-key pepe
 exit
exit

crypto ikev2 profile DMVPN-IKEV2
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEY
exit

crypto ipsec transform-set DMVPN_transform esp-aes 192 esp-sha-hmac
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
 set ikev2-profile DMVPN-IKEV2
exit

interface tunnel0
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
 ip nhrp shortcut
exit

router eigrp 1
 no auto-summary
 network 192.168.20.0 0.0.0.255
 network 172.16.0.0 0.0.0.255
exit

interface tunnel0
 ip mtu 1400
 ip tcp adjust-mss 1360
exit

crypto ikev2 proposal DMVPN-PROP
 encryption aes-cbc-192
 integrity sha1
 group 5
exit

crypto ikev2 policy DMVPN-POLICY
 proposal DMVPN-PROP
exit

crypto ikev2 keyring DMVPN-KEY
 peer HUB
  address 10.0.0.1
  pre-shared-key pepe
 exit
exit

crypto ikev2 profile DMVPN-IKEV2
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEY
exit

crypto ipsec transform-set DMVPN_transform esp-aes 192 esp-sha-hmac
 mode transport
exit

crypto ipsec profile DMVPN_profile
 set transform-set DMVPN_transform
 set ikev2-profile DMVPN-IKEV2
exit

interface tunnel0
 tunnel protection ipsec profile DMVPN_profile
exit
```

### Verificacion
```bash
show ip nhrp
show dmvpn
show ip eigrp neighbors
show crypto ikev2 sa
```
