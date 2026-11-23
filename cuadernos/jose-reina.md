#Mapping The Maze
## Parte 1: Descubrir y Mapear
### Identificar hosts dentro de la red:

Partimos del rango autorizado y descartamos la hoja de inventario del cliente por no ser fiable.
```
sudo nmap -sn 172.17.0.0/24 -oN hosts.nmap
```

![](img/nmap1.png)

**Cómo funciona el descubrimiento de hosts en Nmap.** \
Cuando ejecutamos nmap -sn, la forma predeterminada de realizar el descubrimiento depende de la posición del objetivo respecto a la máquina analista:

Si el objetivo está en el mismo segmento de red que la máquina atacante, Nmap no usa ping ICMP. Utiliza el protocolo ARP: envía una trama a todos los posibles nodos de la red preguntando "¿Quién tiene la IP x.x.x.x?". Quien responda está activo y, además, nos devuelve su dirección IP y su MAC.

Si el objetivo no está en el mismo segmento de red, Nmap usa ICMP Echo Requests (Ping) para tratar de identificar hosts activos.

No obstante, que un nodo no responda a un paquete ICMP Echo Request no significa necesariamente que esté inactivo. Es posible que el administrador de dicho dispositivo haya configurado el firewall para que no responda a paquetes ICMP, o que el host esté detrás de un dispositivo que filtre ese tráfico.

Comprobación con tcpdump: podemos corroborar qué tráfico genera Nmap leyendo la interfaz con tcpdump mientras se ejecuta el escaneo.

Con el comando por defecto sudo nmap -sn 172.17.0.0/24, tcpdump muestra tráfico ARP y respuestas de los hosts vivos), confirmando que en el mismo segmento Nmap no usa ICMP: 

![](img/nmap2.png) 

Si forzamos a Nmap a usar ICMP y desactivamos ARP con sudo nmap --disable-arp-ping -sn 172.17.0.0/24, tcpdump muestra ICMP Echo Request / Echo Reply, y el resultado puede variar: hosts que respondían por ARP pueden no responder a ICMP: 

![](img/nmap3.png) 

En esta ocasión obtenemos el mismo resultado con ambos métodos, pero es muy probable que en otros escenarios usar ICMP no nos muestre los mismos resultados que ARP. Por eso conviene no fiarse de un único método y, si se sospecha filtrado, contrastar con tcpdump y probar --disable-arp-ping.

###Formateo de IPs para automatización:
El archivo hosts.nmap contiene las IPs entre mucha información que, aunque valiosa en algunos contextos, no es el formato más óptimo para automatizar la enumeración de servicios. Si le pasamos varias IPs a nmap, debemos hacerlo separando cada IP con un espacio, por lo que conviene limpiarlo con un comando sencillo:

```
tail -n+2 hosts.nmap | grep -oE "([0-9]+\.){3}[0-9]+" | paste -sd' ' | tee hosts_ips.txt

172.17.0.3 172.17.0.4 172.17.0.5 172.17.0.6 172.17.0.7 172.17.0.8 172.17.0.9 172.17.0.10 172.17.0.1
```

- tail -n+2 hosts.nmap: como la primera línea de hosts.nmap contiene la IP de la red 172.17.0.0/24, empezamos a mostrar desde la segunda línea del archivo para que no la detecte.

- grep -oE "([0-9]+\.){3}[0-9]+": usamos expresiones regulares para detectar las IPs.

- paste -sd' ': ponemos las IPs de forma secuencial y separadas por espacios.

- tee hosts_ips.txt: los guardamos en el archivo hosts_ips.txt.

###Enumeración de puertos global:

Con los hosts identificados, levantamos el mapa de puertos TCP:

```
sudo nmap -p- -sS -Pn -n -oN puertos_global.nmap -oX puertos_global.xml $(cat hosts_ips.txt)
```
Este escaneo genera dos archivos con todos los puertos TCP abiertos. Para automatizar el escaneo de versiones, extraemos las IPs con sus puertos en una sola línea:

``
awk '/Nmap scan report/{if(ip && ports) print ip":"ports; ip=$5; ports=""} /^[0-9]+\/tcp.*open/{split($1,a,"/"); ports=(ports?ports",": "")a[1]} END{if(ip && ports) print ip": "ports}' puertos_global.nmap > puertos_global_ips_ports.txt
``

