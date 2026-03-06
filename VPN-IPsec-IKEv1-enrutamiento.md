

# VPN IPsec IKEv1 con Enrutamiento (Route-Based VPN)

##  Contenido

- Objetivo de la VPN  
- Topología  
- Configuración de R2  
- Configuración de R3  
- Verificación  

---

##  Objetivo de la VPN

El objetivo de esta implementación es configurar una **VPN IPsec IKEv1 con enrutamiento (Route-Based VPN)** utilizando una interfaz **Tunnel**.

A diferencia de la VPN basada en políticas, esta solución:

- Utiliza una interfaz lógica (Tunnel0)
- Permite mayor escalabilidad
- Facilita el enrutamiento dinámico
- Encapsula el tráfico dentro del túnel IPsec
- Garantiza **confidencialidad, integridad y autenticación**

---

## 📡 Tabla de Direccionamiento

### 🔹 R2

| Interfaz | Dirección IP | Máscara | Descripción |
|----------|-------------|----------|-------------|
| G0/0 | 10.0.0.2 | 255.255.255.252 | Enlace hacia ISP |
| G0/1 | 192.168.10.1 | 255.255.255.0 | Red LAN |
| Tunnel0 | 172.16.1.1 | 255.255.255.252 | Túnel IPsec hacia R3 |

---

### 🔹 R3

| Interfaz | Dirección IP | Máscara | Descripción |
|----------|-------------|----------|-------------|
| G0/0 | 10.0.0.6 | 255.255.255.252 | Enlace hacia ISP |
| G0/1 | 192.168.20.1 | 255.255.255.0 | Red LAN |
| Tunnel0 | 172.16.1.2 | 255.255.255.252 | Túnel IPsec hacia R2 |

---

### 🔹 Red del Túnel

| Red | Máscara | Uso |
|------|----------|------|
| 172.16.1.0 | 255.255.255.252 | Enlace lógico del túnel IPsec |

---

### 🔹 Redes LAN Protegidas

| Router | Red Local |
|----------|-------------|
| R2 | 192.168.10.0/24 |
| R3 | 192.168.20.0/24 |

##  Topología

<img width="915" height="575" alt="Topologia VPN Route-Based" src="https://github.com/user-attachments/assets/f421fffa-da12-4796-98b9-dc0eaf61b7c4" />

---

## ⚙ Configuración de R2

```bash
enable
conf t

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key cisco123 address 10.0.0.6

crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set TS
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
enable
conf t

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key cisco123 address 10.0.0.2

crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROFILE
 set transform-set TS
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
show crypto isakmp sa
show crypto ipsec sa
show interface tunnel0
show ip route
ping 192.168.20.1
```
