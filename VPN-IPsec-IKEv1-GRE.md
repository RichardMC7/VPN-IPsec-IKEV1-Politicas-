
# VPN IPsec IKEv1 con GRE (Configuración Completa)

##  Contenido

- Objetivo de la VPN  
- Topología  
- Parámetros Utilizados  
- Tabla de Direccionamiento  
- Configuración Completa  
- Verificación  

---

##  Objetivo de la VPN

Implementar una solución de **GRE sobre IPsec con IKEv1**, permitiendo:

-  Cifrado del tráfico mediante IPsec  
-  Enrutamiento dinámico mediante GRE  
-  Soporte de OSPF sobre el túnel  
-  Autenticación e integridad de datos  
-  Comunicación segura entre redes remotas  

---

##  Topología

<img width="896" height="571" alt="Topologia VPN GRE IPsec" src="https://github.com/user-attachments/assets/de0562fa-174e-4714-993c-861ce989afb0" />

---

## ⚙ Parámetros Utilizados

###  IKEv1 (Fase 1)

| Parámetro | Valor |
|------------|------------|
| Cifrado | AES |
| Hash | SHA |
| Autenticación | Pre-Shared Key |
| Clave | cisco123 |
| Grupo DH | Group 2 |
| Lifetime | 86400 |

###  IPsec (Fase 2)

| Parámetro | Valor |
|------------|------------|
| Transform Set | GRE-TS |
| Cifrado | ESP-AES |
| Integridad | ESP-SHA-HMAC |
| Modo | Transport Mode |
| Método de protección | Crypto Map |
| ACL utilizada | 120 |

###  Túnel GRE

| Parámetro | Valor |
|------------|------------|
| Red del túnel | 172.16.2.0/30 |
| Protocolo | GRE |
| Tipo | tunnel mode gre ip |

---

### Tabla de Direccionamiento

### 🔹 R2

| Interfaz | Dirección IP | Máscara |
|------------|-------------|------------|
| FastEthernet0/0 | 10.0.0.2 | 255.255.255.252 |
| Tunnel0 | 172.16.2.1 | 255.255.255.252 |
| LAN | 192.168.10.0/24 | — |

---

### 🔹 R3

| Interfaz | Dirección IP | Máscara |
|------------|-------------|------------|
| FastEthernet0/1 | 10.0.0.6 | 255.255.255.252 |
| Tunnel0 | 172.16.2.2 | 255.255.255.252 |
| LAN | 192.168.20.0/24 | — |

---

#  CONFIGURACIÓN DE RED

---

##  R2

```bash
interface Tunnel0
 ip address 172.16.2.1 255.255.255.252
 tunnel source 10.0.0.2
 tunnel destination 10.0.0.6
 tunnel mode gre ip
exit

router ospf 1
 network 172.16.2.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
exit

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key cisco123 address 10.0.0.6

access-list 120 permit gre host 10.0.0.2 host 10.0.0.6

crypto ipsec transform-set GRE-TS esp-aes esp-sha-hmac
 mode transport
exit

crypto map GRE-MAP 10 ipsec-isakmp
 set peer 10.0.0.6
 set transform-set GRE-TS
 match address 120
exit

interface FastEthernet0/0
 crypto map GRE-MAP
exit

do wr
```

### R3 

```bash
interface Tunnel0
 ip address 172.16.2.2 255.255.255.252
 tunnel source 10.0.0.6
 tunnel destination 10.0.0.2
 tunnel mode gre ip
exit

router ospf 1
 network 172.16.2.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
exit

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key cisco123 address 10.0.0.2

access-list 120 permit gre host 10.0.0.6 host 10.0.0.2

crypto ipsec transform-set GRE-TS esp-aes esp-sha-hmac
 mode transport
exit

crypto map GRE-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set GRE-TS
 match address 120
exit

interface FastEthernet0/1
 crypto map GRE-MAP
exit

do wr
```

### Verificacion 
```bash
show crypto isakmp sa
show crypto ipsec sa
show interface tunnel0
show ip ospf neighbor
show ip route
```