Resultado:
``
172.17.0.3:21,80
172.17.0.4:22,8009,8080
172.17.0.5:80,443,1883,5672,8161,39789,61613,61614,61616
172.17.0.6:22,8080
172.17.0.7:22
172.17.0.8:22,80,139,445
172.17.0.9:22,80
172.17.0.10:80
``
## Parte 2: Enumerar sin autenticar

Una vez tenemos el mapa de puertos y servicios levantado, profundizamos en qué deja ver cada uno sin necesidad de aportar credenciales. Documentamos la información expuesta, las cuentas de usuario descubiertas y las limitaciones encontradas, sin probar logins ni forzar accesos.

### 2.1 Host `172.17.0.3` (vsftpd y Apache)

**Puertos y Servicios:**
*   21/tcp open ftp vsftpd 2.3.4
*   80/tcp open http Apache httpd 2.4.58 (Ubuntu)

*   **FTP (21):** Al intentar acceder como el usuario `anonymous` sin contraseña, el servidor devuelve un error de configuración. Esto nos indica que el usuario anónimo **está habilitado**, pero el servicio está mal configurado.
*   **HTTP (80):** Se muestra la página por defecto de Apache para Ubuntu. No se realiza fuzzing de directorios para no salir del scope.

#### Vulnerabilidades identificadas
*   **vsftpd 2.3.4:** `CVE-2011-2523` (CWE-78, CVSS 9.8). Contiene una *backdoor* que permite ejecución remota de comandos (RCE). Al conectarse con un usuario terminado en `:)`, se abre el puerto 6200 con una shell de root.
*   **Apache httpd 2.4.58:** `CVE-2023-38709` (CWE-1284, CVSS 7.3). Vulnerabilidad de *HTTP Response Splitting* por falta de validación de datos internos.

---

### 2.2 Host `172.17.0.4` (Tomcat y AJP)

**Puertos y Servicios:**
*   22/tcp open ssh OpenSSH 7.9p1
*   8009/tcp open ajp13 Apache Jserv (Protocol v1.3)
*   8080/tcp open http Apache Tomcat 9.0.30

*   **SSH (22):** Sin vulnerabilidades conocidas en SearchSploit.
*   **HTTP (8080):** Muestra la página por defecto de Tomcat.

#### Vulnerabilidades identificadas
*   **AJP13 (8009):** `CVE-2020-1938` (CWE-285, CVSS 9.8) - *Ghostcat*. El conector AJP está expuesto y acepta peticiones sin autenticación, permitiendo suplantar un servidor interno de confianza.
*   **Tomcat 9.0.30:**
    *   `CVE-2022-34305` (CWE-79, CVSS 6.1): XSS en la aplicación de demostración.
    *   `CVE-2020-1935` y `CVE-2019-17569` (CWE-444, CVSS 4.8): *HTTP Request Smuggling*.

---

### 2.3 Host `172.17.0.5` (ActiveMQ y App Web)

**Puertos y Servicios:**
*   80/tcp open http Apache httpd 2.4.59
*   443/tcp open ssl/http Apache httpd 2.4.59
*   1883/tcp open mqtt
*   5672/tcp open amqp?
*   8161/tcp open http Jetty 9.4.39
*   39789/tcp open tcpwrapped
*   61613/tcp open stomp Apache ActiveMQ
*   61614/tcp open http Jetty 9.4.39
*   61616/tcp open apachemq ActiveMQ OpenWire transport 5.15.15

*   **HTTPS (443):** El certificado está autofirmado y caducado (desde 2025-04-17). Sirve una aplicación web de un restaurante en fase de desarrollo.
    *   *Dependencias:* jQuery 3.4.1 (`CVE-2020-11023` y `CVE-2020-11022`, XSS) y Bootstrap 3.4.1 (`CVE-2019-8331`, XSS).
*   **HTTP (8161):** Muestra un diálogo de autenticación básica. Por convención, corresponde a la consola de administración de ActiveMQ.
*   **HTTP (61614):** `curl` devuelve un `200 OK` con contenido vacío (`Content-Length: 0`).

