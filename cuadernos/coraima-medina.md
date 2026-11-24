## De donde partimos

Partimos de un correo enviado por la empresa Gadir Logística, S.L:

>Buenos días:
>Le escribo de **Gadir Logística, S.L.**, operador logístico del puerto de Cádiz. Movemos mercancía y tenemos nuestros propios sistemas: el portal donde los clientes consultan sus envíos, la mensajería que conecta con los operadores, un servidor donde el chico de informática «montaba cosas» y la consola con la que se gestionan los servidores de la sala técnica.
>Vamos a **renovar el ciberseguro** y la aseguradora nos pide _«un informe de vulnerabilidades priorizado»_ de nuestra red interna. No sabemos muy bien qué es eso, pero necesitamos entregarlo este trimestre.
>Dos cosas importantes: el informático que lo montó todo ya no está, y nos dejó un inventario en una hoja que no sé si está al día. Y que quede claro: solo queremos que miréis, no que toquéis nada — si algo se cae, paramos de facturar. Además, este trimestre solo podemos permitirnos arreglar unas pocas cosas, así que necesitamos que nos 
>digáis **cuáles primero**.
> Quedo a la espera de ese informe.

#### Las reglas del cliente 

- **Solo identificar.** Descubres, escaneas, enumeras y documentas. **No se explota nada.**
- **Nada de intrusión, en concreto:**
    - no lanzar exploits ni _proof-of-concept_;
    - no usar los scripts de Nmap de las categorías `exploit`, `intrusive`, `dos`, `brute` ni `malware` (el NSE que dispara una puerta trasera **cuenta como explotar**);
    - no usar el `check` ni los módulos de Metasploit que interactúan con la vulnerabilidad;
    - no probar credenciales ni usar logins por defecto, aunque los sospeches;
    - no hacer fuerza bruta de usuarios ni de contraseñas;
    - nada de **escáneres de vulnerabilidades en modo intrusivo**: ni `nuclei` sin filtrar (sus plantillas intrusivas y de _default-login_ entran en la máquina), ni OpenVAS/Nessus lanzando comprobaciones activas que tocan la vulnerabilidad. En esta fase: detección de versión y comprobaciones no intrusivas; nada que «pruebe» el fallo.
- **Con cuidado.** Hay un equipo antiguo y delicado en la red: si lo escaneas a lo bruto, se cae. Escanea con cabeza.
- **Si encuentras algo muy grave**, no lo toques: anótalo y avísalo. Confirmar que una vulnerabilidad es real explotándola es trabajo de la **siguiente** fase, y solo con permiso.

---
# 1. Descubrir y Mapear 
### Identificación de hosts en la red
Lo primero que tenemos que hacer es identificar todos los host activos de la red, para eso utilizamos el siguiente comando: 

``` bash
sudo nmap -sn 172.17.0.0/24 -oN hosts.nmap
```
En la terminal podemos observar lo siguiente: 
``` bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-24 12:14 +0000
Nmap scan report for 172.17.0.3
Host is up (0.000035s latency).
MAC Address: 16:35:84:33:E6:1C (Unknown)
Nmap scan report for 172.17.0.4
Host is up (0.000014s latency).
MAC Address: DE:0A:7D:EA:17:66 (Unknown)
Nmap scan report for 172.17.0.5
Host is up (0.000012s latency).
MAC Address: 12:D1:C2:CD:0C:DB (Unknown)
Nmap scan report for 172.17.0.6
Host is up (0.000014s latency).
MAC Address: AA:12:4D:B8:E1:26 (Unknown)
Nmap scan report for 172.17.0.7
Host is up (0.000014s latency).
MAC Address: 7E:F3:7C:8F:37:8A (Unknown)
Nmap scan report for 172.17.0.8
Host is up (0.000012s latency).
MAC Address: D6:85:CB:94:BD:43 (Unknown)
Nmap scan report for 172.17.0.9
Host is up (0.000010s latency).
MAC Address: BA:09:DD:AF:69:C4 (Unknown)
Nmap scan report for 172.17.0.10
Host is up (0.000023s latency).
MAC Address: 1A:40:04:16:0F:29 (Unknown)
Nmap scan report for 172.17.0.1
Host is up.
#Nmap done: 256 IP addresses (9 hosts up) scanned in 4.00 seconds
```

