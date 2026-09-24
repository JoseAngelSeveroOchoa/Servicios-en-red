# UD02: Práctica DNS 2

!!! abstract "Objetivo de la práctica"
    Configurar reenviadores (*forwarders*) hacia DNS públicos, entender la diferencia entre recursión propia y reenvío, añadir registro de logs a Bind, y desplegar un **servidor DNS esclavo** que replique la zona `sre.es` mediante transferencia de zona.

    **Materiales:** el servidor DNS (`dnsserver`) de la Práctica 1 · un segundo Debian sin interfaz gráfica, clonado, para hacer de esclavo

---

## 1. Antes de empezar

Necesitas un **clon** del servidor DNS para que haga de servidor esclavo.

- Ese clon tendrá una interfaz en tu misma red interna `172.1.a.0/24`, configurada para recibir la IP por DHCP (no por IP estática).
- El servidor DHCP (que ya tienes configurado desde UD01) debe asignarle una **IP fija** (reserva) a este nuevo servidor esclavo, para que su dirección no cambie nunca. Repasa la UD01, apartado 7.3, si no recuerdas cómo hacer una reserva DHCP.

!!! warning "Si el clon conserva más de un adaptador de red"
    Si clonaste una máquina que tenía más de un adaptador (por ejemplo, la del ejercicio de DHCP relay de UD01), desconecta o elimina en VirtualBox el que no vayas a usar aquí. El esclavo solo necesita el adaptador NAT (para instalar paquetes) y el de la red interna `172.1.a.0/24`.

---

## 2. Configuración del clon como servidor esclavo

Si ya tenías clonada la máquina de prácticas anteriores, puedes reutilizarla, pero recuerda:

1. **Detén los servicios que pudiera tener instalados** de las prácticas de DHCP, para que no entren en conflicto:
   ```bash
   sudo systemctl stop isc-dhcp-server
   sudo systemctl stop isc-dhcp-relay
   ```
2. La interfaz de la red interna debe estar configurada para recibir la IP por DHCP (usamos `/etc/network/interfaces`, no *netplan*):
   ```text title="/etc/network/interfaces (fragmento)"
   allow-hotplug enp0s8
   iface enp0s8 inet dhcp
   ```
3. Instala Bind y los paquetes adicionales, igual que hiciste en el servidor original (Práctica 1):
   ```bash
   sudo apt update
   sudo apt install bind9 bind9-utils bind9-doc
   ```

---

## 3. Si algo no cuadra: vacía las cachés

Si después de configurar todo no obtienes los resultados esperados, puede que haya quedado información en la caché del cliente o del servidor. Para vaciarla:

```bash
# En el cliente
sudo resolvectl flush-caches

# En el servidor
sudo rndc flush
sudo rndc reload
```

!!! note "`resolvectl` en vez de `systemd-resolve`"
    `systemd-resolve --flush-caches` sigue funcionando como alias en muchas distribuciones, pero está obsoleto. El comando actual es `resolvectl flush-caches`.

---

## 4. Añadir el dominio automáticamente por DHCP

Con la configuración de la Práctica 1, para hacer *ping* hay que incluir el dominio completo:

- ✅ Funciona: `ping dnsserver.sre.es`
- ❌ No funciona: `ping dnsserver`

Podemos arreglarlo enviando nuestro dominio a los clientes por DHCP, para que lo añadan automáticamente a los nombres que no lo incluyan. En la declaración de tu `subnet`, dentro del `dhcpd.conf` (UD01), añade:

```conf hl_lines="1"
option domain-name "sre.es";
```

!!! danger "Ojo con el nombre exacto de la directiva"
    La directiva de ISC DHCP se llama `domain-name`, **no** `domain`. `option domain "sre.es";` no es una directiva válida y no tendrá ningún efecto.

Reinicia el servidor DHCP y renueva la IP en el cliente para que reciba el nuevo parámetro (repasa UD01 si no recuerdas los comandos).

---

## 5. Tarea 1: resolución de dominios de Internet

1. Haz una nueva consulta con `dig` y comprueba que ahora sí se ha usado el servidor que toca, es decir, tu `dnsserver`. Adjunta capturas de la configuración y del resultado.
   ```bash
   dig dnsserver.sre.es
   ```
2. Comprueba que, si consultas un dominio de Internet (`cisco.com`, `github.com`…), se resuelve perfectamente:
   ```bash
   dig cisco.com
   ```

    !!! question "Reflexiona"
        ¿Cómo es posible que se resuelva si solo hemos configurado las zonas autoritativas de `sre.es`, y ninguna más?

        ??? tip "Pista"
            Repasa la UD02, apartado 7: tu Bind actúa como servidor DNS **caché**, con `recursion yes`. Aunque no tenga autoridad sobre `cisco.com`, puede hacer consultas iterativas empezando por los servidores raíz para resolverlo él mismo.

