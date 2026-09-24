# UD03: Práctica 4 — Virtual Host con SSL/TLS

!!! abstract "Objetivo de la práctica"
    Generar un certificado digital autofirmado y configurar un *Virtual Host* que sirva contenido cifrado por HTTPS.

    **Materiales:** el servidor Debian con Apache (UD03-Práctica1) · `openssl`

---

## 1. Crea 4 Virtual Host (repaso)

Antes de meternos con SSL, crea 4 *Virtual Host*, cada uno sirviendo un `.html` distinto que identifique claramente por cuál estás entrando.

!!! tip "Sin pasos detallados, a propósito"
    Ya has hecho esto en las prácticas 1 y 2 de esta unidad — inténtalo sin ayuda. Si te atascas, consulta tus propias notas de esas prácticas antes de preguntar.

---

## 2. Pasos previos para SSL/TLS

### 2.1 Habilita el módulo SSL

```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

### 2.2 Crea un directorio para los archivos del certificado

```bash
sudo mkdir /etc/apache2/ssl
```

### 2.3 Instala las herramientas necesarias

```bash
sudo apt install openssl ca-certificates
```

### 2.4 Genera la clave privada y la solicitud de certificado (CSR)

```bash
cd /etc/apache2/ssl
sudo openssl genrsa -out server.key 2048
sudo openssl req -new -key server.key -out server.csr
```

!!! note "El nombre es cosa tuya"
    `server.key` y `server.csr` son nombres de archivo elegidos libremente; puedes usar otros si quieres, pero luego tendrás que referenciarlos igual en la configuración del *Virtual Host*.

Al ejecutar el segundo comando, rellena los datos que te pide. Por ejemplo:

```text
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Alicante
Locality Name (eg, city) []:Benidorm
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Severo Ochoa
Organizational Unit Name (eg, section) []:SMR
Common Name (e.g. server FQDN or YOUR name) []:tudominio.es
Email Address []:tucorreo@tudominio.es
```

!!! warning "El `Common Name` importa"
    El navegador comprueba que el `Common Name` (o los nombres alternativos del certificado) coincida con el dominio al que te conectas. Si no coincide, el navegador avisará de un problema adicional además del propio de ser un certificado autofirmado.

### 2.5 Genera el certificado (autofirmado, válido 1 año)

```bash
sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

---

## 3. Crea el Virtual Host con SSL

1. Ve a `/etc/apache2/sites-available`.
2. Copia la plantilla que trae Apache, cambiándole el nombre:
   ```bash
   sudo cp default-ssl.conf midominio_ssl.conf
   ```
3. Edita el archivo creado (`sudo nano midominio_ssl.conf`):

```apache hl_lines="4 8 9"
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        ServerName "tudominio.es"
        DocumentRoot /var/www/html/ssl

        # ...

        SSLCertificateFile      /etc/apache2/ssl/server.crt
        SSLCertificateKeyFile   /etc/apache2/ssl/server.key
    </VirtualHost>
</IfModule>
```

!!! danger "Edita los datos que ya hay, no añadas líneas nuevas"
    La plantilla `default-ssl.conf` ya trae casi todo lo necesario. Cambia únicamente los valores marcados (`ServerName`, `DocumentRoot`, las rutas de `SSLCertificateFile`/`SSLCertificateKeyFile`), y añade `ServerName` solo si no lo trae ya. No dupliques directivas.

4. Habilita el sitio:
   ```bash
   sudo a2ensite midominio_ssl.conf
   ```
5. Comprueba que no hay errores de sintaxis:
   ```bash
   sudo apache2ctl configtest
   ```
6. Si todo está correcto, reinicia Apache:
   ```bash
   sudo apache2ctl restart
   ```
7. Copia un `.html` a la carpeta de este *Virtual Host* (`/var/www/html/ssl`) y prueba la conexión por `https://`.

!!! note "El navegador avisará de que el certificado es autofirmado"
    Es normal y esperado: no lo ha emitido una entidad certificadora reconocida (una CA), así que el navegador no se fía por defecto. Pero la conexión **sí** va cifrada — es justo lo que estamos comprobando en esta práctica.

### Checklist

- [ ] Módulo `ssl` activo.
- [ ] Certificado (`server.crt`) y clave (`server.key`) generados en `/etc/apache2/ssl`.
- [ ] `midominio_ssl.conf` creado, editado y habilitado con `a2ensite`.
- [ ] `apache2ctl configtest` sin errores.
- [ ] Acceso por `https://` funcionando (con el aviso esperado de certificado autofirmado).

---

## ¿Alguna duda?
