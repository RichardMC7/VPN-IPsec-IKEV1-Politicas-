
# VPN IPsec IKEv2 con Enrutamiento (Route-Based)

##  Contenido

- Objetivo de la VPN  
- Topología  
- Parámetros Utilizados  
- Tabla de Direccionamiento  
- Configuración Completa  
- Verificación  

---

##  Objetivo de la VPN

Configurar una **VPN IPsec con IKEv2 utilizando enrutamiento (Route-Based VPN)** mediante una interfaz **Tunnel0**.

Este tipo de implementación permite:

-  Cifrado fuerte con IKEv2  
-  Autenticación mediante Pre-Shared Key  
-  Enrutamiento dinámico o estático a través del túnel  
-  Mayor escalabilidad que la VPN basada en políticas  
-  Uso de una interfaz lógica (Tunnel)

---

##  Topología

<img width="428" height="284" alt="Topologia VPN IKEv2 Route-Based" src="https://github.com/user-attachments/assets/24fb7144-1289-4389-a476-485bf1a76100" />

---

##  Parámetros Utilizados

###  IKEv2 (Fase 1)

| Parámetro | Valor |
|------------|------------|
| Versión | IKEv2 |
| Cifrado | AES-CBC-256 |
| Integridad | SHA-256 |
| Grupo DH | Group 14 |
| Autenticación | Pre-Shared Key |
| Clave | cisco123 |

---

###  IPsec (Fase 2)

| Parámetro | Valor |
|------------|------------|
| Transform Set | IPSEC-SET |
| Cifrado | ESP-AES-256 |
| Integridad | ESP-SHA256-HMAC |
| Modo | Tunnel Mode |
| Tipo de VPN | Route-Based |
| Protección | IPsec Profile |

---

###  Túnel

| Parámetro | Valor |
|------------|------------|
| Red del túnel | 172.16.1.0/30 |
| IP R2 | 172.16.1.1 |
| IP R3 | 172.16.1.2 |
| Tipo | tunnel mode ipsec ipv4 |

---

##  Tabla de Direccionamiento

###  R2

| Interfaz | IP | Descripción |
|------------|------|--------------|
| G0/0 | 10.0.0.2 | Enlace hacia R3 |
| Tunnel0 | 172.16.1.1/30 | Túnel IPsec |
| LAN | 192.168.10.0/24 | Red interna |

---

###  R3

| Interfaz | IP | Descripción |
|------------|------|--------------|
| G0/0 | 10.0.0.6 | Enlace hacia R2 |
| Tunnel0 | 172.16.1.2/30 | Túnel IPsec |
| LAN | 192.168.20.0/24 | Red interna |

---

#  CONFIGURACIÓN COMPLETA

---

##  R2

```bash
conf t

crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
exit

crypto ikev2 policy IKEV2-POL
 proposal IKEV2-PROP
exit

crypto ikev2 keyring IKEV2-KEYRING
 peer R3
  address 10.0.0.6
  pre-shared-key cisco123
exit

crypto ikev2 profile IKEV2-PROFILE
 match identity remote address 10.0.0.6 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local IKEV2-KEYRING
exit

crypto ipsec transform-set IPSEC-SET esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
exit

interface Tunnel0
 ip address 172.16.1.1 255.255.255.252
 tunnel source 10.0.0.2
 tunnel destination 10.0.0.6
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-PROFILE
exit

ip route 192.168.20.0 255.255.255.0 172.16.1.2

do wr
```
### R3
```bash
conf t

crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
exit

crypto ikev2 policy IKEV2-POL
 proposal IKEV2-PROP
exit

crypto ikev2 keyring IKEV2-KEYRING
 peer R2
  address 10.0.0.2
  pre-shared-key cisco123
exit

crypto ikev2 profile IKEV2-PROFILE
 match identity remote address 10.0.0.2 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local IKEV2-KEYRING
exit

crypto ipsec transform-set IPSEC-SET esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
exit

interface Tunnel0
 ip address 172.16.1.2 255.255.255.252
 tunnel source 10.0.0.6
 tunnel destination 10.0.0.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-PROFILE
exit

ip route 192.168.10.0 255.255.255.0 172.16.1.1

do wr

```

### Verificacion
```bash
show crypto ikev2 sa
show crypto ipsec sa
show interface tunnel0
show ip route
ping 192.168.20.1

```



