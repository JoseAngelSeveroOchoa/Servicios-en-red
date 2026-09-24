# UD03: Práctica 2 — Virtual Host + DNS

!!! abstract "Objetivo de la práctica"
    Añadir una IP virtual a tu servidor, crear en tu servidor DNS (UD02) los nombres de dominio que vas a usar para tu web, y configurar 4 *Virtual Host* combinando esos dominios con dos puertos distintos.

    **Materiales:** el servidor Debian con Apache (UD03-Práctica1) y con bind9 (UD02) · una IP libre en tu red interna

---

## 1. Añade una IP virtual a tu interfaz

Vas a necesitar una segunda dirección IP en el mismo servidor, para poder apuntar un segundo nombre de dominio a una IP distinta de la principal. Se hace añadiendo una **interfaz virtual**, un alias sobre tu interfaz real:

```text title="/etc/network/interfaces"
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo enp0s3
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet static
        address 172.1.a.1
        netmask 255.255.255.0
        network 172.1.a.0
        gateway 172.1.a.1

# Interfaz virtual
auto enp0s3:0
iface enp0s3:0 inet static
        address 172.1.a.11
        netmask 255.255.255.0
        network 172.1.a.0
        gateway 172.1.a.1
```

!!! tip "Nombre de la interfaz virtual"
    El alias se llama igual que la interfaz real, añadiendo `:0` (o `:1`, `:2`... si necesitas más de uno). Sustituye `172.1.a.11` por una IP libre dentro de tu red interna.

Aplica el cambio:

```bash
sudo systemctl restart networking
ip a   # comprueba que aparece enp0s3:0 con su IP
```

---

## 2. Da de alta tus dominios en el DNS

Ya sabes hacerlo (UD02). Vas a crear un dominio personal: `Nombre.es`, sustituyendo `Nombre` por tu propio nombre (por ejemplo, `joseangel.es`).

Añade estos registros a tu zona en `/etc/bind/db.Nombre.es` (repasa UD02 si no recuerdas la estructura completa del archivo con su `SOA` y `NS`):

```dns
www          IN   A       172.1.a.1
apellido     IN   CNAME   www.Nombre.es.
segundoApellido   IN   A  172.1.a.11
```

- **`www.Nombre.es`**: un registro `A` apuntando a la IP principal de tu servidor.
- **`apellido.Nombre.es`**: un alias (`CNAME`) de `www.Nombre.es`.
- **`segundoApellido.Nombre.es`**: otro registro `A`, esta vez apuntando a la **IP virtual** que acabas de crear.

No olvides declarar la zona `Nombre.es` en `named.conf.local` (como maestro) y reiniciar Bind:

```bash
sudo named-checkzone Nombre.es /etc/bind/db.Nombre.es
sudo systemctl restart named
```

### Comprueba que el DNS resuelve correctamente

```bash
dig www.Nombre.es
dig apellido.Nombre.es
dig segundoApellido.Nombre.es
```

- [ ] `www.Nombre.es` resuelve a la IP principal.
- [ ] `apellido.Nombre.es` resuelve (vía CNAME) a la misma IP que `www`.
- [ ] `segundoApellido.Nombre.es` resuelve a la IP virtual.

---

## 3. Configura 4 Virtual Host

Ya sabes cómo crear un *Virtual Host* (UD03-Práctica1: archivos en `/etc/apache2/sites-available`, activados con `a2ensite`). Ahora vas a combinar tus **dos nombres de dominio** (`www.Nombre.es` y `segundoApellido.Nombre.es`) con **dos puertos distintos** (por ejemplo, 80 y 8080), para tener 4 *Virtual Host* en total:

| Virtual Host | `ServerName` | Puerto |
|---|---|---|
| VH1 | `www.Nombre.es` | 80 |
| VH2 | `www.Nombre.es` | 8080 |
| VH3 | `segundoApellido.Nombre.es` | 80 |
| VH4 | `segundoApellido.Nombre.es` | 8080 |

!!! tip "Puertos adicionales"
    Si vas a usar un puerto distinto de 80 y 443, recuerda añadirlo a `/etc/apache2/ports.conf` con una línea `Listen 8080`, o Apache no escuchará en él.

Cada *Virtual Host* debe servir un `index.html` distinto que identifique claramente por cuál estás entrando (por ejemplo, "Estás en www.Nombre.es puerto 8080"), para poder comprobar desde el cliente que cada combinación lleva al sitio correcto.

### Comprueba desde el cliente

```bash
curl http://www.Nombre.es
curl http://www.Nombre.es:8080
curl http://segundoApellido.Nombre.es
curl http://segundoApellido.Nombre.es:8080
```

O accede con el navegador a cada una de las 4 combinaciones.

### Checklist

- [ ] Interfaz virtual `enp0s3:0` activa y con IP correcta.
- [ ] Los 3 registros DNS (`www`, `apellido`, `segundoApellido`) resuelven correctamente.
- [ ] Los 4 *Virtual Host* están activos (`a2ensite` + reinicio) y cada uno responde con contenido distinto.
- [ ] Comprobación desde el cliente de las 4 combinaciones dominio × puerto.

---

## ¿Alguna duda?
