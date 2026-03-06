# VPN IPsec IKEv1 Basada en Políticas

## 📌 Contenido

- Objetivo de la VPN  
- Topología  
- Configuración de Red  
- Parámetros Utilizados  
- Verificación  

---

## 🎯 Objetivo de la VPN

El objetivo de configurar una VPN utilizando **IPsec con IKEv1 basado en políticas (Policy-Based VPN)** es establecer un canal de comunicación seguro entre dos redes a través de una red pública como Internet, garantizando la **confidencialidad, integridad y autenticación** de la información transmitida.

---

## 📊 Parámetros de la VPN IPsec IKEv1 (Basada en Políticas)

| Parámetro                | Valor |
|--------------------------|--------|
| Versión IKE              | IKEv1 |
| Modo Fase 1              | Main Mode |
| Autenticación            | Pre-Shared Key (PSK) |
| Clave PSK                | cisco123 |
| Cifrado Fase 1           | AES-256 |
| Hash Fase 1              | SHA |
| Grupo Diffie-Hellman     | Group 1 |
| Lifetime Fase 1 (R2)     | 3600 segundos |
| Lifetime Fase 1 (R3)     | 86400 segundos |
| Cifrado Fase 2 (IPsec)   | ESP-AES-256 |
| Hash Fase 2              | ESP-SHA256-HMAC |
| Modo IPsec               | Tunnel Mode |
| Selección de tráfico     | Crypto ACL |
| Red protegida R2         | 192.168.10.0/24 ↔ 192.168.20.0/24 |
| Red protegida R3         | 192.168.20.0/24 ↔ 192.168.10.0/24 |

---

## 🖧 Topología

<img width="926" height="614" alt="Topologia VPN" src="https://github.com/user-attachments/assets/8b5283a7-cea7-492b-b699-1708bd6b17ef" />

---

## ⚙️ Configuración de Red

### 🔹 R2 (Peer)

```bash
conf t
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 lifetime 3600
 group 1
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
 match address VPN
 set transform-set VPN
exit

interface f0/0
 crypto map VPN
exit

do wr

