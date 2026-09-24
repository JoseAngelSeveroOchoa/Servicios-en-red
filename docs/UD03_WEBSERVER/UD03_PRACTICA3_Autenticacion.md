# UD03: Práctica 3 — Autenticación por usuario y por grupo

!!! abstract "Objetivo de la práctica"
    Proteger un directorio de tu servidor web con autenticación **Basic**, primero por usuario/contraseña y después por pertenencia a un grupo.

    **Materiales:** el servidor Debian con Apache (UD03-Práctica1)

!!! note "¿Por qué "Basic" y no "Digest"?"
    Repasa la UD03 de teoría, apartado 5: con HTTPS ya generalizado, la autenticación Basic sobre una conexión cifrada es igual de segura que Digest, y mucho más sencilla de configurar. Por eso esta práctica usa `AuthType Basic` en todos los casos.

---

## 1. Crea el archivo de usuarios y contraseñas

Antes de nada, crea un archivo de usuarios/contraseñas en una ruta **inaccesible** desde los *Virtual Host* (fuera de `/var/www`), por ejemplo `/usr/local/apache/passwd/password`:

```bash
sudo mkdir -p /usr/local/apache/passwd
```

**Crear el archivo con el primer usuario** (`-c` crea el archivo; úsalo solo la primera vez, o sobrescribirás el archivo entero):

```bash
sudo htpasswd -c /usr/local/apache/passwd/password alumne1
```

**Añadir un usuario nuevo a un archivo ya existente** (sin `-c`):

```bash
sudo htpasswd /usr/local/apache/passwd/password alumne2
```

Crea también la ruta que vas a proteger, por ejemplo `/var/www/html/protegido`:

```bash
sudo mkdir /var/www/html/protegido
```

---

## 2. Protege el directorio por usuario

Los archivos de configuración de tus *Virtual Host* están en `/etc/apache2/sites-available`. Añade a la configuración:

```apache
<Directory "/var/www/html/protegido">
    AuthType Basic
    AuthName "Área privada con autenticación Basic"
    AuthBasicProvider file
    AuthUserFile "/usr/local/apache/passwd/password"
    Require valid-user
</Directory>
```

Reinicia Apache:

```bash
sudo apache2ctl restart
```

---

## 3. Pruébalo con varios usuarios

Crea 4 usuarios: `alumne1`, `alumne2`, `professor1` y `professor2` (con `htpasswd`, como en el paso 1).

- Prueba a acceder con un usuario y contraseña válidos.
- Prueba a acceder con un usuario que no existe, o con la contraseña equivocada.

!!! question "Reflexiona"
    ¿Qué código de estado HTTP devuelve el servidor en cada caso? ¿Cuántos intentos te deja el navegador antes de mostrar el error?

### Checklist

- [ ] Los 4 usuarios están creados en el archivo de contraseñas.
- [ ] Acceso correcto con un usuario válido.
- [ ] Acceso denegado con un usuario o contraseña incorrectos (captura del error).

---

## 4. Prepara la autenticación por grupo

Crea un archivo de texto (por ejemplo, `/usr/local/apache/passwd/groups`) con este contenido:

```text title="/usr/local/apache/passwd/groups"
grup1: alumne1 alumne2
professors: professor1 professor2
```

Activa el módulo de Apache necesario para poder usar grupos:

```bash
sudo a2enmod authz_groupfile
sudo systemctl restart apache2
```

---

## 5. Protege el directorio por grupo

Edita de nuevo la configuración de tu *Virtual Host*, dando permiso solo al `grup1`:

```apache
<Directory "/var/www/html/protegido">
    AuthType Basic
    AuthName "Área privada"
    AuthBasicProvider file
    AuthUserFile "/usr/local/apache/passwd/password"
    AuthGroupFile "/usr/local/apache/passwd/groups"
    Require group grup1
</Directory>
```

Reinicia Apache:

```bash
sudo apache2ctl restart
```

Prueba a acceder con un usuario del `grup1` (debería funcionar) y con uno de `professors` (debería denegarse, aunque sus credenciales sean correctas).

### Checklist

- [ ] Archivo de grupos creado con `grup1` y `professors`.
- [ ] Módulo `authz_groupfile` activo.
- [ ] Un usuario de `grup1` accede correctamente.
- [ ] Un usuario de `professors` (credenciales válidas, grupo incorrecto) es rechazado.

---

## ¿Alguna duda?