#### Vulnerabilidades identificadas
*   **ActiveMQ OpenWire 5.15.15:** `CVE-2023-46604` (CWE-502, CVSS 9.8). Ejecución remota de código (RCE) crítica mediante deserialización/instanciación arbitraria de clases.

---

### 2.4 Host `172.17.0.6` (Jenkins)

**Puertos y Servicios:**
*   22/tcp open ssh OpenSSH 9.2p1
*   8080/tcp open http Jetty 10.0.18

*   **HTTP (8080):** Panel de inicio de sesión de Jenkins. La versión detectada es la `2.441`.

#### Vulnerabilidades identificadas
*   **Jenkins 2.441:**
    *   `CVE-2024-23897` (CWE-22/CWE-27, CVSS 9.8): Lectura arbitraria de archivos mediante la expansión de argumentos `@ruta` en la CLI.
    *   `CVE-2024-23898` (CWE-346, CVSS 8.8): *Cross-Site WebSocket Hijacking* (CSWSH) por falta de validación del origen.

---

### 2.5 Host `172.17.0.7` (OpenSSH antiguo)

**Puertos y Servicios:**
*   22/tcp open ssh OpenSSH 7.7

#### Vulnerabilidades identificadas
*   `CVE-2019-16905` (CWE-190, CVSS 7.8): Desbordamiento de enteros en claves XMSS (experimental, no habilitado por defecto).
*   `CVE-2018-15473` (CWE-362, CVSS 5.3): Enumeración de usuarios mediante respuestas diferenciales.

---

### 2.6 Host `172.17.0.8` (Samba y Apache)

**Puertos y Servicios:**
*   22/tcp open ssh OpenSSH 9.6p1
*   80/tcp open http Apache httpd 2.4.58
*   139/tcp open netbios-ssn Samba smbd 4
*   445/tcp open netbios-ssn Samba smbd 4

*   **Samba (445) - Enumeración anónima:**
    *   `smbclient -L //172.17.0.8 -N` lista los recursos: `myshare` (accesible), `backup24` (denegado), `home` (denegado) y `$IPC`.
    *   En `myshare` encontramos el archivo `access.txt` (956 bytes) con posible información sensible. *No se descarga.*
    *   Mediante sesión nula y `rpcclient`, se enumeran 5 usuarios: `administrador` (RID 0x3ec), `satriani7` (0x3eb), `usuario1` (0x3e8), `usuario2` (0x3e9) y `usuario3` (0x3ea).
    *   Se expone la política de contraseñas: `min_password_length: 5` y sin requisitos de complejidad (configuración débil).
    *   `netshareenum` expone las rutas reales del sistema: `/srv/samba/myshare`, `/srv/samba/backup24`, `/var/www/html` y `/tmp`.

``
smbclient -L //172.17.0.8 -N
smbclient //172.17.0.8/myshare -N -c 'ls'
rpcclient -N -U '' 172.17.0.8 -c 'querydispinfo; enumdomusers'
rpcclient -N -U '' 172.17.0.8 -c 'enumdomgroups;getdompwinfo'
rpcclient -N -U '' 172.17.0.8 -c 'netshareenum; netshareenumall'
``

2.7 Host 172.17.0.9 (App web de prueba)
Puertos y Servicios:

22/tcp open ssh OpenSSH 9.2p1

80/tcp open http Apache httpd 2.4.68

HTTP (80): whatweb revela el título test page. Hay un panel de login (login.php) pero el código HTML y CSS no filtran información sensible.

Vulnerabilidades identificadas
OpenSSH 9.2p1: Sin vulnerabilidades.

Apache httpd 2.4.68: La única vulnerabilidad reportada se soluciona en esta versión, por lo que no le afecta.

2.8 Host 172.17.0.10 (IPMI expuesto)
Puertos y Servicios:

80/tcp open http SimpleHTTPServer 0.6 (Python 3.11.2)

623/udp open asf-rmcp (IPMI 2.0)

HTTP (80): Laboratorio de pruebas sobre IPMI. Nos muestra información sobre el BMC y la vulnerabilidad CVE-2013-4786.

IPMI (623/UDP): Confirmado mediante nmap --script ipmi-version. Soporta IPMI 2.0 y métodos de autenticación débiles.

``
sudo nmap -Pn -n -sU --script ipmi-version -p623 172.17.0.10
``

