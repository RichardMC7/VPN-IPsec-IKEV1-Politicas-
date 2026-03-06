# VPN IPsec IKEv2 con Túnel GRE

## Contenido

- Objetivo de la VPN  
- Topología  
- Parámetros Utilizados  
- Tabla de Direccionamiento  
- Configuración Completa  
- Verificación  

---

#  Objetivo de la VPN

El objetivo de esta práctica es configurar una **VPN utilizando GRE sobre IPsec con IKEv2**, permitiendo conectar dos redes remotas a través de una red pública de forma segura.

Esta implementación permite:

-  Proteger el tráfico mediante **IPsec**
-  Transportar múltiples tipos de tráfico con **GRE**
-  Utilizar **enrutamiento dinámico o estático**
-  Garantizar **confidencialidad, integridad y autenticación**
-  Permitir comunicación segura entre redes remotas

En este escenario, **GRE crea el túnel lógico** y **IPsec con IKEv2 protege el tráfico GRE**.

---

#  Topología

<img width="884" height="573" alt="Topologia VPN GRE IKEv2" src="https://github.com/user-attachments/assets/e0101bd4-5f9a-41dd-8575-1a38bf7ac6e6" />

---

#  Parámetros Utilizados

##  IKEv2 (Fase 1)

| Parámetro | Valor |
|-----------|------|
| Versión | IKEv2 |
| Cifrado | AES-CBC-256 |
| Integridad | SHA256 |
| Grupo Diffie-Hellman | Group 14 |
| Autenticación | Pre-Shared Key |
| Clave | cisco123 |

---

##  IPsec (Fase 2)

| Parámetro | Valor |
|-----------|------|
| Transform Set | IPSEC-SET |
| Cifrado | ESP-AES-256 |
| Integridad | ESP-SHA256-HMAC |
| Modo | Transport Mode |
| Protección | IPsec Profile |

---

##  Parámetros del Túnel GRE

| Parámetro | Valor |
|-----------|------|
| Tipo de túnel | GRE |
| Red del túnel | 172.16.2.0/30 |
| IP Tunnel R2 | 172.16.2.1 |
| IP Tunnel R3 | 172.16.2.2 |

---

#  Tabla de Direccionamiento

## 🔹 R2

| Interfaz | Dirección IP | Descripción |
|---------|--------------|-------------|
| G0/0 | 10.0.0.2 | Enlace hacia R3 |
| Tunnel0 | 172.16.2.1 /30 | Túnel GRE |
| LAN | 192.168.10.0/24 | Red interna |

---

## 🔹 R3

| Interfaz | Dirección IP | Descripción |
|---------|--------------|-------------|
| G0/0 | 10.0.0.6 | Enlace hacia R2 |
| Tunnel0 | 172.16.2.2 /30 | Túnel GRE |
| LAN | 192.168.20.0/24 | Red interna |

---

# ⚙ Configuración Completa

---

# 🔹 R2

```bash
conf t

interface Tunnel0
 ip address 172.16.2.1 255.255.255.252
 tunnel source 10.0.0.2
 tunnel destination 10.0.0.6
 tunnel mode gre ip
exit

ip route 192.168.20.0 255.255.255.0 172.16.2.2

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
 mode transport
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
exit

interface Tunnel0
 tunnel protection ipsec profile IPSEC-PROFILE
exit
```
### R3
```bash
conf t

interface Tunnel0
 ip address 172.16.2.2 255.255.255.252
 tunnel source 10.0.0.6
 tunnel destination 10.0.0.2
 tunnel mode gre ip
exit

ip route 192.168.10.0 255.255.255.0 172.16.2.1

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
 mode transport
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
exit

interface Tunnel0
 tunnel protection ipsec profile IPSEC-PROFILE
exit

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


do wr
