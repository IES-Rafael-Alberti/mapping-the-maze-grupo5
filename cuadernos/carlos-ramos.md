# 1. Descubrir y mapear

## Identificar hosts dentro de la red

Primero tenemos que identificar todos los hosts activos de la red utilizando nmap:
``` bash
sudo nmap -sn 172.17.0.0/24 -oN hosts.nmap
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-23 18:58 +0000
Nmap scan report for 172.17.0.3
Host is up (0.000041s latency).
MAC Address: 0A:2A:A7:EB:68:12 (Unknown)
Nmap scan report for 172.17.0.4
Host is up (0.000017s latency).
MAC Address: D2:63:24:55:CF:A7 (Unknown)
Nmap scan report for 172.17.0.5
Host is up (0.000017s latency).
MAC Address: A2:6F:15:C7:51:5C (Unknown)
Nmap scan report for 172.17.0.6
Host is up (0.000016s latency).
MAC Address: 2A:D7:A9:1B:98:3E (Unknown)
Nmap scan report for 172.17.0.7
Host is up (0.000016s latency).
MAC Address: E2:46:92:21:99:88 (Unknown)
Nmap scan report for 172.17.0.8
Host is up (0.000018s latency).
MAC Address: 02:BD:EC:0E:81:9E (Unknown)
Nmap scan report for 172.17.0.9
Host is up (0.000015s latency).
MAC Address: DE:A1:F7:40:EB:39 (Unknown)
Nmap scan report for 172.17.0.10
Host is up (0.000027s latency).
MAC Address: 66:AE:B1:77:FD:C9 (Unknown)
Nmap scan report for 172.17.0.1
Host is up.
Nmap done: 256 IP addresses (9 hosts up) scanned in 4.03 seconds
```
![Pasted image 20261123223732.png](https://github.com/ealtgar/mapping-the-maze/blob/main/img/Pasted%20image%2020261123223732.png)

_Aparentemente existen 9 máquina activas en la red. 8 relevantes y 1 aparte que es la nuestra_

>[!important]
>`nmap -sn` descubre hosts de dos formas, según dónde estén:
>- **Misma red (segmento) que yo:** usa **ARP** ("¿quién tiene la IP x.x.x.x?"). Quien responde está vivo y además revela su IP y su MAC.
>- **Otra red:** usa **ICMP Echo Request** (ping).
>**Idea clave:** que un equipo no responda al ping no significa que esté apagado, porque un firewall puede bloquear ICMP. ARP es más fiable en la red local, ya que no se suele filtrar.
>- `sudo nmap -sn 172.17.0.0/24` usa ARP por defecto.
>- `sudo nmap --disable-arp-ping -sn 172.17.0.0/24` fuerza ICMP.
>``` bash
sudo nmap -sn 172.17.0.0/24
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-23 21:40 +0000
Nmap scan report for 172.17.0.3
Host is up (0.000011s latency).
MAC Address: 0A:2A:A7:EB:68:12 (Unknown)
Nmap scan report for 172.17.0.4
Host is up (0.000010s latency).
MAC Address: D2:63:24:55:CF:A7 (Unknown)
Nmap scan report for 172.17.0.5
Host is up (0.0000080s latency).
MAC Address: A2:6F:15:C7:51:5C (Unknown)
Nmap scan report for 172.17.0.6
Host is up (0.0000080s latency).
MAC Address: 2A:D7:A9:1B:98:3E (Unknown)
Nmap scan report for 172.17.0.7
Host is up (0.0000080s latency).
MAC Address: E2:46:92:21:99:88 (Unknown)
Nmap scan report for 172.17.0.8
Host is up (0.0000090s latency).
MAC Address: 02:BD:EC:0E:81:9E (Unknown)
Nmap scan report for 172.17.0.9
Host is up (0.0000060s latency).
MAC Address: DE:A1:F7:40:EB:39 (Unknown)
Nmap scan report for 172.17.0.10
Host is up (0.000015s latency).
MAC Address: 66:AE:B1:77:FD:C9 (Unknown)
Nmap scan report for 172.17.0.1
Host is up.
Nmap done: 256 IP addresses (9 hosts up) scanned in 4.00 seconds
>```
![img](https://github.com/ealtgar/mapping-the-maze/blob/main/img/Pasted%20image%2020261123224242.png)
>
>_ARP por defecto_
>
>``` bash
>sudo nmap --disable-arp-ping -sn 172.17.0.0/24
>Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-23 19:04 +0000
Nmap scan report for 172.17.0.3
Host is up (0.000054s latency).
MAC Address: 0A:2A:A7:EB:68:12 (Unknown)
Nmap scan report for 172.17.0.4
Host is up (0.000026s latency).
MAC Address: D2:63:24:55:CF:A7 (Unknown)
Nmap scan report for 172.17.0.5
Host is up (0.000024s latency).
MAC Address: A2:6F:15:C7:51:5C (Unknown)
Nmap scan report for 172.17.0.6
Host is up (0.000023s latency).
MAC Address: 2A:D7:A9:1B:98:3E (Unknown)
Nmap scan report for 172.17.0.7
Host is up (0.000022s latency).
MAC Address: E2:46:92:21:99:88 (Unknown)
Nmap scan report for 172.17.0.8
Host is up (0.000022s latency).
MAC Address: 02:BD:EC:0E:81:9E (Unknown)
Nmap scan report for 172.17.0.9
Host is up (0.000025s latency).
MAC Address: DE:A1:F7:40:EB:39 (Unknown)
Nmap scan report for 172.17.0.10
Host is up (0.00021s latency).
MAC Address: 66:AE:B1:77:FD:C9 (Unknown)
Nmap scan report for 172.17.0.1
Host is up.
Nmap done: 256 IP addresses (9 hosts up) scanned in 28.07 seconds
>```
![img](https://github.com/ealtgar/mapping-the-maze/blob/main/img/Pasted%20image%2020261123224002.png)
>
>_Fuerza ICMP_
>
## Formateo de IPs para automatización

`hosts.nmap` tiene mucha información, y para pasar varias IPs a `nmap` hacen falta solo las IPs, separadas por espacios.

**Solución:** un comando que extrae las IPs y las deja en una sola línea:
``` bash
tail -n+2 hosts.nmap | grep -oE "([0-9]+\.){3}[0-9]+" | paste -sd' ' | tee hosts_ips.txt

172.17.0.3 172.17.0.4 172.17.0.5 172.17.0.6 172.17.0.7 172.17.0.8 172.17.0.9 172.17.0.10 172.17.0.1
```

- `tail -n+2`: salta la primera línea (la del comentario de nmap, que contiene el rango de red).
- `grep -oE`: con una expresión regular, extrae solo las IPs.
- `paste -sd' '`: las junta en una línea separadas por espacios.
- `tee`: las muestra y las guarda en `hosts_ips.txt`.

Después, habría que borrar a mano tu propia IP de la lista, porque también aparece en el escaneo, usando un editor de texto por ejemplo.

Para guardar la relación IP y MAC en otro archivo, se usa un comando parecido que extrae IPs y MACs y las une por parejas con `sed` (`hosts_datos.txt`).
``` bash
tail -n+2 | grep -oE "([0-9]+\.){3}[0-9]+|([A-Z0-9]+\:){5}[A-Z0-9]+" | sed 'N;s/\n/: /' | tee hosts_datos.txt
```

## Enumeración de puertos globales

Al tener las máquinas activas en la red podemos enumerar sus puertos abiertos siguiendo el siguiente comando:
``` bash
sudo nmap -p- -sS -Pn -n -oN puertos_global.nmap -oX puertos_global.xml $(cat ~/Documents/HE/RECON_ACTIVE/informatico_fantasma/hosts_ips.txt)
```

Ese comando crea dos archivos con todos los puertos TCP abiertos de las máquinas identificadas. Para automatizar el escaneo de versiones, extraemos las IPs con sus puertos en una sola línea:
``` bash
awk '/Nmap scan report/{if(ip && ports) print ip":"ports; ip=$5; ports=""} /^[0-9]+\/tcp.*open/{split($1,a,"/"); ports=(ports?ports",": "")a[1]} END{if(ip && ports) print ip": "ports}' puertos_global.nmap > puertos_global_ips_ports.txt
```

El resultado es el siguiente:
``` bash
172.17.0.3:21,80
172.17.0.4:22,8009,8080
172.17.0.5:80,443,1883,5672,8161,39789,61613,61614,61616
172.17.0.6:22,8080
172.17.0.7:22
172.17.0.8:22,80,139,445
172.17.0.9:22,80
172.17.0.10:80
```

# 2. Enumerar sin autenticar

Una vez tenemos el mapa de puntos y servicios levantados, profundizamos en qué deja ver cada uno sin usar credenciales.

Documentamos la información expuesta, las cuentas descubiertas y las limitaciones encontradas sin forzar nada.

### 172.17.0.3: vsftpd y Apache

**Servicios:** 21/tcp vsftpd 2.3.4 · 80/tcp Apache httpd 2.4.58 (Ubuntu)

- **FTP (21):** el usuario `anonymous` sin contraseña devuelve un error de configuración, lo que indica que el acceso anónimo está habilitado pero mal configurado.
- **HTTP (80):** página por defecto de Apache. No se hizo fuzzing de directorios para no salir del scope.

**Vulnerabilidades:**

- **vsftpd 2.3.4, CVE-2011-2523** (CWE-78, CVSS 9.8): backdoor con RCE. Un usuario terminado en `:)` abre el puerto 6200 con una shell de root.
- **Apache 2.4.58, CVE-2023-38709** (CWE-1284, CVSS 7.3): HTTP Response Splitting por falta de validación de datos internos.

### 172.17.0.4: Tomcat y AJP

**Servicios:** 22/tcp OpenSSH 7.9p1 · 8009/tcp AJP13 · 8080/tcp Apache Tomcat 9.0.30

- **SSH (22):** sin vulnerabilidades conocidas en SearchSploit.
- **HTTP (8080):** página por defecto de Tomcat.

**Vulnerabilidades:**

- **AJP13, CVE-2020-1938 "Ghostcat"** (CWE-285, CVSS 9.8): el conector AJP está expuesto y acepta peticiones sin autenticación, lo que permite suplantar un servidor interno de confianza.
- **Tomcat 9.0.30:**
    - CVE-2022-34305 (CWE-79, CVSS 6.1): XSS en la aplicación de demostración.
    - CVE-2020-1935 y CVE-2019-17569 (CWE-444, CVSS 4.8): HTTP Request Smuggling.

### 172.17.0.5: ActiveMQ y aplicación web

**Servicios:** 80/tcp y 443/tcp Apache 2.4.59 · 1883 MQTT · 5672 AMQP · 8161 Jetty 9.4.39 · 39789 tcpwrapped · 61613 STOMP · 61614 Jetty 9.4.39 · 61616 ActiveMQ OpenWire 5.15.15

- **HTTPS (443):** certificado autofirmado y caducado (desde 2025-04-17). Sirve la web de un restaurante en desarrollo.
- **Dependencias:** jQuery 3.4.1 (CVE-2020-11023 y CVE-2020-11022, XSS) y Bootstrap 3.4.1 (CVE-2019-8331, XSS).
- **HTTP (8161):** pide autenticación básica; por convención es la consola de administración de ActiveMQ.
- **HTTP (61614):** curl devuelve 200 OK con contenido vacío (Content-Length: 0).

**Vulnerabilidades:**

- **ActiveMQ OpenWire 5.15.15, CVE-2023-46604** (CWE-502, CVSS 9.8): RCE crítica por deserialización/instanciación arbitraria de clases.

### 172.17.0.6: Jenkins

**Servicios:** 22/tcp OpenSSH 9.2p1 · 8080/tcp Jetty 10.0.18

- **HTTP (8080):** panel de login de Jenkins, versión 2.441.

**Vulnerabilidades:**

- **CVE-2024-23897** (CWE-22/CWE-27, CVSS 9.8): lectura arbitraria de archivos mediante la expansión de argumentos `@ruta` en la CLI.
- **CVE-2024-23898** (CWE-346, CVSS 8.8): Cross-Site WebSocket Hijacking por falta de validación del origen.

### 172.17.0.7: OpenSSH antiguo

**Servicios:** 22/tcp OpenSSH 7.7

**Vulnerabilidades:**

- **CVE-2019-16905** (CWE-190, CVSS 7.8): desbordamiento de enteros en claves XMSS (experimental, no habilitado por defecto).
- **CVE-2018-15473** (CWE-362, CVSS 5.3): enumeración de usuarios mediante respuestas diferenciales.

### 172.17.0.8: Samba y Apache

**Servicios:** 22/tcp OpenSSH 9.6p1 · 80/tcp Apache 2.4.58 · 139 y 445/tcp Samba smbd 4

**Samba (445), enumeración anónima:**

- `smbclient -L //172.17.0.8 -N` lista los recursos: `myshare` (accesible), `backup24` (denegado), `home` (denegado) e `IPC$`.
- En `myshare` está `access.txt` (956 bytes) con posible información sensible; no se descargó por razones de seguridad por la información sensible.
- Con sesión nula y `rpcclient` se enumeran 5 usuarios: administrador (RID 0x3ec), satriani7 (0x3eb), usuario1 (0x3e8), usuario2 (0x3e9) y usuario3 (0x3ea).
- Política de contraseñas débil: `min_password_length: 5` y sin requisitos de complejidad.
- `netshareenum` expone rutas reales del sistema: `/srv/samba/myshare`, `/srv/samba/backup24`, `/var/www/html` y `/tmp`.

**Comandos utilizados:**

``` bash
smbclient -L //172.17.0.8 -N
smbclient //172.17.0.8/myshare -N -c 'ls'
rpcclient -N -U '' 172.17.0.8 -c 'querydispinfo; enumdomusers'
rpcclient -N -U '' 172.17.0.8 -c 'enumdomgroups;getdompwinfo'
rpcclient -N -U '' 172.17.0.8 -c 'netshareenum; netshareenumall'
```

**Vulnerabilidades:** no se indican CVE; los hallazgos son de configuración (sesión nula, política de contraseñas débil y exposición de rutas).

### 172.17.0.9: aplicación web de prueba

**Servicios:** 22/tcp OpenSSH 9.2p1 · 80/tcp Apache 2.4.68

- **HTTP (80):** `whatweb` muestra el título "test page". Hay un panel de login (`login.php`), pero el HTML y CSS no filtran información sensible.

**Vulnerabilidades:**

- **OpenSSH 9.2p1:** ninguna.
- **Apache 2.4.68:** la única vulnerabilidad reportada se corrige en esta versión, por lo que no le afecta.

### 172.17.0.10: IPMI expuesto

**Servicios:** 80/tcp SimpleHTTPServer 0.6 (Python 3.11.2) · 623/udp asf-rmcp (IPMI 2.0)

- **HTTP (80):** laboratorio de pruebas sobre IPMI; muestra información sobre el BMC y la vulnerabilidad CVE-2013-4786.
- **IPMI (623/UDP):** confirmado con `nmap --script ipmi-version`. Soporta IPMI 2.0 y métodos de autenticación débiles.

``` bash
sudo nmap -Pn -n -sU --script ipmi-version -p623 172.17.0.10
```

**Vulnerabilidades:**

- **CVE-2013-4786** (CWE-255, CVSS 7.5): divulgación de credenciales (hashes HMAC) mediante RAKP Message 2, lo que permite ataques de fuerza bruta offline.

# 3. De la versión a la vulnerabilidad y priorización

Tras la enumeración de servicios y versiones, se cruzó la información con bases de datos públicas (CVE, CWE, CVSS) para caracterizar el riesgo y priorizar la remediación, teniendo en cuenta que el cliente solo puede arreglar unas pocas cosas este trimestre.

### 3.1 Tabla de hallazgos CVE

|Host|Puerto|Servicio (versión)|CVE|CWE|CVSS|Descripción|
|---|---|---|---|---|---|---|
|172.17.0.3|21|vsftpd 2.3.4|CVE-2011-2523|CWE-78|9.8|Backdoor (RCE)|
|172.17.0.3|80|Apache httpd 2.4.58|CVE-2023-38709|CWE-1284|7.3|HTTP Response Splitting|
|172.17.0.4|8009|Apache Jserv (AJP 1.3)|CVE-2020-1938|CWE-285|9.8|Ghostcat|
|172.17.0.4|8080|Apache Tomcat 9.0.30|CVE-2022-34305|CWE-79|6.1|XSS|
|172.17.0.4|8080|Apache Tomcat 9.0.30|CVE-2020-1935|CWE-444|4.8|HTTP Request Smuggling|
|172.17.0.4|8080|Apache Tomcat 9.0.30|CVE-2019-17569|CWE-444|4.8|HTTP Request Smuggling|
|172.17.0.5|443|jQuery 3.4.1|CVE-2020-11023|CWE-79|6.1|XSS|
|172.17.0.5|443|jQuery 3.4.1|CVE-2020-11022|CWE-79|6.1|XSS|
|172.17.0.5|443|Bootstrap 3.4.1|CVE-2019-8331|CWE-79|6.1|XSS|
|172.17.0.5|61616|ActiveMQ OpenWire 5.15.15|CVE-2023-46604|CWE-502|9.8|RCE|
|172.17.0.6|8080|Jenkins 2.441|CVE-2024-23897|CWE-22/CWE-27|9.8|Arbitrary File Read|
|172.17.0.6|8080|Jenkins 2.441|CVE-2024-23898|CWE-346|8.8|Cross-Site WebSocket Hijacking|
|172.17.0.7|22|OpenSSH 7.7|CVE-2019-16905|CWE-190|7.8|Desbordamiento de Enteros (XMSS)|
|172.17.0.7|22|OpenSSH 7.7|CVE-2018-15473|CWE-362|5.3|User Enumeration|
|172.17.0.10|623/UDP|IPMI 2.0|CVE-2013-4786|CWE-255|7.5|Divulgación de credenciales|

### 3.2 Tabla de hallazgos de configuración

|Host|Puerto|Servicio (versión)|CWE|Descripción|
|---|---|---|---|---|
|172.17.0.3|21|vsftpd 2.3.4|CWE-287|Autenticación anónima habilitada (mal configurada)|
|172.17.0.8|445|Samba smbd 4|CWE-200, CWE-306, CWE-287, CWE-284|Fuga de información mediante sesión anónima|
|172.17.0.8|445|Samba smbd 4|CWE-345|Datos de autenticación insuficientes|
|172.17.0.8|445|Samba smbd 4|CWE-319|Tráfico viaja sin cifrado a nivel de protocolo|
|172.17.0.8|445|Samba smbd 4|-|Política de contraseñas débil (min_password_length: 5)|

### 3.3 Priorización de riesgos

El orden de remediación se basa en el CVSS, la facilidad de explotación y el impacto en el negocio (por ejemplo, una parada de facturación):

1. **ActiveMQ (172.17.0.5):** actualizar. El RCE de CVE-2023-46604 es crítico (9.8) y relativamente fácil de explotar; permitiría controlar el servidor de mensajería.
2. **vsftpd (172.17.0.3):** actualizar o retirar de inmediato. La versión 2.3.4 contiene una backdoor por diseño (CVE-2011-2523).
3. **Jenkins (172.17.0.6):** actualizar. CVE-2024-23897 puede exponer credenciales de CI/CD y secretos.
4. **AJP (172.17.0.4):** deshabilitar el puerto 8009 o protegerlo con un secreto para evitar Ghostcat (CVE-2020-1938).
5. **Samba (172.17.0.8):** deshabilitar sesiones nulas, forzar la firma SMB (contra ataques Relay) y aplicar una política de contraseñas robusta.
6. **IPMI (172.17.0.10):** restringir el puerto 623/UDP a la red de administración para evitar la captura de hashes (CVE-2013-4786).
7. **Apache y dependencias web (172.17.0.3, .5, .8):** parchear CVE-2023-38709, actualizar jQuery y Bootstrap, y retirar del puerto 443 la aplicación en desarrollo.

# 4. Priorizar e información al cliente

### 4.1 Fichas de vulnerabilidades: CVSS, EPSS y KEV

Para ordenar las vulnerabilidades con criterio profesional no basta con el CVSS: hay que consultar también la probabilidad de explotación (**EPSS**) y el catálogo de vulnerabilidades explotadas de CISA (**KEV**). 
_Fecha de consulta de los datos: **2026-11-23**._

### 4.1 Fichas de los hallazgos más críticos

|CVE|Host / Servicio|CVSS v3.1 (Vector)|Implicación CIA|EPSS (30d)|CISA KEV|Ransomware|
|---|---|---|---|---|---|---|
|CVE-2023-46604|172.17.0.5:61616 ActiveMQ OpenWire 5.15.15|10.0 (CNA) / 9.8 (NVD) AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:H/A:H|C: Baja, I: Alta, A: Alta. RCE sin autenticación|99.99% (Percentil 100)|Sí (añadido 2023-11-02)|Sí (uso confirmado)|
|CVE-2020-1938|172.17.0.4:8009 Apache Jserv (AJP 1.3)|9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H|C: Alta, I: Alta, A: Alta. Ghostcat: lectura de archivos y posible RCE|94.47% (Percentil 100)|Sí (añadido 2022-03-03)|Desconocido|
|CVE-2024-23897|172.17.0.6:8080 Jenkins 2.441|9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H|C: Alta, I: Alta, A: Alta. Lectura arbitraria de archivos vía CLI|~100% (Percentil >99.9)|Sí (añadido 2024-08-19)|Sí (uso en malware)|
|CVE-2011-2523|172.17.0.3:21 vsftpd 2.3.4|9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H|C: Alta, I: Alta, A: Alta. Backdoor con shell de root en el puerto 6200|96.0% (Percentil 99)|No|No consta|
|CVE-2013-4786|172.17.0.10:623/UDP IPMI 2.0|7.5 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N|C: Alta, I: Ninguna, A: Ninguna. Captura de hashes HMAC|78.6% - 98.0% (según fuente)|No|No consta|

### 4.2 Orden de prioridad justificado

Se cruzaron los datos de explotación activa (KEV) y probabilidad de ataque (EPSS) con el impacto operativo en Gadir Logística:

1. **CVE-2023-46604, ActiveMQ (172.17.0.5):** la amenaza más grave. Está en KEV, tiene EPSS casi del 100% y uso confirmado en ransomware. Un RCE en la mensajería interna permitiría detener la operativa, manipular datos o usarlo como puerta de entrada al resto de la red.
2. **CVE-2020-1938 Ghostcat, Tomcat (172.17.0.4:8009):** en KEV desde 2022 y EPSS del 94%. El conector AJP está expuesto sin autenticación; si el servidor aloja el portal de clientes, el impacto reputacional y operativo es directo.
3. **CVE-2024-23897, Jenkins (172.17.0.6):** en KEV con uso en malware confirmado y EPSS máximo. Puede exponer credenciales de CI/CD (claves de API, contraseñas de repositorios) y comprometer toda la cadena de despliegue.
4. **CVE-2011-2523, vsftpd (172.17.0.3:21):** CVSS 9.8 y EPSS alto (96%), pero no está en KEV, así que no hay evidencia de explotación masiva actual. Aun así, al ser una backdoor por diseño, cualquiera con acceso al FTP puede tomar el control total de forma trivial.
5. **CVE-2013-4786, IPMI (172.17.0.10:623/UDP):** no está en KEV, pero el EPSS es alto. Permite capturar hashes de administración del BMC; si se rompen, un atacante podría apagar o reiniciar servidores físicos.

El resto de hallazgos (XSS en jQuery/Bootstrap, smuggling en Tomcat, enumeración de usuarios en OpenSSH y configuraciones débiles de Samba) tienen menor impacto o requieren condiciones más específicas, por lo que se recomienda abordarlos en el ciclo de parcheo habitual.

### 4.3 Respuesta al aviso del cliente

**Contexto:** el cliente pregunta por una vulnerabilidad reciente que "están aprovechando los atacantes".

**Lo que se puede sostener con las evidencias:**

- **Inventario actualizado:** están mapeados los 8 equipos con sus puertos y versiones. Si el CVE del aviso corresponde a ActiveMQ, Tomcat AJP, Jenkins, vsftpd o IPMI, sí les afecta y ya está recogido en el informe.
- **Explotación activa:** el informe ya incluye las vulnerabilidades que CISA (KEV) cataloga como explotadas.

**Lo que queda pendiente de comprobar:**

- Si el aviso menciona un producto o versión que no está en el inventario (un router, una impresora u otro software fuera de la red 172.17.0.0/24), en principio no les afecta.
- Si se refiere a una configuración muy específica que no se puede ver sin autenticarse en los sistemas.

**Recomendación:** pedir al cliente el identificador exacto (CVE) o el enlace de la noticia. En menos de una hora se puede contrastar con las versiones de los 8 equipos y confirmar si están expuestos.
