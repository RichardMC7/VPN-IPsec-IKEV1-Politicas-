# VPN Client-to-Site con IPsec IKEv1 y L2TP

## Objetivo de la configuración

El objetivo de esta práctica es implementar una **VPN Client-to-Site punto a multipunto utilizando IPsec con IKEv1 y L2TP**, permitiendo que clientes remotos se conecten de forma segura a la red corporativa a través de Internet.

En esta arquitectura:

- **L2TP** se encarga de crear el túnel lógico para transportar el tráfico del cliente.
- **IPsec** proporciona **cifrado, integridad y autenticación** para proteger los datos que viajan por el túnel.
- **IKEv1** se utiliza para negociar las asociaciones de seguridad (SA) entre el cliente y el servidor VPN.

De esta forma, múltiples clientes remotos pueden conectarse simultáneamente al servidor VPN y acceder a la red corporativa de forma segura.

---

# Topología

![Topologia](https://github.com/user-attachments/assets/8f8f171f-3ca7-4a83-889e-c7693719b9c2)

---

# Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| ISP | G2/0 | 172.30.0.1 /24 | Conexión hacia cliente Ubuntu |
| ISP | G1/0 | 172.30.0.5 /24 | Conexión hacia servidor VPN |

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| Servidor VPN | G1/0 | 172.30.0.6 /24 | Conexión hacia ISP |
| Servidor VPN | G2/0 | 10.50.10.1 /24 | Red corporativa |

| Red | Dirección |
|---|---|
| Red corporativa | 10.50.10.0 /24 |
| Pool de clientes VPN | 10.200.50.10 - 10.200.50.60 |
| Red WAN | 172.30.0.0 /24 |

---

# Parámetros utilizados

| Parámetro | Valor |
|---|---|
| Tipo de VPN | Client-to-Site |
| Protocolo de túnel | L2TP |
| Seguridad | IPsec |
| Versión IKE | IKEv1 |
| Cifrado | AES-256 |
| Integridad | SHA |
| Grupo Diffie-Hellman | 2 |
| Autenticación IPsec | Pre-shared key |
| Clave compartida | IPSecL2TP |
| Modo IPsec | Transport |
| Transform-set | esp-aes 256 esp-sha-hmac |
| Autenticación PPP | MS-CHAPv2 |
| Usuario VPN | vpnuser |
| Password VPN | VpnPass123 |
| Pool de direcciones VPN | 10.200.50.10 - 10.200.50.60 |

---

# Configuración

## Router ISP

```bash
enable
conf t

hostname ISP

interface g2/0
 description hacia-Ubuntu
 ip address 172.30.0.1 255.255.255.0
 no shutdown

interface g1/0
 description hacia-VPN-SERVER
 ip address 172.30.0.5 255.255.255.0
 no shutdown

! Ruta hacia la red corporativa
ip route 10.50.10.0 255.255.255.0 172.30.0.6

! Ruta hacia el pool VPN
ip route 10.200.50.0 255.255.255.0 172.30.0.6

end
wr
```


### Servidor VPN
```bash
enable
conf t

hostname Servidor

interface GigabitEthernet1/0
 description WAN hacia ISP
 ip address 172.30.0.6 255.255.255.0
 no shutdown

interface GigabitEthernet2/0
 description LAN corporativa
 ip address 10.50.10.1 255.255.255.0
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 172.30.0.1

aaa new-model
aaa authentication ppp VPN-AUTH local
aaa authorization network VPN-AUTH local

username vpnuser password 0 VpnPass123

ip local pool POOL-VPN 10.200.50.10 10.200.50.60

crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key IPSecL2TP address 0.0.0.0

crypto ipsec transform-set TS-L2TP esp-aes 256 esp-sha-hmac
 mode transport
exit

crypto dynamic-map DMAP-L2TP 10
 set transform-set TS-L2TP
exit

crypto map CMAP-L2TP 65535 ipsec-isakmp dynamic DMAP-L2TP

interface GigabitEthernet1/0
 crypto map CMAP-L2TP
exit

vpdn enable

vpdn-group L2TP-SERVER
 accept-dialin
  protocol l2tp
  virtual-template 1
 exit
 no l2tp tunnel authentication
exit

interface Virtual-Template1
 ip unnumbered GigabitEthernet2/0
 peer default ip address pool POOL-VPN
 ppp authentication ms-chap-v2 VPN-AUTH
 ppp authorization VPN-AUTH
 no shutdown
exit

end
write memory
```

### SW1
```bash
enable
configure terminal

hostname SW1

interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

end
write memory
```
### CLIENTE UBUNTU
1. Instalación de Paquetes
El primer paso es instalar las herramientas que gestionan el cifrado y el protocolo de túnel.
```bash
sudo apt update
sudo apt install strongswan xl2tpd -y
```
2. Fase de Cifrado: IPsec (strongSwan)
IPsec se encarga de crear un "tubo" cifrado antes de que pasen los datos.

A. Configuración de la conexión
Edita /etc/ipsec.conf para definir los algoritmos de cifrado y el destino.

```bash
sudo nano /etc/ipsec.conf
```
Contenido del archivo:
Ini, TOML
```bash
config setup
    charondebug="ike 1, knl 1, cfg 0" # Nivel de log para depuración

conn L2TP-PSK
    authby=secret            # Autenticación por clave compartida (PSK)
    auto=add
    keyexchange=ikev1        # L2TP requiere IKE versión 1
    type=transport           # Modo transporte (el túnel va dentro de IPsec)
    left=%defaultroute       # Tu IP local automática
    leftprotoport=17/1701    # Puerto UDP 1701 (L2TP)
    right=172.30.0.6         # IP del servidor VPN remoto
    rightprotoport=17/1701
    ike=aes256-sha1-modp1024! # Algoritmos de fase 1
    esp=aes256-sha1!          # Algoritmos de fase 2
```
B. Definición de la Llave Maestra (PSK)
Debes definir la clave secreta en /etc/ipsec.secrets. Sin esta clave, el túnel no se levantará.

```bash
sudo nano /etc/ipsec.secrets
Formato: IP_SERVIDOR : PSK "tu_clave_secreta"
```

3. Fase de Túnel: L2TP (xl2tpd)
Una vez cifrada la conexión, xl2tpd crea el túnel virtual.

A. Configuración del Cliente L2TP
Edita /etc/xl2tpd/xl2tpd.conf.

```bash
sudo nano /etc/xl2tpd/xl2tpd.conf
```
Contenido:
```bash
Ini, TOML
[lac vpn]
lns = 172.30.0.6                      # LNS (L2TP Network Server)
ppp debug = yes                       # Activa logs detallados
pppoptfile = /etc/ppp/options.l2tpd.client # Archivo de opciones PPP
length bit = yes                      # Requerido por muchos servidores
```
4. Fase de Autenticación de Usuario (PPP)
Aquí se configuran las credenciales del usuario final y parámetros de red.

A. Opciones del cliente PPP
Edita /etc/ppp/options.l2tpd.client.

```bash
sudo nano /etc/ppp/options.l2tpd.client
```
Contenido:
```bash
Plaintext
ipcp-accept-local
ipcp-accept-remote
refuse-eap
require-mschap-v2     # Método de autenticación estándar
noccp                 # Desactiva compresión de datos
noauth
mtu 1280              # MTU ajustado para evitar fragmentación por el overhead de IPsec
mru 1280
noipdefault
defaultroute          # Enviar todo el tráfico por la VPN
usepeerdns            # Usar los DNS que asigne la VPN
connect-delay 5000
```
# Credenciales
```bash
name vpnuser
password VpnPass123
```
5. Ejecución y Conexión
Sigue este orden estricto para levantar la conexión:

Reiniciar servicios para cargar configuraciones:
```bash
sudo ipsec restart
sudo systemctl restart xl2tpd
```
Iniciar IPsec (Establecer el cifrado):

```bash
sudo ipsec up L2TP-PSK
```

Iniciar L2TP (Crear el túnel y autenticar usuario):
```bash
echo "c vpn" | sudo tee /var/run/xl2tpd/l2tp-control
```

### Verificacion
```bash
show crypto ipsec sa
show vpdn session
show crypto isakmp sa
show users
show ip local pool
```



