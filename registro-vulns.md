### Ficha 01 — Backdoor en vsftpd 2.3.4

- **Activo:** `172.17.0.3` | **Puerto:** `21/tcp` | **Servicio:** `vsftpd 2.3.4`
- **CVE:** `CVE-2011-2523`
- **CWE:** CWE-78 (_Improper Neutralization of Special Elements used in an OS Command_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_) / T1059 (_Command and Scripting Interpreter_)
- **Descripción:** La versión 2.3.4 de vsftpd distribuida originalmente en 2011 contenía una puerta trasera introducida maliciosamente que abre una shell en el puerto `6200/tcp` al recibir una secuencia específica (`:)`) en el nombre de usuario.
- **Evidencias observadas:** El escaneo de versiones de Nmap (`-sV`) identifica el banner `vsftpd 2.3.4` en el puerto `21/tcp`.
- **Limitaciones y qué queda por comprobar:** El banner por sí solo no garantiza que el binario compilado incluya la puerta trasera (podría ser una versión recompilada limpia o un _honeypot_). Quedaría pendiente verificar el hash del binario en el servidor o confirmar con el equipo de sistemas el origen del paquete, sin activar la puerta trasera.

---

### Ficha 02 — HTTP Response Splitting en Apache httpd 2.4.58

- **Activo:** `172.17.0.3` (y `172.17.0.8`) | **Puerto:** `80/tcp` | **Servicio:** `Apache httpd 2.4.58 ((Ubuntu))`
- **CVE:** `CVE-2023-38709`
- **CWE:** CWE-1284 (_Improper Validation of Specified Quantity in Input_) / CWE-113 (_HTTP Response Splitting_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_)
- **Descripción:** Fallo de validación en el núcleo de Apache HTTP Server (versiones hasta la 2.4.58) que permite a aplicaciones o módulos maliciosos inyectar cabeceras `Content-Type` no válidas, posibilitando ataques de división de respuesta HTTP.
- **Evidencias observadas:** Nmap detecta `Apache httpd 2.4.58 ((Ubuntu))` en el puerto `80/tcp`.
- **Limitaciones y qué queda por comprobar:** Ubuntu aplica parches de seguridad mediante _backporting_ manteniendo el número de versión `2.4.58`. Queda pendiente contrastar la versión exacta del paquete `apache2` (ej. mediante `dpkg -l apache2`) con el equipo de sistemas para descartar que ya esté parcheado.

---

### Ficha 03 — Ghostcat (Lectura/Inclusión de archivos en AJP)

- **Activo:** `172.17.0.4` | **Puerto:** `8009/tcp` | **Servicio:** `Apache Jserv (Protocol v1.3)` / `Apache Tomcat 9.0.30`
- **CVE:** `CVE-2020-1938`
- **CWE:** CWE-285 (_Improper Authorization_) / CWE-22 (_Path Traversal_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_) / T1005 (_Data from Local System_)
- **Descripción:** El conector AJP de Apache Tomcat (versiones 9.0.x anteriores a 9.0.31) confía por defecto en las conexiones entrantes sin autenticación, permitiendo a un atacante remoto leer archivos internos de la aplicación web (como `WEB-INF/web.xml`) o lograr ejecución de código si existe una funcionalidad de subida de archivos.
- **Evidencias observadas:** Nmap confirma que el puerto `8009/tcp` (`ajp13`) está expuesto en red junto con `Apache Tomcat 9.0.30` en el puerto `8080/tcp`.
- **Limitaciones y qué queda por comprobar:** No se ha intentado leer ningún archivo interno para respetar la regla de no explotación. Queda pendiente revisar `server.xml` en el servidor para confirmar si el conector AJP exige secreto compartido (`requiredSecret`) o si está restringido por configuración.

---

### Ficha 04 — HTTP Request Smuggling y XSS en Apache Tomcat 9.0.30

