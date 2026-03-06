# VPN IPsec IKEv1 Basada en Políticas (Policy-Based VPN)

##  Contenido

- Objetivo de la VPN  
- Topología  
- Parámetros Utilizados  
- Configuración de Red  
- Tabla de Enrutamiento  
- Verificación  

---

##  Objetivo de la VPN

El objetivo de esta configuración es establecer una **VPN IPsec utilizando IKEv1 basada en políticas (Policy-Based VPN)** para permitir la comunicación segura entre dos redes a través de una red pública (Internet).

La VPN garantiza:

-  **Confidencialidad**
-  **Autenticación**
-  **Integridad**
-  **Protección del tráfico entre redes internas**

---

##  Topología

<img width="926" height="614" alt="Topologia VPN" src="https://github.com/user-attachments/assets/8b5283a7-cea7-492b-b699-1708bd6b17ef" />

---

##  Parámetros Utilizados (IPsec IKEv1)

| Parámetro | Valor |
|------------|------------|
| Versión IKE | IKEv1 |
| Fase 1 | Main Mode |
| Autenticación | Pre-Shared Key (PSK) |
| Clave PSK | cisco123 |
| Cifrado Fase 1 | AES-256 |
| Hash Fase 1 | SHA |
| Grupo Diffie-Hellman | Group 1 |
| Lifetime Fase 1 (R2) | 3600 segundos |
| Lifetime Fase 1 (R3) | 86400 segundos |
| Cifrado Fase 2 | ESP-AES-256 |
| Hash Fase 2 | ESP-SHA256-HMAC |
| Modo IPsec | Tunnel Mode |
| Selección de tráfico | Crypto ACL |

### Redes Protegidas

- **R2 ↔ R3**
  - 192.168.10.0/24
  - 192.168.20.0/24

---

##  Configuración de Red

### 🔹 R2 (Peer)

```bash
conf t

crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 1
 lifetime 3600
exit

crypto isakmp key cisco123 address 10.0.0.6

ip access-list extended VPN
 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
exit

crypto ipsec transform-set VPN esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto map VPN 10 ipsec-isakmp
 set peer 10.0.0.6
 set transform-set VPN
 match address VPN
exit

interface f0/0
 crypto map VPN
exit

do wr

```
### R3 (Peer)
```bash
conf t

crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 1
 lifetime 86400
exit

crypto isakmp key cisco123 address 10.0.0.2

ip access-list extended VPN
 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
exit

crypto ipsec transform-set VPN esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto map VPN 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set VPN
 match address VPN
exit

interface f0/0
 crypto map VPN
exit

do wr
```

### Verificacion 
```bash
show crypto isakmp sa
show crypto ipsec sa
show crypto map
ping 192.168.20.1
```