Vulnerabilidades identificadas
IPMI 2.0: CVE-2013-4786 (CWE-255, CVSS 7.5). Divulgación de credenciales (hashes HMAC) mediante RAKP Message 2, lo que permite ataques de fuerza bruta offline.

## Parte 3: De la versión a la vulnerabilidad (CVE) y priorización:

Una vez identificados los servicios y sus versiones en la fase de enumeración, el siguiente paso es cruzar esa información con bases de datos públicas de vulnerabilidades. El objetivo es caracterizar el riesgo (CVE, CWE, CVSS) y priorizar las acciones de remediación, teniendo en cuenta la limitación del cliente (solo pueden arreglar unas pocas cosas este trimestre).

### 3.1 Tabla de hallazgos CVE

A continuación se detallan las vulnerabilidades potenciales identificadas para las versiones de software descubiertas:

| Host | Puerto | Servicio (versión) | CVE | CWE | CVSS | Descripción |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `172.17.0.3` | `21` | vsftpd 2.3.4 | CVE-2011-2523 | CWE-78 | 9.8 | Backdoor (RCE) |
| `172.17.0.3` | `80` | Apache httpd 2.4.58 | CVE-2023-38709 | CWE-1284 | 7.3 | HTTP Response Splitting |
| `172.17.0.4` | `8009` | Apache Jserv (AJP 1.3) | CVE-2020-1938 | CWE-285 | 9.8 | Ghostcat |
| `172.17.0.4` | `8080` | Apache Tomcat 9.0.30 | CVE-2022-34305 | CWE-79 | 6.1 | XSS |
| `172.17.0.4` | `8080` | Apache Tomcat 9.0.30 | CVE-2020-1935 | CWE-444 | 4.8 | HTTP Request Smuggling |
| `172.17.0.4` | `8080` | Apache Tomcat 9.0.30 | CVE-2019-17569 | CWE-444 | 4.8 | HTTP Request Smuggling |
| `172.17.0.5` | `443` | jQuery 3.4.1 | CVE-2020-11023 | CWE-79 | 6.1 | XSS |
| `172.17.0.5` | `443` | jQuery 3.4.1 | CVE-2020-11022 | CWE-79 | 6.1 | XSS |
| `172.17.0.5` | `443` | Bootstrap 3.4.1 | CVE-2019-8331 | CWE-79 | 6.1 | XSS |
| `172.17.0.5` | `61616` | ActiveMQ OpenWire 5.15.15 | CVE-2023-46604 | CWE-502 | 9.8 | RCE |
| `172.17.0.6` | `8080` | Jenkins 2.441 | CVE-2024-23897 | CWE-22/CWE-27 | 9.8 | Arbitrary File Read |
| `172.17.0.6` | `8080` | Jenkins 2.441 | CVE-2024-23898 | CWE-346 | 8.8 | Cross-Site WebSocket Hijacking |
| `172.17.0.7` | `22` | OpenSSH 7.7 | CVE-2019-16905 | CWE-190 | 7.8 | Desbordamiento de Enteros (XMSS) |
| `172.17.0.7` | `22` | OpenSSH 7.7 | CVE-2018-15473 | CWE-362 | 5.3 | User Enumeration |
| `172.17.0.10` | `623/UDP` | IPMI 2.0 | CVE-2013-4786 | CWE-255 | 7.5 | Divulgación de credenciales |

### 3.2 Tabla de hallazgos de configuración

Además de los fallos de software, se han detectado debilidades en la configuración de los servicios que exponen la red interna a riesgos innecesarios:

| Host | Puerto | Servicio (versión) | CWE | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `172.17.0.3` | `21` | vsftpd 2.3.4 | CWE-287 | Autenticación anónima habilitada (mal configurada) |
| `172.17.0.8` | `445` | Samba smbd 4 | CWE-200, CWE-306, CWE-287, CWE-284 | Fuga de información mediante sesión anónima |
| `172.17.0.8` | `445` | Samba smbd 4 | CWE-345 | Datos de autenticación insuficientes |
| `172.17.0.8` | `445` | Samba smbd 4 | CWE-319 | Tráfico viaja sin cifrado a nivel de protocolo |
| `172.17.0.8` | `445` | Samba smbd 4 | - | Política de contraseñas débil (`min_password_length: 5`) |

### 3.3 Priorización de riesgos