- **Activo:** `172.17.0.4` | **Puerto:** `8080/tcp` | **Servicio:** `Apache Tomcat 9.0.30`
- **CVE:** `CVE-2020-1935`, `CVE-2019-17569`, `CVE-2022-34305`
- **CWE:** CWE-444 (_Inconsistent Interpretation of HTTP Requests_) y CWE-79 (_Cross-site Scripting_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_) / T1189 (_Drive-by Compromise_)
- **Descripción:** La versión 9.0.30 de Tomcat presenta fallos en el análisis de cabeceras `Transfer-Encoding` que permiten _HTTP Request Smuggling_ (`CVE-2020-1935`, `CVE-2019-17569`), así como un fallo de XSS en la aplicación de ejemplos (`CVE-2022-34305`).
- **Evidencias observadas:** Nmap identifica `Apache Tomcat 9.0.30` en el puerto `8080/tcp`.
- **Limitaciones y qué queda por comprobar:** El _Request Smuggling_ requiere habitualmente la presencia de un proxy inverso delante de Tomcat, y el XSS depende de que la aplicación `/examples` esté desplegada. Queda pendiente verificar la arquitectura de red y si los ejemplos por defecto siguen activos.

---

### Ficha 05 — Ejecución Remota de Código (RCE) en Apache ActiveMQ

- **Activo:** `172.17.0.5` | **Puerto:** `61616/tcp` | **Servicio:** `ActiveMQ OpenWire transport 5.15.15`
- **CVE:** `CVE-2023-46604`
- **CWE:** CWE-502 (_Deserialization of Untrusted Data_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_) / T1059 (_Command and Scripting Interpreter_)
- **Descripción:** Apache ActiveMQ (versiones anteriores a 5.15.16 / 5.16.7 / 5.17.6 / 5.18.3) permite a un atacante remoto con acceso al puerto OpenWire manipular tipos de clase serializados para instanciar clases arbitrarias en el classpath, derivando en ejecución remota de código.
- **Evidencias observadas:** Nmap detecta el servicio `ActiveMQ OpenWire transport 5.15.15` expuesto en el puerto `61616/tcp`.
- **Limitaciones y qué queda por comprobar:** No se han enviado cargas útiles (_payloads_) de deserialización. Queda pendiente confirmar con el administrador si existe algún parche manual o restricción de red/autenticación aplicada sobre el broker OpenWire.

---

### Ficha 06 — Cross-Site Scripting (XSS) en librerías front-end (jQuery y Bootstrap)

- **Activo:** `172.17.0.5` | **Puerto:** `443/tcp` | **Servicio:** `HTTPS (jQuery 3.4.1 / Bootstrap 3.4.1)`
- **CVE:** `CVE-2020-11022`, `CVE-2020-11023` (jQuery) y `CVE-2019-8331` (Bootstrap)
- **CWE:** CWE-79 (_Improper Neutralization of Input During Web Page Generation_)
- **MITRE ATT&CK:** T1189 (_Drive-by Compromise_) / T1059.007 (_JavaScript_)
- **Descripción:** jQuery 3.4.1 puede ejecutar código malicioso al pasar HTML no confiable a métodos de manipulación del DOM; Bootstrap 3.4.1 presenta XSS en el atributo `data-template` de los componentes _tooltip_ y _popover_.
- **Evidencias observadas:** Identificación de las versiones `jQuery 3.4.1` y `Bootstrap 3.4.1` servidas por la aplicación web en el puerto `443/tcp`.
- **Limitaciones y qué queda por comprobar:** La presencia de una librería vulnerable no implica explotabilidad automática: queda pendiente revisar el código fuente de la aplicación para comprobar si datos controlados por el usuario llegan sin sanear a esas funciones específicas.

---

### Ficha 07 — Lectura Arbitraria de Archivos y CSWSH en Jenkins 2.441

- **Activo:** `172.17.0.6` | **Puerto:** `8080/tcp` | **Servicio:** `Jetty 10.0.18 (Jenkins 2.441)`
- **CVE:** `CVE-2024-23897` y `CVE-2024-23898`
- **CWE:** CWE-22 (_Path Traversal_) y CWE-346 (_Origin Validation Error_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_) / T1005 (_Data from Local System_)
- **Descripción:** Jenkins 2.441 (y anteriores) utiliza la librería `args4j` en su CLI con la expansión de argumentos `@` habilitada, permitiendo leer archivos arbitrarios del servidor (`CVE-2024-23897`). Además, no valida la cabecera `Origin` en conexiones WebSocket (`CVE-2024-23898`).
- **Evidencias observadas:** Servicio `Jetty 10.0.18` exponiendo `Jenkins 2.441` en el puerto `8080/tcp`.
- **Limitaciones y qué queda por comprobar:** No se ha invocado el CLI para leer archivos. Queda pendiente confirmar si el acceso anónimo de lectura (_Overall/Read_) está habilitado (lo que amplía el impacto de `CVE-2024-23897`) o si el CLI está deshabilitado.