3. Descomenta la sección de `forwarders` en `named.conf.options` y añade estas direcciones:
   ```text hl_lines="2 3"
   forwarders {
       208.67.222.222; // Servidor DNS de OpenDNS
       1.1.1.1;        // Servidor DNS de Cloudflare
   };
   ```
   Cambia también la opción `dnssec-validation` a `no`:
   ```text
   dnssec-validation no;
   ```

    !!! warning "Esto es solo para la práctica"
        Desactivar la validación DNSSEC reduce la seguridad de las resoluciones (no se verifica la autenticidad de las respuestas). Lo hacemos aquí para simplificar el ejercicio, pero **no es una práctica recomendable en un servidor real**.

4. Vuelve a hacer un `dig` a una dirección de Internet.

    !!! question "Reflexiona"
        ¿Qué crees que hemos conseguido al añadir estas direcciones de reenvío?

---

## 6. Tarea 2: recursión y forwarders

1. En `named.conf.options`, cambia la línea `recursion yes;` por `recursion no;` y reinicia Bind.
    a. ¿Sigue funcionando la resolución de subdominios externos de Internet? Compruébalo con `dig` o `nslookup`.
    b. ¿Y la de tus propios equipos? Compruébalo con `dig` y el nombre de dominio completo (`dnsserver.sre.es`, `clidebian.sre.es`).
    c. ¿Por qué pasa esto, si tenemos los *forwarders* configurados?

    ??? tip "Pista"
        Reenviar una consulta a un *forwarder* es, en sí mismo, un tipo de resolución recursiva: tu servidor le pide a otro que resuelva por él. Si `recursion no;`, tu Bind deja de intentar resolver nada que no sea autoritativo — ni siquiera reenviándolo.

!!! tip "No olvides volver a dejar recursion yes;"
    Antes de continuar con el resto de la práctica, vuelve a poner `recursion yes;`, o el resto de comprobaciones no funcionarán como se espera.

---

## 7. Registro de logs

Históricamente, todos los servicios de Linux han tenido archivos de log que registran su actividad. Hoy en día `systemd` centraliza esto (`sudo journalctl -u nombre_del_servicio`), pero vamos a configurar también nuestros propios archivos de log, más accesibles.

### 7.1 Incluir el archivo de logging

En `/etc/bind/named.conf`, indica que se debe tener en cuenta un nuevo archivo de configuración para el logging:

```text title="/etc/bind/named.conf (añadido)"
include "/etc/bind/named.conf.logging";
```

### 7.2 Crear `named.conf.logging`

```text title="/etc/bind/named.conf.logging" hl_lines="6 13"
logging {
    channel errors_syslog {
        syslog daemon;
        severity warning;
    };
    // Para loggear todas las consultas DNS al servidor
    channel queries_log {
        file "/var/log/named/queries.log" versions 600 size 20m;
        print-time yes;
        print-category yes;
        print-severity yes;
        severity dynamic;
    };
    // Para loggear todos los errores al realizar consultas al servidor
    channel query-errors_log {
        file "/var/log/named/query-errors.log" versions 5 size 20m;
        print-time yes;
        print-category yes;
        print-severity yes;
        severity dynamic;
    };
    // Definimos las categorías asociadas a los canales
    category lame-servers { null; };
    category edns-disabled { null; };
    category resolver { null; };
    category dnssec { errors_syslog; };
    category default { errors_syslog; default_debug; };
    category unmatched { null; };
    category queries { queries_log; };
    category query-errors { query-errors_log; };
};
```

Las líneas resaltadas son los archivos donde se volcará toda la información de las consultas y de los errores.

### 7.3 Crear el directorio y los archivos de log

Comprueba que el directorio y los archivos indicados **no existen todavía**, y créalos:

```bash
sudo mkdir -p /var/log/named
sudo touch /var/log/named/queries.log
sudo touch /var/log/named/query-errors.log
```

Cambia la propiedad de ambos archivos al usuario `bind`, que es con el que corre el servidor DNS:

```bash
sudo chown bind:bind /var/log/named/queries.log
sudo chown bind:bind /var/log/named/query-errors.log
```

!!! note "`bind:bind` en vez de `bind.bind`"
    Ambas notaciones (`usuario.grupo` y `usuario:grupo`) funcionan en `chown`, pero `:` es la forma moderna recomendada.