Teniendo en cuenta que el cliente solo dispone de presupuesto para arreglar unas pocas cosas este trimestre, se propone el siguiente orden de remediación. Este orden se basa en la criticidad técnica (CVSS), la facilidad de explotación y el impacto potencial en la operativa del negocio (parada de facturación):

1.  **Actualizar ActiveMQ (`172.17.0.5`):** El RCE de `CVE-2023-46604` es crítico (CVSS 9.8) y relativamente sencillo de explotar. Permitiría a un atacante tomar el control del servidor de mensajería.
2.  **Eliminar la backdoor de vsftpd (`172.17.0.3`):** El servicio `vsftpd 2.3.4` contiene una puerta trasera por diseño (`CVE-2011-2523`). Es un riesgo inaceptable que debe ser actualizado o retirado de inmediato.
3.  **Actualizar Jenkins (`172.17.0.6`):** La lectura arbitraria de archivos (`CVE-2024-23897`) puede exponer credenciales de CI/CD y secretos de la compañía.
4.  **Restringir el conector AJP (`172.17.0.4`):** Deshabilitar el puerto `8009` o protegerlo mediante un secreto para evitar el ataque *Ghostcat* (`CVE-2020-1938`).
5.  **Asegurar Samba (`172.17.0.8`):** Deshabilitar las sesiones nulas (anónimas), forzar la firma SMB (para evitar ataques de tipo *Relay*) y aplicar una política de contraseñas robusta.
6.  **Mitigar la exposición de IPMI (`172.17.0.10`):** Restringir el acceso al puerto `623/UDP` exclusivamente a la red de administración para evitar la captura de hashes (`CVE-2013-4786`).
7.  **Actualizar Apache y dependencias web (`172.17.0.3`, `.5`, `.8`):** Parchear `CVE-2023-38709`, actualizar las librerías jQuery y Bootstrap, y retirar la aplicación en desarrollo del puerto `443`.

## Parte 4: Priorización e informe al cliente

### 4.1 Fichas de vulnerabilidades: CVSS, EPSS y KEV

Para ordenar las vulnerabilidades con criterio profesional, no basta con mirar el CVSS. Es necesario consultar la probabilidad de explotación (EPSS) y el catálogo de vulnerabilidades explotadas conocidas (KEV de CISA). A continuación se detallan las fichas de los hallazgos más críticos.

*Fecha de consulta de datos EPSS/KEV: 2026-11-23.*

