# Presentación del curso

---

## 1. Información General del Módulo

* **Curso:** 2º SMR (Sistemas Microinformáticos y Redes) - Curso 2026-2027
* **Profesor:** José Ángel Segarra Castillo ([ja.segarracastillo@edu.gva.es](mailto:ja.segarracastillo@edu.gva.es))
* **Carga horaria:** 233 horas en total (7 horas/semana)
* **Centro:** IES Severo Ochoa

### Horario Semanal
* **Lunes:** 19:10 - 21:00 h
* **Martes:** 16:05 - 17:55 h
* **Jueves:** 15:10 - 16:05 h
* **Viernes:** 15:10 - 17:00 h


---

## 2. Resultados de Aprendizaje (RA)

* **RA1:** Instala servicios de configuración dinámica, describiendo sus características y aplicaciones.
* **RA2:** Instala servicios de resolución de nombres, describiendo sus características y aplicaciones.
* **RA3:** Instala servicios de transferencia de ficheros, describiendo sus características y aplicaciones.
* **RA4:** Gestiona servidores de correo electrónico identificando requerimientos de utilización y aplicando criterios de configuración.
* **RA5:** Gestiona servidores web identificando requerimientos de utilización y aplicando criterios de configuración.
* **RA6:** Gestiona métodos de acceso remoto describiendo sus características e instalando los servicios correspondientes.
* **RA7:** Despliega redes inalámbricas seguras justificando la configuración elegida y describiendo los procedimientos de implantación.
* **RA8:** Establece el acceso desde redes locales a redes públicas identificando posibles escenarios y aplicando software específico.

---

## 3. Temporalización

El contenido se distribuye en 9 unidades didácticas organizadas a lo largo de dos evaluaciones *(temporalización orientativa)*:

| Unidad Didáctica | Evaluación | Sesiones estimadas |
| :--- | :---: | :---: |
| **UD1:** Introducción a los Servicios en Red. Servicio DHCP | Primera | 16 |
| **UD2:** Servicio DNS | Primera | 16 |
| **UD3:** Servidor Web | Primera | 16 |
| **UD4:** Servicio de acceso y control remoto | Primera | 6 |
| **UD5:** Interconexión de redes privadas con redes públicas | Segunda | 16 |
| **UD6:** Servicio FTP | Segunda | 16 |
| **UD7:** Servicios de correo electrónico | Segunda | 16 |
| **UD8:** Despliegue de redes inalámbricas | Segunda | 16 |
| **UD9:** Proyecto final integrador | Segunda | 16 |

---

## 4. Criterios de Evaluación

### Ponderación por Resultado de Aprendizaje

Todos los resultados de aprendizaje tienen el mismo peso dentro del módulo *(orientativo)*:

| Resultado de Aprendizaje | Porcentaje |
| :--- | :---: |
| **RA1 a RA8** | 12,5% cada uno |

!!! warning "Condiciones de Evaluación Continua y Autoría"
    * **Asistencia mínima:** Faltar al **15% o más** de las horas implica la pérdida de la evaluación continua, teniendo que realizar un examen final global del módulo.
    * **Aprobado independiente:** Es necesario **aprobar por separado** cada uno de los Resultados de Aprendizaje.
    * **Verificación de autoría:** Tras la entrega de cualquier práctica, el docente podrá solicitar que se explique cualquier paso realizado. La imposibilidad de justificar la autoría resultará en la **anulación de la práctica (nota nula)**.
    * **Pruebas objetivas:** Se realizarán al final de cada unidad utilizando los ordenadores del aula.

---

## 5. Herramientas Necesarias

Para el seguimiento de las clases prácticas y teóricas se utilizará:

* Correo del estudiante (Outlook institucional).
* Plataforma de aprendizaje **Aules**.
* Paquete informático Microsoft Office.
* Entornos de virtualización (máquinas virtuales).

---

## 6. Resumen de Contenidos del Módulo


### Servicios y Protocolos que Estudiaremos

#### DHCP (*Dynamic Host Configuration Protocol*)
* Automátiza la asignación de direcciones IP y parámetros de red a los dispositivos.
* **IP Manual vs DHCP:** La IP manual la asigna el usuario; con DHCP, el servidor la entrega automáticamente.

#### DNS (*Domain Name System*)
* Sistema encargado de traducir nombres de dominio comprensibles (ej. `www.google.com`) a direcciones IP numéricas (ej. `142.250.184.196`).

#### Servidor Web
* Servicio que almacena y sirve páginas web mediante protocolos HTTP/HTTPS.
* **Ejemplos de software:** Apache, Nginx.

#### Servidor FTP (*File Transfer Protocol*)
* Permite el intercambio y gestión remota de archivos entre cliente y servidor.
* Utilizado habitualmente para copias de seguridad y publicación web.
* **Clientes de transferencia:** FileZilla, WinSCP.

#### Servidor de Correo Electrónico
* Sistema para envío, recepción y almacenamiento de correo digital.
* **Correo Saliente:** Protocolo **SMTP**.
* **Correo Entrante:**
  * **POP3:** Descarga los mensajes al cliente y suele borrarlos del servidor.
  * **IMAP:** Mantiene los mensajes en el servidor permitiendo la sincronización multidispositivo.

#### Servicio SSH (*Secure Shell*)
* Proporciona acceso remoto seguro a la línea de comandos de un servidor.
* Cifra toda la comunicación (a diferencia de Telnet, que envía datos en plano y es inseguro).

#### Redes Inalámbricas (Wi-Fi)
* **Estándares habituales:**
  * `802.11n` (Wi-Fi 4)
  * `802.11ac` (Wi-Fi 5)
* **Seguridad:** Protocolos WPA2 y WPA3 (estándares actuales seguros) frente a WEP y WPA (obsoletos/inseguros).

#### Acceso a Redes Públicas y Seguridad
* **Riesgos:** Robo de credenciales, interceptación de tráfico (*Man-in-the-Middle*), propagación de malware.
* **Medidas de protección:** Uso de VPNs, navegación cifrada (HTTPS), cortafuegos activos y evitar el acceso a información confidencial.