---

### Ficha 08 — Enumeración de usuarios y desbordamiento de enteros en OpenSSH 7.7

- **Activo:** `172.17.0.7` | **Puerto:** `22/tcp` | **Servicio:** `OpenSSH 7.7`
- **CVE:** `CVE-2018-15473` y `CVE-2019-16905`
- **CWE:** CWE-200 (_Exposure of Sensitive Information_) / CWE-190 (_Integer Overflow_)
- **MITRE ATT&CK:** T1087 (_Account Discovery_)
- **Descripción:** OpenSSH 7.7 permite inferir si un nombre de usuario existe en el sistema analizando respuestas ante paquetes de autenticación malformados (`CVE-2018-15473`) y presenta un desbordamiento de enteros al procesar claves privadas XMSS (`CVE-2019-16905`).
- **Evidencias observadas:** Nmap detecta el banner `OpenSSH 7.7 (protocol 2.0)` en el puerto `22/tcp`.
- **Limitaciones y qué queda por comprobar:** Queda pendiente comprobar si la distribución subyacente ha aplicado parches _backported_ y si el servidor tiene habilitado soporte experimental para claves XMSS (necesario para `CVE-2019-16905`).

---

### Ficha 09 — Divulgación de hashes de autenticación en IPMI 2.0 (RAKP)

- **Activo:** `172.17.0.10` | **Puerto:** `623/udp` | **Servicio:** `IPMI 2.0 (asf-rmcp)`
- **CVE:** `CVE-2013-4786`
- **CWE:** CWE-200 (_Exposure of Sensitive Information_) / CWE-287 (_Improper Authentication_)
- **MITRE ATT&CK:** T1110.002 (_Brute Force: Password Cracking_) / T1552 (_Unsecured Credentials_)
- **Descripción:** Fallo de diseño en el protocolo RMCP+ (RAKP) de IPMI 2.0 que envía el hash HMAC de la contraseña en el mensaje _RAKP Message 2_ antes de completar la autenticación, permitiendo capturar hashes y someterlos a fuerza bruta _offline_. Adicionalmente, el servicio anuncia métodos débiles (`password`, `md5`, `md2`, `null`) y soporte para IPMI 1.5.
- **Evidencias observadas:** El script no intrusivo `ipmi-version` confirma `IPMI-2.0`, `Level: 1.5, 2.0` y `UserAuth: password, md5, md2, null` en el puerto `623/udp`.
- **Limitaciones y qué queda por comprobar:** No se han capturado hashes ni probado contraseñas. Queda pendiente verificar con el equipo de infraestructura si el puerto `623/udp` está segmentado y si las cuentas configuradas utilizan contraseñas robustas.

---

### Ficha 10 — Configuraciones inseguras confirmadas (Sin CVE)

- **`172.17.0.8` (`445/tcp` y `139/tcp`, Samba smbd 4):**
    - **Sesión anónima en `IPC$` y fuga de información** (CWE-200, CWE-306 | ATT&CK: T1087, T1135): Evidencia directa mediante `rpcclient` (`enumdomusers`, `getdompwinfo`, `netshareenumall` exponiendo `/var/www/html` en `home`). La enumeración con sesión nula devuelve 5 cuentas: `administrador` (RID 0x3ec), `satriani7` (0x3eb), `usuario1` (0x3e8), `usuario2` (0x3e9) y `usuario3` (0x3ea).
    - **_SMB Signing_ no obligatorio y falta de cifrado SMBv3** (CWE-345, CWE-319 | ATT&CK: T1557, T1040): Evidencia directa mediante Nmap `smb2-security-mode` y `smb2-capabilities`.
    - **Política de contraseñas débil** (CWE-521 | ATT&CK: T1110): Evidencia directa mediante `rpcclient getdompwinfo`: `min_password_length: 5` y sin requisitos de complejidad, lo que facilita ataques de fuerza bruta sobre las cuentas enumeradas.
    - **Recurso `myshare` accesible de forma anónima** (CWE-284, CWE-200 | ATT&CK: T1135, T1039): Evidencia directa mediante `smbclient -L //172.17.0.8 -N` y `smbclient //172.17.0.8/myshare -N -c 'ls'`, que lista el recurso como accesible y muestra el archivo `access.txt` (956 bytes) con posible información sensible. Los recursos `backup24` y `home` devuelven acceso denegado. El archivo no se ha descargado.
    - **Limitación:** No se ha descargado ni abierto `access.txt` para respetar la regla de no intrusión; queda pendiente confirmar con el administrador su contenido y si existe necesidad de negocio para el acceso anónimo a `myshare` o para el puerto `139/tcp`.
