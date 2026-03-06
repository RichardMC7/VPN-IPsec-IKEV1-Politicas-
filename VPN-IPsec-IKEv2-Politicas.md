# VPN IPsec IKEv2 con Políticas (Policy-Based)

##  Contenido

- Objetivo de la VPN  
- Topología  
- Parámetros Utilizados  
- Tabla de Direccionamiento  
- Configuración Completa  
- Verificación  

---

##  Objetivo de la VPN

Implementar una **VPN IPsec utilizando IKEv2 basada en políticas (Policy-Based VPN)** para establecer un canal seguro entre dos redes remotas.

Esta solución proporciona:

-  Cifrado fuerte con IKEv2  
-  Autenticación mediante Pre-Shared Key  
-  Integridad de los datos  
-  Comunicación segura entre redes internas  
-  Protección del tráfico definido por ACL  

---

##  Topología

<img width="428" height="284" alt="Topologia VPN IKEv2" src="https://github.com/user-attachments/assets/24fb7144-1289-4389-a476-485bf1a76100" />

---

##  Parámetros Utilizados

###  IKEv2 (Fase 1)

| Parámetro | Valor |
|------------|------------|
| Versión | IKEv2 |
| Cifrado | AES-CBC-256 |
| Integridad | SHA-256 |
| Grupo Diffie-Hellman | Group 14 |
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
| Tipo de VPN | Policy-Based |
| Método | Crypto Map |

---

###  Redes Protegidas

- 192.168.10.0/24 ↔ 192.168.20.0/24

---

##  Tabla de Direccionamiento

### 🔹 R2

| Interfaz | Dirección IP | Descripción |
|------------|-------------|--------------|
| G1/0 | 10.0.0.2 | Enlace hacia R3 |
| LAN | 192.168.10.0/24 | Red interna |

---

### 🔹 R3

| Interfaz | Dirección IP | Descripción |
|------------|-------------|--------------|
| G1/0 | 10.0.0.6 | Enlace hacia R2 |
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

ip access-list extended VPN-TRAFFIC
 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
exit

crypto ipsec transform-set IPSEC-SET esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.6
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
 match address VPN-TRAFFIC
exit

interface g1/0
 crypto map VPN-MAP
exit

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

ip access-list extended VPN-TRAFFIC
 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
exit

crypto ipsec transform-set IPSEC-SET esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set IPSEC-SET
 set ikev2-profile IKEV2-PROFILE
 match address VPN-TRAFFIC
exit

interface g1/0
 crypto map VPN-MAP
exit

do wr
```

### Verificaion

```bash
show crypto ikev2 sa
show crypto ipsec sa
show crypto map
show ip route
ping 192.168.20.1
```