| CVE | Host / Servicio | CVSS v3.1 (Vector) | Implicación CIA | EPSS (30d) | CISA KEV | Ransomware |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CVE-2023-46604** | `172.17.0.5:61616`<br>ActiveMQ OpenWire 5.15.15 | **10.0** (CNA)<br>9.8 (NVD)<br>`AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:H/A:H` | **C: Baja**<br>**I: Alta**<br>**A: Alta**<br>Permite ejecución remota de código (RCE) sin autenticación. | **99.99%**<br>(Percentil 100) | **Sí**<br>(Añadido: 2023-11-02) | **Sí**<br>(Uso confirmado) |
| **CVE-2020-1938** | `172.17.0.4:8009`<br>Apache Jserv (AJP 1.3) | **9.8**<br>`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | **C: Alta**<br>**I: Alta**<br>**A: Alta**<br>*Ghostcat*: lectura de archivos y posible RCE. | **94.47%**<br>(Percentil 100) | **Sí**<br>(Añadido: 2022-03-03) | **Desconocido** |
| **CVE-2024-23897** | `172.17.0.6:8080`<br>Jenkins 2.441 | **9.8**<br>`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | **C: Alta**<br>**I: Alta**<br>**A: Alta**<br>Lectura arbitraria de archivos vía CLI. | **~100%**<br>(Percentil >99.9) | **Sí**<br>(Añadido: 2024-08-19) | **Sí**<br>(Uso en malware) |
| **CVE-2011-2523** | `172.17.0.3:21`<br>vsftpd 2.3.4 | **9.8**<br>`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | **C: Alta**<br>**I: Alta**<br>**A: Alta**<br>Backdoor que abre shell de root en el puerto 6200. | **96.0%**<br>(Percentil 99) | **No** | **No consta** |
| **CVE-2013-4786** | `172.17.0.10:623/UDP`<br>IPMI 2.0 | **7.5**<br>`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` | **C: Alta**<br>**I: Ninguna**<br>**A: Ninguna**<br>Captura de hashes HMAC de contraseñas. | **78.6% - 98.0%**<br>(Según fuente) | **No** | **No consta** |

---

### 4.2 Orden de prioridad justificado

Para decidir qué arreglar primero, hemos cruzado los datos de explotación activa (KEV) con la probabilidad de ataque (EPSS) y el impacto operativo en Gadir Logística. El orden recomendado es el siguiente:

1.  **Prioridad 1: CVE-2023-46604 (ActiveMQ en `172.17.0.5`)**
    *   **Justificación:** Es la amenaza más grave. Está en KEV, tiene un EPSS casi del 100% y se ha confirmado su uso en campañas de *ransomware*. El servicio gestiona la mensajería interna; un RCE aquí permitiría al atacante detener la operativa, manipular datos o usarlo como puerta de entrada al resto de la red.

2.  **Prioridad 2: CVE-2020-1938 - Ghostcat (Tomcat en `172.17.0.4:8009`)**
    *   **Justificación:** Aunque su CVSS es 9.8 (similar a otras), lleva en KEV desde 2022 y tiene un EPSS del 94%. El conector AJP está expuesto sin autenticación. Si este servidor aloja el portal de clientes, el impacto reputacional y operativo es directo.

3.  **Prioridad 3: CVE-2024-23897 (Jenkins en `172.17.0.6`)**
    *   **Justificación:** Está en KEV con uso en malware confirmado y EPSS máximo. La lectura arbitraria de archivos puede exponer credenciales de CI/CD (claves de API, contraseñas de repositorios) que comprometan toda la cadena de despliegue de la empresa.

4.  **Prioridad 4: CVE-2011-2523 (vsftpd en `172.17.0.3:21`)**
    *   **Justificación:** Tiene un CVSS 9.8, pero **no está en KEV**. Aunque el EPSS es alto (96%), no hay evidencia de explotación masiva actual. Sin embargo, al ser una puerta trasera por diseño, si el servicio FTP está expuesto, cualquier atacante puede obtener control total del servidor de forma trivial.

5.  **Prioridad 5: CVE-2013-4786 (IPMI en `172.17.0.10:623/UDP`)**
    *   **Justificación:** No está en KEV, pero el EPSS es alto. Permite capturar hashes de contraseñas de administración del hardware. Al afectar a la gestión de la placa base (BMC), el riesgo es que un atacante pueda apagar o reiniciar servidores físicos si consigue romper el hash.

*El resto de hallazgos (XSS en jQuery/Bootstrap, *smuggling* en Tomcat, enumeración de usuarios en OpenSSH, y configuraciones débiles de Samba) tienen un impacto menor o requieren condiciones de explotación más específicas. Se recomienda abordarlos en el ciclo de parcheo habitual.*

---

### 4.3 Respuesta al aviso del cliente

**Contexto:** El cliente pregunta por una vulnerabilidad reciente que «están aprovechando los atacantes».

**Lo que podemos sostener con nuestras evidencias:**
*   **Inventario actualizado:** Tenemos mapeados los 8 equipos de la red con sus puertos y versiones. Si el CVE del aviso corresponde a alguna de las tecnologías que ya hemos identificado como críticas (ActiveMQ, Tomcat AJP, Jenkins, vsftpd o IPMI), **sí les afecta y ya está recogido en este informe**.
*   **Vulnerabilidades de explotación activa:** Nuestro informe ya incluye las vulnerabilidades que la propia CISA (KEV) tiene catalogadas como explotadas.

**Lo que queda pendiente de comprobar:**
*   Si el aviso menciona un **producto o versión distinta** a las que tenemos en el inventario (por ejemplo, un fallo en un router, en una impresora o en un software que no esté en la red `172.17.0.0/24`), en principio no les afectaría.
*   Si el aviso se refiere a una configuración muy específica que no podemos ver sin autenticarnos en los sistemas.

**Recomendación:** Solicitar al cliente el identificador exacto (CVE) o el enlace de la noticia. En menos de una hora podremos contrastar la versión afectada con las versiones de los 8 equipos de su red y confirmarles si están o no expuestos.