>Salen `9` equipos activos en la red. Quitando la `.1`, que es la máquina desde la que escaneamos, nos quedan `8` objetivos.

>[!important]
>Cuando lanzamos `nmap -sn`, la forma de descubrir equipos depende de dónde estén:
>
>- **Si están en nuestro mismo segmento de red**, Nmap usa **ARP**. Básicamente manda una pregunta a toda la red: *¿quién tiene la IP x.x.x.x?*. El equipo que contesta está activo, y de paso nos da su IP y su MAC.
>- **Si están en otra red**, recurre a **ICMP Echo Request** (el típico ping).
>
>Ojo, porque que un equipo no conteste al ping no significa que esté apagado: puede tener un firewall que bloquee el ICMP. Por eso, dentro de la red local, ARP es más fiable, ya que normalmente no se filtra.
>
>- `sudo nmap -sn 172.17.0.0/24` → usa **ARP** (comportamiento por defecto).
>![[Pasted image 20261124173948.png]]
>- `sudo nmap --disable-arp-ping -sn 172.17.0.0/24` → fuerza el uso de **ICMP**.
>![[Pasted image 20261124174339.png]]

### Formateo de IPS

El archivo `hosts.nmap` trae mucha información, pero para pasarle varias IPs a `nmap` solo necesitamos las direcciones, separadas por espacios. Para dejarlo limpio, usamos un comando que extrae las IPs y las coloca todas en una sola línea:
``` bash
 tail -n+2 hosts.nmap | grep -oE "([0-g]+\.){3}[0-9]+" | paste -sd' ' | tee hosts_ips.txt
```

``` bash
172.17.0.3 172.17.0.4 172.17.0.5 172.17.0.6 172.17.0.7 172.17.0.8 172.17.0.9 172.17.0.10 172.17.0.1
```

`tail -n+2`: salta la primera línea (la del comentario de nmap, que contiene el rango de red).
- `grep -oE`: con una expresión regular, extrae solo las IPs.
- `paste -sd' '`: las junta en una línea separadas por espacios.
- `tee`: las muestra y las guarda en `hosts_ips.txt`.

Como nuestra propia IP también aparece en el escaneo, después habría que quitarla de la lista a mano, con un editor de texto, por ejemplo.

Para guardar en otro archivo la relación entre IP y MAC de cada equipo (`hosts_datos.txt`), usamos un comando parecido que extrae las IPs y las MACs y las une por parejas con `sed`.
```bash 
tail -n+2 | grep -oE "([0-9]+\.){3}[0-9]+|([A-Z0-9]+\:){5}[A-Z0-9]+" | sed 'N;s/\n/: /' | tee hosts_datos.txt
```

``` bash
172.17.0.3: EA:FB:EF:D2:DD:F1
172.17.0.4: 72:83:A3:FF:A6:D3
172.17.0.5: C2:EB:1A:D6:E7:04
172.17.0.6: 3A:0D:4E:B3:B3:92
172.17.0.7: 3E:8E:8B:DE:39:B7
172.17.0.8: DA:41:B3:D0:CB:9D
172.17.0.9: 06:FE:85:80:3E:A0
172.17.0.10: 12:4F:2B:31:4F:86
```

### Enumeración de puertos global
Al tener las máquinas activas en la red podemos enumerar sus puertos abiertos siguiendo el siguiente comando:

``` bash
sudo nmap -p- -sS -Pn -n -oN puertos_global.nmap -oX puertos_global.xml $(cat ~/Documents/HE/RECON_ACTIVE/informatico_fantasma/hosts_ips.txt)
```

Con este comando generamos dos archivos que recogen todos los puertos TCP abiertos de las máquinas que hemos identificado. Eso sí, solo nos dicen qué puertos están abiertos, sin comprobar qué servicio hay realmente detrás de cada uno. Para poder automatizar ese escaneo de servicios, primero tenemos que volver a darle formato al contenido de `puertos_global.nmap`.

Lo que nos interesa es tener cada IP con sus puertos abiertos en una sola línea, y para eso podemos usar el siguiente comando:

```bash
awk '/Nmap scan report/{if(ip && ports) print ip":"ports; ip=$5; ports=""} /^[0-9]+\/tcp.*open/{split($1,a,"/"); ports=(ports?ports",": "")a[1]} END{if(ip && ports) print ip": "ports}' puertos_global.nmap > puertos_global_ips_ports.txt

172.17.0.3:21,80
172.17.0.4:22,8009,8080
172.17.0.5:80,443,1883,5672,8161,39789,61613,61614,61616
172.17.0.6:22,8080
172.17.0.7:22
172.17.0.8:22,80,139,445
172.17.0.9:22,80
172.17.0.10:80
```

#### Puertos UDP
Ahora, sacamos todos los puertos UDP de todas las máquina de la red. 
``` bash
sudo nmap -sU --top-ports 1000 --max-retries 2 --open -T2 --max-rate 10 -oN scans_udp/puertos_udp_global_1000.nmap -oX scans_udp/puertos_udp_global_1000.xml -iL hosts_ips.txt
```

# 2. Enumerar sin autenticar
Con el mapa de puertos y servicios ya montado, toca ver qué información suelta cada servicio sin necesidad de credenciales. Para cada uno hay que decidir qué técnicas de enumeración tienen sentido y qué datos expone: su configuración, los recursos que ofrece, las cuentas de usuario que deja ver o cualquier otra pista útil para conocer el sistema.

No todos los servicios enseñan lo mismo, así que iremos documentando qué hemos conseguido sacar y cómo, y también las limitaciones que nos hemos encontrado por el camino. En ningún momento se prueban credenciales ni se intenta entrar en nada que pida autenticación.

### 172.17.0.3: vsftpd y Apache
```
Servicios: 21/tcp vsftpd 2.3.4 · 80/tcp Apache httpd 2.4.58 (Ubuntu)

- FTP (21):** el usuario `anonymous` sin contraseña devuelve un error de configuración, lo que indica que el acceso anónimo está habilitado pero mal configurado.
- HTTP (80):** página por defecto de Apache. No se hizo fuzzing de directorios para no salir del scope.
```

```
Vulnerabilidades:
-vsftpd 2.3.4, CVE-2011-2523 (CWE-78, CVSS 9.8): backdoor con RCE. Un usuario terminado en `:)` abre el puerto 6200 con una shell de root.
-Apache 2.4.58, CVE-2023-38709 (CWE-1284, CVSS 7.3): HTTP Response Splitting por falta de validación de datos internos.
```

### 172.17.0.4: Tomcat y AJP

```
Servicios: 22/tcp OpenSSH 7.9p1 · 8009/tcp AJP13 · 8080/tcp Apache Tomcat 9.0.30

- SSH (22): sin vulnerabilidades conocidas en SearchSploit.
- HTTP (8080): página por defecto de Tomcat.
```

```
Vulnerabilidades:

- AJP13, CVE-2020-1938 "Ghostcat" (CWE-285, CVSS 9.8): el conector AJP está expuesto y acepta peticiones sin autenticación, lo que permite suplantar un servidor interno de confianza.
- Tomcat 9.0.30:
    - CVE-2022-34305 (CWE-79, CVSS 6.1): XSS en la aplicación de demostración.
    - CVE-2020-1935 y CVE-2019-17569 (CWE-444, CVSS 4.8): HTTP Request Smuggling.
```

### 172.17.0.5: ActiveMQ y aplicación web

```
Servicios: 80/tcp y 443/tcp Apache 2.4.59 · 1883 MQTT · 5672 AMQP · 8161 Jetty 9.4.39 · 39789 tcpwrapped · 61613 STOMP · 61614 Jetty 9.4.39 · 61616 ActiveMQ OpenWire 5.15.15
- HTTPS (443): certificado autofirmado y caducado (desde 2025-04-17). Sirve la web de un restaurante en desarrollo.
- Dependencias: jQuery 3.4.1 (CVE-2020-11023 y CVE-2020-11022, XSS) y Bootstrap 3.4.1 (CVE-2019-8331, XSS).
- HTTP (8161): pide autenticación básica; por convención es la consola de administración de ActiveMQ.
- HTTP (61614): curl devuelve 200 OK con contenido vacío (Content-Length: 0).
```

```
Vulnerabilidades:
- ActiveMQ OpenWire 5.15.15, CVE-2023-46604 (CWE-502, CVSS 9.8): RCE crítica por deserialización/instanciación arbitraria de clases.
```   

