# UD01: Práctica DHCP 1


---

## 1. Piensa la estructura de red virtual

Antes de empezar, es necesario tener clara la topología: un router de casa/clase que da acceso a Internet y a la LAN, y dentro de tu ordenador dos máquinas virtuales conectadas entre sí:

- **Máquina virtual 1 – Ubuntu Server** (SERVIDOR/Router): con dos tarjetas de red, una hacia la LAN de casa/clase (`192.168.X.X`) y otra hacia la red interna con las máquinas virtuales (`172.1.a.1`).
- **Máquina virtual 2 – Xubuntu** (CLIENTE): conectada únicamente a la red interna (`172.1.a.2`).

![Esquema de la red virtual](imagenes/01-esquema-red.png)
*Captura: diagrama o resultado de tu propia topología de red virtual.*

---

## 2. Configura las tarjetas de red virtuales

En VirtualBox (o el hipervisor que uses), configura las tarjetas de red de la máquina servidor:

- **Adaptador 1:** conectado en modo *Adaptador puente*, para que salga a la LAN de casa/clase.
- **Adaptador 2:** conectado en modo *Red interna* (o el modo que use tu topología), para comunicarse con la máquina cliente.

![Configuración de las tarjetas de red en VirtualBox](imagenes/02-configuracion-adaptadores.png)
*Captura: pantalla de Configuración > Red de la máquina virtual, con los dos adaptadores configurados.*

---

## 3. Identifica las interfaces de red

Desde la máquina servidor, ejecuta el siguiente comando para listar las interfaces de red disponibles:

```bash
ip a
```

Identifica el nombre de cada interfaz (por ejemplo, `enp0s3` y `enp0s8`) y anota cuál corresponde a la LAN externa y cuál a la red interna con el cliente.

![Salida del comando ip a](imagenes/03-ip-a.png)
*Captura: terminal con el resultado del comando `ip a`, señalando las interfaces relevantes.*

---

## 4. Configura las redes

En la ruta `/etc/network` encontramos el archivo de interfaces de red y podemos configurarlo. Un ejemplo sería el siguiente:

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo enp0s3 enp0s8
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp

iface enp0s8 inet static
        address 172.1.2.1
        netmask 255.255.255.0
```

> **Nota:** cada cambio en la red necesita reinicio: `sudo systemctl restart networking`.

![Archivo /etc/network/interfaces editado](imagenes/04-etc-network-interfaces.png)
*Captura: editor `nano` (o el que uses) mostrando tu archivo de configuración de red ya editado.*

---

## 5. Configura el servidor DHCP

1. Instala el servidor DHCP llamado `isc-dhcp-server` en tu servidor.
2. Configura la interfaz en el archivo del servidor DHCP que se encuentra en `/etc/default`.
3. Configura el archivo de configuración que se encuentra en `/etc/dhcp`.
   - Teniendo en cuenta los ejemplos de configuración de este archivo que hemos visto en teoría (puedes consultarlos en el PDF del tema 2), debes configurar el servidor con los siguientes parámetros:
   - El rango debe ir desde la IP de la interfaz correspondiente número 3, hasta la 10. Es decir, se deben asignar las IPs desde la X.X.X.3 hasta la X.X.X.10 (X.X.X.0 es la red de la interfaz donde debe estar el DHCP).
   - Debes enviar también la puerta de enlace a los clientes a los que asignes la IP.
   - También debes enviar a los clientes los DNS, que serán dos: `1.1.1.1` y `8.8.8.8`.

### 5.1 Instalación del servidor DHCP

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

![Instalación de isc-dhcp-server](imagenes/05-instalacion-dhcp-server.png)
*Captura: terminal con la instalación del paquete completada sin errores.*

### 5.2 Configuración de la interfaz (`/etc/default/isc-dhcp-server`)

![Archivo /etc/default/isc-dhcp-server editado](imagenes/06-etc-default-isc-dhcp-server.png)
*Captura: editor mostrando la interfaz configurada (por ejemplo, `INTERFACESv4="enp0s8"`).*

### 5.3 Configuración del archivo `/etc/dhcp/dhcpd.conf`

![Archivo dhcpd.conf editado](imagenes/07-dhcpd-conf.png)
*Captura: editor mostrando el bloque `subnet` con el `range`, `option routers` y `option domain-name-servers` configurados.*

---

## 6. Hagamos la prueba

Último paso: comprueba que todo funciona según lo diseñado.

- Reinicia el servicio DHCP: `sudo systemctl restart isc-dhcp-server`.
- Desde la máquina cliente (Xubuntu), renueva o solicita la IP por DHCP y comprueba que recibe una dirección dentro del rango configurado, la puerta de enlace y los DNS indicados.

![Servicio DHCP activo en el servidor](imagenes/08-estado-servicio-dhcp.png)
*Captura: `sudo systemctl status isc-dhcp-server` mostrando el servicio activo (`active (running)`).*

![IP asignada por DHCP en el cliente](imagenes/09-ip-cliente.png)
*Captura: en la máquina cliente, resultado de `ip a` (o `ipconfig`/equivalente) mostrando la IP, puerta de enlace y DNS recibidos por DHCP.*

---

## ¿Alguna duda?