- **`172.17.0.3` (`21/tcp`, vsftpd 2.3.4):**
    - **Posible acceso anónimo en FTP** (CWE-287 | ATT&CK: T1078): Queda pendiente verificar en modo solo lectura si el inicio de sesión `anonymous` está realmente habilitado y qué directorio expone.

---

### Ficha 11 — Certificado TLS autofirmado y caducado y aplicación en desarrollo expuesta (Sin CVE)

- **Activo:** `172.17.0.5` | **Puerto:** `443/tcp` (y `80/tcp`) | **Servicio:** `Apache httpd 2.4.59`
- **CVE:** Sin CVE (configuración insegura)
- **CWE:** CWE-295 (_Improper Certificate Validation_) / CWE-298 (_Improper Validation of Certificate Expiration_)
- **MITRE ATT&CK:** T1557 (_Adversary-in-the-Middle_) / T1190 (_Exploit Public-Facing Application_)
- **Descripción:** El servicio HTTPS presenta un certificado autofirmado y caducado desde `2025-04-17`, por lo que los clientes no pueden validar la identidad del servidor y se habitúan a ignorar las advertencias del navegador, lo que facilita ataques de interceptación. Además, el servidor sirve la aplicación web de un restaurante que se encuentra en fase de desarrollo, lo que amplía la superficie de ataque sin aportar valor al negocio de Gadir Logística.
- **Evidencias observadas:** El escaneo de versiones de Nmap (`-sV`) y la inspección del certificado en el puerto `443/tcp` muestran un certificado autofirmado con fecha de caducidad `2025-04-17`. La aplicación en desarrollo es visible en los puertos `80/tcp` y `443/tcp`.
- **Limitaciones y qué queda por comprobar:** No se ha interactuado con la aplicación más allá de identificar sus dependencias (ver Ficha 06). Queda pendiente confirmar con el cliente si la aplicación tiene alguna función de negocio o puede retirarse, y si existe un proceso de renovación de certificados.

---

### Ficha 12 — Servicios de mensajería y consola de administración de ActiveMQ expuestos (Sin CVE)

- **Activo:** `172.17.0.5` | **Puertos:** `1883/tcp` (MQTT), `5672/tcp` (AMQP), `8161/tcp` (Jetty 9.4.39), `61613/tcp` (STOMP), `61614/tcp` (Jetty 9.4.39) y `39789/tcp` (`tcpwrapped`) | **Servicio:** `Apache ActiveMQ`
- **CVE:** Sin CVE (exposición de servicios)
- **CWE:** CWE-668 (_Exposure of Resource to Wrong Sphere_)
- **MITRE ATT&CK:** T1190 (_Exploit Public-Facing Application_)
- **Descripción:** Además del puerto OpenWire (`61616/tcp`, Ficha 05), el broker expone en red varios protocolos de mensajería (MQTT, AMQP, STOMP) y la consola de administración web. Cada servicio adicional amplía la superficie de ataque y, si no está restringido a las redes que lo necesitan, permite a un atacante interactuar con el broker.
- **Evidencias observadas:** Nmap detecta los puertos abiertos indicados. El puerto `8161/tcp` solicita autenticación básica y, por convención, corresponde a la consola de administración de ActiveMQ. El puerto `61614/tcp` responde con `200 OK` y `Content-Length: 0` mediante `curl`.
- **Limitaciones y qué queda por comprobar:** No se han probado credenciales (ni las de fábrica) ni se ha interactuado con los protocolos de mensajería. Queda pendiente confirmar si MQTT, AMQP y STOMP exigen autenticación, si la consola `8161/tcp` debería estar accesible desde esta red, qué hay detrás de `61614/tcp` y `39789/tcp`, y si el acceso está restringido por firewall.