### 7.4 Activar el registro de consultas

Finalmente, en `named.conf.options`, dentro de `options {...}`, añade:

```text
querylog yes;
```

Y reinicia el servicio:

```bash
sudo systemctl restart named
```

### 7.5 Tarea

Realiza algunas consultas desde el cliente y comprueba que quedan registradas:

```bash
tail -f /var/log/named/queries.log
```

Haz una captura de pantalla que lo demuestre.

---

## 8. Configuración del servidor esclavo

Para configurar el servidor esclavo, partimos del clon que preparamos al principio, con Bind ya instalado igual que en el maestro.

Este servidor esclavo será capaz de resolver las mismas consultas que el principal, e incluso seguirá resolviendo si el principal deja de funcionar.

### 8.1 `named.conf.options` del esclavo

Configura `named.conf.options` **exactamente igual** que en el maestro (misma ACL, mismas opciones).

### 8.2 `named.conf.local` del esclavo

Para indicarle al esclavo qué zonas va a recibir, declaramos las mismas que en el maestro, pero en modo esclavo (`type slave`), indicando de qué IP las va a recibir (`masters`):

```text title="/etc/bind/named.conf.local (esclavo)"
zone "sre.es" {
    type slave;
    file "/etc/bind/esclavos/db.sre.es";
    masters { 172.1.a.1; };
};

zone "a.1.172.in-addr.arpa" {
    type slave;
    file "/etc/bind/esclavos/db.a.1.172";
    masters { 172.1.a.1; };
};
```

!!! warning "El directorio `esclavos` debe existir"
    Las zonas transferidas se guardarán en `/etc/bind/esclavos/`, así que crea ese directorio antes de reiniciar Bind:
    ```bash
    sudo mkdir /etc/bind/esclavos
    sudo chown bind:bind /etc/bind/esclavos
    ```
    Si te da un error de permisos con esa ruta, usa en su lugar `/var/cache/bind/<nombre_de_zona>`, que ya pertenece al usuario `bind` por defecto.

### 8.3 Autorizar la transferencia en el maestro

En el servidor DNS **maestro**, hay que indicar a qué esclavo/s se le van a transferir las zonas. Se usan estas líneas:

```text
allow-transfer { 172.1.a.X; };
also-notify { 172.1.a.X; };
```

Sustituye `172.1.a.X` por la IP real de tu servidor esclavo (la reserva fija que le diste por DHCP).

!!! tip "Dos formas de aplicarlo"
    - **De forma genérica**, para todas las zonas del servidor: dentro de `named.conf.options`.
    - **Zona a zona**, con más control: dentro de cada bloque `zone {...}` en `named.conf.local`.

Reinicia Bind tanto en el maestro como en el esclavo:

```bash
sudo systemctl restart named
```

---

## 9. Comprobación final

### 9.1 Verifica la transferencia de zona

Monitoriza el servicio del esclavo mientras se produce la transferencia:

```bash
sudo journalctl -u named -f
```

Deberías ver una línea indicando que la zona se ha transferido correctamente.

### 9.2 Comprueba que el esclavo resuelve

Comprueba que el servidor esclavo resuelve correctamente los nombres de `dnsserver` y `clidebian`, así como los de Internet:

```bash
dig dnsserver.sre.es @172.1.a.X      # X = IP del servidor esclavo
nslookup dnsserver.sre.es 172.1.a.X
```

Indica en la captura de pantalla en qué se ve que es el **esclavo** (y no el maestro) quien está respondiendo.

### 9.3 Comprueba la tolerancia a fallos

Con todo funcionando, apaga el servicio `named` del servidor **principal/maestro**:

```bash
# En el maestro
sudo systemctl stop named
```

El servidor esclavo debería seguir respondiendo a las consultas DNS sin problema.

- Adjunta capturas de los comandos usados para consultar al esclavo, señalando que es él quien responde.
- Adjunta una captura de los logs del esclavo, mostrando que está recibiendo las consultas.

### Checklist de verificación

- [ ] `dig`/`nslookup` a dominios externos funciona con `recursion yes;` y forwarders configurados.
- [ ] Con `recursion no;`, ni los dominios externos ni los propios (por nombre corto) se resuelven — solo por FQDN dentro de zonas propias, si aplica.
- [ ] `/var/log/named/queries.log` registra las consultas de los clientes.
- [ ] El esclavo transfiere correctamente ambas zonas (directa e inversa) desde el maestro.
- [ ] El esclavo sigue resolviendo consultas aunque el maestro esté apagado.

---

## ¿Alguna duda?