### 172.17.0.6: Jenkins
```
Servicios: 22/tcp OpenSSH 9.2p1 · 8080/tcp Jetty 10.0.18
- HTTP (8080): panel de login de Jenkins, versión 2.441.
```

```
Vulnerabilidades:
- CVE-2024-23897 (CWE-22/CWE-27, CVSS 9.8): lectura arbitraria de archivos mediante la expansión de argumentos `@ruta` en la CLI.
- CVE-2024-23898 (CWE-346, CVSS 8.8): Cross-Site WebSocket Hijacking por falta de validación del origen.
```

### 172.17.0.7: OpenSSH antiguo
```
Servicios: 22/tcp OpenSSH 7.7
```

```
Vulnerabilidades:
- CVE-2019-16905 (CWE-190, CVSS 7.8): desbordamiento de enteros en claves XMSS (experimental, no habilitado por defecto).
- CVE-2018-15473 (CWE-362, CVSS 5.3): enumeración de usuarios mediante respuestas diferenciales.
```

### 172.17.0.8: Samba y Apache
```
Servicios: 22/tcp OpenSSH 9.6p1 · 80/tcp Apache 2.4.58 · 139 y 445/tcp Samba smbd 4
```

```
Samba (445), enumeración anónima:
- `smbclient -L //172.17.0.8 -N` lista los recursos: `myshare` (accesible), `backup24` (denegado), `home` (denegado) e `IPC$`.
- En `myshare` está `access.txt` (956 bytes) con posible información sensible; no se descargó por razones de seguridad por la información sensible.
- Con sesión nula y `rpcclient` se enumeran 5 usuarios: administrador (RID 0x3ec), satriani7 (0x3eb), usuario1 (0x3e8), usuario2 (0x3e9) y usuario3 (0x3ea).
- Política de contraseñas débil: `min_password_length: 5` y sin requisitos de complejidad.
- `netshareenum` expone rutas reales del sistema: `/srv/samba/myshare`, `/srv/samba/backup24`, `/var/www/html` y `/tmp`.
```

**Comandos utilizados:**
``` bash
smbclient -L //172.17.0.8 -N
smbclient //172.17.0.8/myshare -N -c 'ls'
rpcclient -N -U '' 172.17.0.8 -c 'querydispinfo; enumdomusers'
rpcclient -N -U '' 172.17.0.8 -c 'enumdomgroups;getdompwinfo'
rpcclient -N -U '' 172.17.0.8 -c 'netshareenum; netshareenumall'
```

```
**Vulnerabilidades:** no se indican CVE; los hallazgos son de configuración (sesión nula, política de contraseñas débil y exposición de rutas).
```

### 172.17.0.9: aplicación web de prueba
```
Servicios: 22/tcp OpenSSH 9.2p1 · 80/tcp Apache 2.4.68

- HTTP (80):`whatweb` muestra el título "test page". Hay un panel de login (`login.php`), pero el HTML y CSS no filtran información sensible.
```

```
Vulnerabilidades:

- OpenSSH 9.2p1: ninguna.
- Apache 2.4.68: la única vulnerabilidad reportada se corrige en esta versión, por lo que no le afecta.
```

### 172.17.0.10: IPMI expuesto
```
Servicios: 80/tcp SimpleHTTPServer 0.6 (Python 3.11.2) · 623/udp asf-rmcp (IPMI 2.0)

- HTTP (80): laboratorio de pruebas sobre IPMI; muestra información sobre el BMC y la vulnerabilidad CVE-2013-4786.
- IPMI (623/UDP): confirmado con `nmap --script ipmi-version`. Soporta IPMI 2.0 y métodos de autenticación débiles.
``` 

``` bash
sudo nmap -Pn -n -sU --script ipmi-version -p623 172.17.0.10
```

```
Vulnerabilidades:

- CVE-2013-4786 (CWE-255, CVSS 7.5): divulgación de credenciales (hashes HMAC) mediante RAKP Message 2, lo que permite ataques de fuerza bruta offline.
```

# 3. De la versión a la vulnerabilidad y priorización

Una vez claros los servicios y sus versiones, los cruzamos con bases de datos públicas (CVE, CWE y CVSS) para ver qué riesgo suponía cada uno. Con eso pudimos ordenar qué arreglar primero, sabiendo que el cliente solo puede abordar unas pocas cosas este trimestre.
### Tabla de hallazgos CVE

|     Host      |  Puerto   |          Servicio (versión)           |       CVE        |                                                               CWE                                                                | CVSS  |                                      Descripción                                      |
| :-----------: | :-------: | :-----------------------------------: | :--------------: | :------------------------------------------------------------------------------------------------------------------------------: | :---: | :-----------------------------------------------------------------------------------: |
| `172.17.0.3`  |   `21`    |            `vsftpd 2.3.4`             | `CVE-2011-2523`  |                                  ***[CWE-78](https://cwe.mitre.org/data/definitions/78.html)***                                  | `9.8` |                                    ***Backdoor***                                     |
| `172.17.0.3`  |   `80`    |         `Apache httpd 2.4.58`         | `CVE-2023-38709` |                                ***[CWE-1284](https://cwe.mitre.org/data/definitions/1284.html)***                                | `7.3` |                             ***HTTP Response Splitting***                             |
| `172.17.0.4`  |  `8009`   |    `Apache Jserv (Protocol v1.3)`     | `CVE-2020-1938`  |                                 ***[CWE-285](https://cwe.mitre.org/data/definitions/285.html)***                                 | `9.8` |                                    ***Ghostcat***                                     |
| `172.17.0.4`  |  `8080`   |        `Apache Tomcat 9.0.30`         | `CVE-2022-34305` |                                  ***[CWE-79](https://cwe.mitre.org/data/definitions/79.html)***                                  | `6.1` |                                       ***XSS***                                       |
| `172.17.0.4`  |  `8080`   |        `Apache Tomcat 9.0.30`         | `CVE-2020-1935`  |                                 ***[CWE-444](https://cwe.mitre.org/data/definitions/444.html)***                                 | `4.8` |                             ***HTTP Request Smuggling***                              |
| `172.17.0.4`  |  `8080`   |        `Apache Tomcat 9.0.30`         | `CVE-2019-17569` |                                 ***[CWE-444](https://cwe.mitre.org/data/definitions/444.html)***                                 | `4.8` |                             ***HTTP Request Smuggling***                              |
| `172.17.0.5`  |   `443`   |        `HTTPS (jQuery 3.4.1)`         | `CVE-2020-11023` |                                  ***[CWE-79](https://cwe.mitre.org/data/definitions/79.html)***                                  | `6.1` |                                       ***XSS***                                       |
| `172.17.0.5`  |   `443`   |        `HTTPS (jQuery 3.4.1)`         | `CVE-2020-11022` |                                  ***[CWE-79](https://cwe.mitre.org/data/definitions/79.html)***                                  | `6.1` |                                       ***XSS***                                       |
| `172.17.0.5`  |   `443`   |       `HTTPS (Bootstrap 3.4.1)`       | `CVE-2019-8331`  |                                  ***[CWE-79](https://cwe.mitre.org/data/definitions/79.html)***                                  | `6.1` |                                       ***XSS***                                       |
| `172.17.0.5`  |  `61616`  | `ActiveMQ OpenWire transport 5.15.15` | `CVE-2023-46604` |                                 ***[CWE-502](https://cwe.mitre.org/data/definitions/502.html)***                                 | `9.8` |                                       ***RCE***                                       |
| `172.17.0.6`  |  `8080`   |    `Jetty 10.0.18 (Jenkins 2.441)`    | `CVE-2024-23897` | ***[CWE-22](https://cwe.mitre.org/data/definitions/22.html)***<br>***[CWE-27](https://cwe.mitre.org/data/definitions/27.html)*** | `9.8` |                               ***Arbitrary File Read***                               |
| `172.17.0.6`  |  `8080`   |    `Jetty 10.0.18 (Jenkins 2.441)`    | `CVE-2024-23898` |                                 ***[CWE-346](https://cwe.mitre.org/data/definitions/346.html)***                                 | `8.8` |                          ***Error de Validación de Origen***                          |
| `172.17.0.7`  |   `22`    |             `OpenSSH 7.7`             | `CVE-2019-16905` |                                 ***[CWE-190](https://cwe.mitre.org/data/definitions/190.html)***                                 | `7.8` |                            ***Desbordamiento de Enteros***                            |
| `172.17.0.7`  |   `22`    |             `OpenSSH 7.7`             | `CVE-2018-15473` |                                 ***[CWE-362](https://cwe.mitre.org/data/definitions/362.html)***                                 | `5.3` |                                ***User Enumeration***                                 |
| `172.17.0.10` | `623/UDP` |              `IPMI 2.0`               | `CVE-2013-4786`  |                                 ***[CWE-255](https://cwe.mitre.org/data/definitions/255.html)***                                 | `7.5` | ***Divulgación de credenciales***<br>***Debilidad en el protocolo de autenticación*** |

### Tabla de hallazgos configuraciones
|     Host     | Puerto | Servicio (versión) |                                                                                                                                     CWE                                                                                                                                      |                    Descripción                     |
| :----------: | :----: | :----------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------: |
| `172.17.0.3` |  `21`  |   `vsftpd 2.3.4`   |                                                                                                       ***[CWE-287](https://cwe.mitre.org/data/definitions/287.html)***                                                                                                       |   Posible autenticación pública con `anonymous`    |
| `172.17.0.8` | `445`  |   `Samba smbd 4`   | ***[CWE-200](https://cwe.mitre.org/data/definitions/200.html)***<br>***[CWE-306](https://cwe.mitre.org/data/definitions/306.html)***<br>***[CWE-287](https://cwe.mitre.org/data/definitions/287.html)***<br>***[CWE-284](https://cwe.mitre.org/data/definitions/284.html)*** | **Fuga de información a través de sesión anónima** |
| `172.17.0.8` | `445`  |   `Samba smbd 4`   |                                                                                                      ***[CWE-345](https://cwe.mitre.org/data/definitions/3445.html)***                                                                                                       |      **Datos de autenticación insuficiente**       |
| `172.17.0.8` | `445`  |   `Samba smbd 4`   |                                                                                                       ***[CWE-319](https://cwe.mitre.org/data/definitions/319.html)***                                                                                                       | El tráfico viaja sin cifrado a nivel de protocolo. |

### 3.3 Priorización de riesgos
El orden de remediación lo hemos montado con tres criterios: el CVSS, lo fácil que es explotar el fallo y lo que le dolería al negocio (por ejemplo, que se pare la facturación):

1. **ActiveMQ (172.17.0.5):** actualizar cuanto antes. El RCE de CVE-2023-46604 es crítico (9.8) y relativamente fácil de explotar, y con él alguien podría hacerse con el control del servidor de mensajería.
2. **vsftpd (172.17.0.3):** actualizar o retirar ya. La versión 2.3.4 trae una puerta trasera de serie (CVE-2011-2523).
3. **Jenkins (172.17.0.6):** actualizar. CVE-2024-23897 puede dejar al descubierto credenciales y secretos del CI/CD.
4. **AJP (172.17.0.4):** desactivar el puerto 8009 o protegerlo con un secreto para cerrar la puerta a Ghostcat (CVE-2020-1938).
5. **Samba (172.17.0.8):** desactivar las sesiones nulas, forzar la firma SMB (para frenar ataques de relay) y poner una política de contraseñas decente.
6. **IPMI (172.17.0.10):** limitar el puerto 623/UDP a la red de administración para que no se puedan capturar hashes (CVE-2013-4786).
7. **Apache y dependencias web (172.17.0.3, .5 y .8):** parchear CVE-2023-38709, actualizar jQuery y Bootstrap, y quitar del puerto 443 la aplicación que sigue en desarrollo.

# 4. Priorizar e información al cliente

### 4.1 Fichas de vulnerabilidades: CVSS, EPSS y KEV

Para ordenar las vulnerabilidades con criterio profesional no basta con el CVSS: hay que consultar también la probabilidad de explotación (**EPSS**) y el catálogo de vulnerabilidades explotadas de CISA (**KEV**). 
_Fecha de consulta de los datos: **2026-11-23**._

### 4.1 Fichas de los hallazgos más críticos
| CVE            | Host / Servicio                            | CVSS v3.1 (Vector)                                         | Implicación CIA                                                         | EPSS (30d)                   |
| -------------- | ------------------------------------------ | ---------------------------------------------------------- | ----------------------------------------------------------------------- | ---------------------------- |
| CVE-2023-46604 | 172.17.0.5:61616 ActiveMQ OpenWire 5.15.15 | 10.0 (CNA) / 9.8 (NVD) AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:H/A:H | C: Baja, I: Alta, A: Alta. RCE sin autenticación                        | 99.99% (Percentil 100)       |
| CVE-2020-1938  | 172.17.0.4:8009 Apache Jserv (AJP 1.3)     | 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H                    | C: Alta, I: Alta, A: Alta. Ghostcat: lectura de archivos y posible RCE  | 94.47% (Percentil 100)       |
| CVE-2024-23897 | 172.17.0.6:8080 Jenkins 2.441              | 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H                    | C: Alta, I: Alta, A: Alta. Lectura arbitraria de archivos vía CLI       | ~100% (Percentil >99.9)      |
| CVE-2011-2523  | 172.17.0.3:21 vsftpd 2.3.4                 | 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H                    | C: Alta, I: Alta, A: Alta. Backdoor con shell de root en el puerto 6200 | 96.0% (Percentil 99)         |
| CVE-2013-4786  | 172.17.0.10:623/UDP IPMI 2.0               | 7.5 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N                    | C: Alta, I: Ninguna, A: Ninguna. Captura de hashes HMAC                 | 78.6% - 98.0% (según fuente) |

## 4.2 Orden de prioridad justificado

Hemos cruzado los datos de explotación activa (KEV) y la probabilidad de ataque (EPSS) con lo que supondría para el día a día de Gadir Logística:

1. **CVE-2023-46604, ActiveMQ (172.17.0.5):** es lo más grave que hemos encontrado. Está en KEV, tiene un EPSS cercano al 100 % y se ha usado en ataques de ransomware. Un RCE en la mensajería interna podría parar la operativa, permitir manipular datos o servir de puerta de entrada al resto de la red.
2. **CVE-2020-1938 Ghostcat, Tomcat (172.17.0.4:8009):** está en KEV desde 2022 y tiene un EPSS del 94 %. El conector AJP está expuesto y no pide autenticación. Si ese servidor aloja el portal de clientes, el golpe a la operativa y a la reputación sería directo.
3. **CVE-2024-23897, Jenkins (172.17.0.6):** está en KEV, con uso en malware confirmado y el EPSS al máximo. Podría dejar al descubierto credenciales de CI/CD (claves de API, contraseñas de repositorios) y comprometer toda la cadena de despliegue.
4. **CVE-2011-2523, vsftpd (172.17.0.3:21):** tiene un CVSS de 9.8 y un EPSS alto (96 %), pero no está en KEV, así que no hay evidencia de explotación masiva ahora mismo. Aun así, es una puerta trasera de serie: cualquiera con acceso al FTP puede hacerse con el control total del equipo sin esfuerzo.
5. **CVE-2013-4786, IPMI (172.17.0.10:623/UDP):** no está en KEV, pero su EPSS es alto. Permite capturar hashes de administración del BMC y, si se llegan a descifrar, un atacante podría apagar o reiniciar servidores físicos.

El resto de hallazgos (XSS en jQuery y Bootstrap, smuggling en Tomcat, enumeración de usuarios en OpenSSH y las configuraciones débiles de Samba) tienen menos impacto o necesitan condiciones más concretas para aprovecharse. Por eso recomendamos tratarlos dentro del ciclo de parcheo habitual.

## 4.3 Respuesta al aviso del cliente

**Contexto:** el cliente nos pregunta por una vulnerabilidad reciente que, según le han dicho, los atacantes están aprovechando.

**Lo que podemos asegurar con las evidencias:**

- **Inventario al día:** tenemos mapeados los 8 equipos con sus puertos y versiones. Si el CVE del aviso es de ActiveMQ, Tomcat AJP, Jenkins, vsftpd o IPMI, sí les afecta y ya está recogido en el informe.
- **Explotación activa:** el informe ya incluye las vulnerabilidades que CISA cataloga como explotadas (KEV).

**Lo que queda pendiente de comprobar:**

- Si el aviso habla de un producto o una versión que no aparece en nuestro inventario (un router, una impresora u otro software fuera de la red 172.17.0.0/24), en principio no les afecta.
- Si se refiere a una configuración muy específica, no podemos verla sin autenticarnos en los sistemas, así que no se puede confirmar con lo que tenemos.

**Recomendación:** pedirle al cliente el identificador exacto (el CVE) o el enlace de la noticia. En menos de una hora podemos contrastarlo con las versiones de los 8 equipos y confirmarle si están expuestos o no.