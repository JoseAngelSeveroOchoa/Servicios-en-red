# UD03: Práctica 1 — Instalación y configuración de Apache

!!! abstract "Objetivo de la práctica"
    Instalar Apache en Debian, conocer su estructura de archivos de configuración, publicar una web por usuario y restringir el acceso por dirección IP.

    **Materiales:** Debian (sin interfaz gráfica) · `apache2`

---

## 1. Antes de instalar: comprueba los puertos

Antes de instalar el servidor web, comprueba que los puertos **80** y **443** no están ocupados.

Una forma sencilla es con un navegador, accediendo a la IP de la máquina. Una forma más adecuada es con el escáner `nmap`:

```bash
sudo apt install nmap
sudo nmap -v 127.0.0.1 -p 1-65535
```

Repite la comprobación sustituyendo `127.0.0.1` por la IP que vas a exponer en la red.

---

## 2. Instalación

```bash
sudo apt update
sudo apt install apache2
```

Esto instalará también sus dependencias (`apache2-bin`, `apache2-utils`, `libapr1`...). Comprueba que se ha instalado correctamente accediendo con el navegador a la IP del servidor: deberías ver la página de bienvenida de Apache.

!!! note "Apache queda activo como servicio"
    Tras la instalación, Apache queda configurado como servicio de Debian y arrancará automáticamente con el sistema operativo.

---

## 3. Arrancar, parar y reiniciar Apache

Hay dos formas equivalentes de gestionar el servicio: con las órdenes genéricas de Linux, o con las propias de Apache.

### Con órdenes de Linux

```bash
sudo service apache2 start
sudo service apache2 stop
sudo service apache2 restart
sudo service apache2 reload   # relee la configuración sin cortar el servicio
sudo service apache2 status
```

### Con órdenes de Apache

```bash
sudo apache2ctl start
sudo apache2ctl stop
sudo apache2ctl restart
sudo apache2ctl graceful   # relee la configuración sin cortar el servicio
sudo apache2ctl status
```

!!! tip "`apache2ctl status` necesita un navegador de texto"
    En una instalación sin entorno gráfico, instala `lynx` para poder ver la salida de `status` desde la propia terminal: `sudo apt install lynx`.

---

## 4. Estructura de archivos de configuración

La configuración de Apache está bajo `/etc/apache2`:

| Archivo/Directorio | Para qué sirve |
|---|---|
| `apache2.conf` | Archivo principal. Se puede configurar todo aquí, pero se recomienda repartir la configuración en archivos pequeños e incluirlos con `Include`. Haz siempre copia de seguridad antes de tocarlo. |
| `ports.conf` | Puertos donde escucha Apache (80 por defecto; 443 si tienes SSL/TLS activo). |
| `conf-available/` | Configuraciones disponibles para usar. |
| `conf-enabled/` | Configuraciones activas (enlaces simbólicos a `conf-available`). |
| `mods-available/` | Módulos disponibles y su configuración. |
| `mods-enabled/` | Módulos activos (enlaces simbólicos a `mods-available`). |
| `sites-available/` | Configuraciones de *Virtual Host* disponibles. |
| `sites-enabled/` | *Virtual Host* activos (enlaces simbólicos a `sites-available`). |

El directorio de publicación por defecto es `/var/www/html`, reflejado en `sites-available/000-default.conf`.

### Scripts de utilidad

```bash
sudo a2enconf nombre   # activa una configuración de conf-available
sudo a2disconf nombre  # la desactiva

sudo a2enmod nombre    # activa un módulo
sudo a2dismod nombre   # lo desactiva

sudo a2ensite nombre   # activa un Virtual Host
sudo a2dissite nombre  # lo desactiva
```

!!! warning "Después de usar estos scripts, recarga Apache"
    `a2en*`/`a2dis*` solo crean o borran el enlace simbólico. El cambio no se aplica hasta que reinicies o recargues el servicio (`sudo systemctl reload apache2`).

> En los archivos de configuración, los comentarios se escriben con `#`.

---

## 5. Publica una web por usuario

Los archivos de configuración de tus *Virtual Host* están en `/etc/apache2/sites-available`.

1. Crea, en el directorio *home* de cada usuario, una carpeta `public_html` con permisos `755`:
   ```bash
   mkdir ~/public_html
   chmod 755 ~/public_html
   ```
2. Comprueba que el módulo `userdir` está activo (es el que publica `~/public_html` como web personal de cada usuario):
   ```bash
   sudo a2enmod userdir
   sudo systemctl restart apache2
   ```
3. Coloca un `index.html` de prueba dentro de `public_html` y accede desde el navegador a `http://IP_DEL_SERVIDOR/~usuario/`.

### Checklist

- [ ] `public_html` creada con permisos `755` para cada usuario de prueba.
- [ ] Módulo `userdir` activo.
- [ ] Accedes correctamente a `http://IP/~usuario/` desde el cliente.

---

## 6. Restringe el acceso por IP

!!! danger "Sintaxis moderna, no la de Apache 2.2"
    El material original de esta práctica usaba la sintaxis antigua de Apache 2.2 (`Order Deny,Allow` / `Allow from` / `Deny from`). Esa sintaxis sigue funcionando en Apache 2.4 (el que trae Debian) gracias al módulo de compatibilidad `mod_access_compat`, pero está **obsoleta**: lo correcto en Apache 2.4 es usar `Require`. Usamos la sintaxis moderna directamente.

Dentro de la configuración de tu *Virtual Host* (o en un bloque `<Directory>` de `apache2.conf`), prueba a restringir el acceso a tu propia red:

```apache
<Directory /var/www/html>
    Require ip 192.168.1.0/25
</Directory>
```

Reinicia Apache para aplicar el cambio:

```bash
sudo apache2ctl restart
```

### Ahora prueba distintas variantes y observa qué pasa en cada caso

```apache
<Directory /var/www/html>
    Require all denied
</Directory>
```

```apache
<Directory /var/www/html>
    Require all granted
</Directory>
```

```apache
<Directory /var/www/html>
    <RequireAll>
        Require all granted
        Require not ip 192.168.1.50
    </RequireAll>
</Directory>
```

!!! question "Reflexiona"
    - ¿Qué diferencia hay entre `Require all denied` y no tener ningún `Require`?
    - ¿Qué hace `RequireAll` frente a poner varias líneas `Require` sueltas? (Pista: no es lo mismo exigir que se cumplan **todas** las condiciones que exigir que se cumpla **alguna**; investiga también `<RequireAny>`.)

Recuerda reiniciar Apache (`sudo apache2ctl restart`) después de cada cambio para probarlo.

### Checklist

- [ ] Acceso restringido correctamente a una subred concreta con `Require ip`.
- [ ] Probadas las variantes `all denied` / `all granted` / `RequireAll`, con capturas de cada resultado.

---

## ¿Alguna duda?
