# Comparativa agente vs agente

Nomenclatura: **A** = `Carlos`, **B** = `Coraima`, **C** = `Esteban`, **D** = `Jose`.

| Integrante | Informe | Duración | Llamadas | Tokens entrada | Tokens salida | Total  |
| ---------- | ------- | -------- | -------- | -------------- | ------------- | ------ |
| Carlos     | A       | 464 s    | 30       | 35.619         | 11.181        | 46.800 |
| Coraima    | B       | 810 s    | 36       | 68.205         | 9.375         | 77.580 |
| José       | D       | 414 s    | 29       | 32.812         | 10.902        | 43.714 |
| Esteban    | C       | 949 s    | 31       | 61.043         | 13.679        | 74.722 |

---

## 1. Qué encontró cada agente

Los cuatro informes coinciden en lo básico: 9 equipos, los mismos servicios y CVE-2023-46604 (ActiveMQ) como prioridad nº 1. Las diferencias están en el recuento, en qué CVE entran en el top 10 y en la calidad interna.

| Criterio                           | Carlos | Coraima | Esteban | Jose           |
| ---------------------------------- | ------ | ------- | ------- | -------------- |
| Servicios declarados en el resumen | 23     | 22      | 25      | 20             |
| Filas reales en la tabla           | 25     | 25      | 25      | 19 (+6 aparte) |
| ¿Cuadra el recuento?               | No     | No      | Sí      | No             |
| Última evidencia citada            | E30    | E31     | E31     | E28            |
| CVE en el top 10                   | 10     | 10      | 10      | 10             |

### Composición del top 10

- **Comunes a los 4:** CVE-2023-46604, CVE-2024-38475, CVE-2025-24813, CVE-2017-7494, CVE-2020-1938, CVE-2026-34197 y CVE-2026-123158.
- **A y C:** idénticos en conjunto y en orden. Añaden CVE-2023-44487 (Tomcat), CVE-2020-1472 (Samba, Zerologon) y CVE-2024-6387 (OpenSSH, regreSSHion).
- **B:** no incluye CVE-2023-44487 ni CVE-2024-6387. Añade CVE-2011-2523 (backdoor de vsftpd) y CVE-2023-38408 (ssh-agent, EPSS 0,797).
- **D:** no incluye CVE-2020-1472 (Zerologon). Sí incluye CVE-2011-2523.

---

## 2. Por qué difieren

- **No determinismo del agente.** Cada agente eligió consultas distintas, lo que se nota en que la última evidencia va de E28 a E31. D hizo menos consultas y por eso no encontró Zerologon.
- **Cada uno hizo su propio escaneo (o el laboratorio se reinició).** El puerto efímero `tcpwrapped` de 172.17.0.5 cambia en cada informe: 40761, 40833, 36395 y 40763. Indica ejecuciones de nmap separadas, así que ni siquiera partían de la misma foto.
- **Criterio de "servicio".** D cuenta solo los servicios con versión, de ahí su "20". C cuenta todos. A y B tienen 25 filas pero escriben 23 y 22, lo que parece un error de recuento del LLM sin verificar.
- **Política de agrupación del top 10.** B consolida un mismo CVE en varios equipos, lo que libera huecos para otros CVE. A, C y D listan un CVE por equipo o servicio principal y mencionan los demás en el texto.
- **Método de búsqueda.** Los productos con versión se buscan por CPE (más fiable); los que no, por palabras clave (más ruido). Eso explica que CVE como los de vsftpd o SimpleHTTPServer aparezcan solo en algunos informes.

---

## 3. Ventajas e inconvenientes

### A (`Carlos`)

- ✅ Criterios de ordenación explicados uno a uno, fecha de consulta visible y aviso de CVE duplicadas por servicio.
- ❌ Recuento erróneo (23 frente a 25 filas) y la justificación de la fila 4 ("segundo lugar en EPSS entre las KEV") es incorrecta.
- ❌ Omite Apache 2.4.68 de la lista de backports.
- ❌ Atribuye CVE-2026-123158 solo a .1, aunque lo menciona para otros equipos.

### B (`Coraima`)

- ✅ Es el único que consolida por CVE (6 equipos con OpenSSH, 4 con Apache) y detecta vsftpd y CVE-2023-38408.
- ❌ El orden contradice su propia regla: CVE-2024-38475 (KEV, EPSS 1,000) queda en la fila 8, por debajo de otros con menor EPSS.
- ❌ Omite CVE-2023-44487 (EPSS 1,000, KEV) del top 10.
- ❌ Incoherencias internas: dice "5 equipos" y luego "6", e incluye .7 (OpenSSH 7.7, sin sufijo de distribución) entre los "con prefijo Debian/Ubuntu".
- ❌ Recuento erróneo (22 frente a 25 filas).

### C (`Esteban`)

- ✅ Es el más consistente: el recuento cuadra, el orden respeta KEV → EPSS → CVSS y los empates están bien resueltos.
- ✅ Incluye ambos CVE de Samba, con el aviso de que no encontrar CVE no demuestra que el servicio sea seguro.
- ❌ Repite Jetty en dos viñetas y no consolida CVE entre equipos.
- ❌ No incluye el backdoor de vsftpd, que es un riesgo evidente.

### D (`Jose`)

- ✅ Es el más compacto, con una justificación del orden clara y una sección "A verificar" bien organizada.
- ✅ Probablemente el más barato y rápido, por tener menos evidencias (E28). Es una inferencia, no un dato.
- ❌ Recuento incoherente: dice 20 pero hay 19 filas más 6 "omitidas". Además llama "sin versión" a puertos que sí tienen versión (61614 Jetty, 139 Samba).
- ❌ Pierde Zerologon (CVE-2020-1472), que está en KEV con ransomware.

### Valoración global

**C** es el más fiable. **D** es el más eficiente. **A** y **B** aportan ideas útiles (consolidar por CVE, vsftpd), pero son los que más errores de coherencia tiene.

---

## 4. Mejoras propuestas

1. **Que el código haga lo mecánico y el LLM solo la narrativa.** Recuentos, ordenación por KEV/EPSS/CVSS, agrupación por CVE y tablas deben salir de un script. Así desaparecen los errores 23/22/20 frente a 25 y los órdenes incoherentes.
2. **Validador automático al final.** Que compruebe que el número del resumen coincide con las filas, que el orden respeta la regla declarada y que todo ID de evidencia existe.
3. **Consolidar por CVE con la lista de equipos afectados** (idea de B) y reservar los huecos del top 10 para CVE distintos. Así entrarían vsftpd, Zerologon y regreSSHion a la vez.
4. **Procedimiento común de consulta**, para que todos los agentes busquen los mismos productos: cada servicio con versión por CPE y, si falla, por palabras clave marcado como "ruido".
5. **Resolver los backports consultando los trackers de Debian y Ubuntu** en lugar de dejarlo como "a verificar". Reduciría mucho los falsos positivos de OpenSSH y Apache.
6. **Verificación activa autorizada** (scripts NSE no destructivos o comprobaciones de versión por banner) para pasar de "potencial (solo versión)" a "confirmado".
7. **Revisar CVE-2026-123158.** Es el único con un identificador atípico y aparece como KEV desde 2026-11-09, posterior al conocimiento fiable del modelo. Conviene comprobarlo directamente contra NVD y KEV.
8. **Instrumentar coste y tiempo.** Registrar tokens, llamadas a herramientas y duración de cada ejecución, y repetir cada agente 3 veces para medir su variabilidad.


# Comparativa: agentes vs. equipo

**Leyenda:** ✅ = coincide con el informe · ⚠️ = parcialmente correcto o con matices · ❌ = incorrecto / no detectado / falso positivo · ➖ = no aplica (no lo cubre o no aplica a ese agente)

---

## 1. Equipos vivos (descubrimiento)

| Equipo | Consolidado / inventario.md | `informe (1)` | `informe (2)` | `informe (3)` | `informe (4)` | Coincide |
|---|---|---|---|---|---|---|
| 172.17.0.1 | ✅ activo (analista) | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.3 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.4 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.5 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.6 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.7 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.8 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.9 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 172.17.0.10 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| Total | 9 (8 objetivos) | ✅ 9 | ✅ 9 | ✅ 9 | ✅ 9 | Sí |

**Observaciones:** los cuatro agentes coinciden con el equipo en el descubrimiento (9 hosts, .1 = analista). Ningún agente reproduce el contraste metodológico ARP/ICMP con `tcpdump` ni `--disable-arp-ping` que documentan los apuntes del equipo; tampoco mencionan la retirada de la IP propia de `hosts_ips.txt` salvo de forma implícita. `informe (2)` acierta al decir "22 servicios" frente a "20" de (4) y "23/25" de (1)/(3), pero esa diferencia es de conteo de puertos, no de equipos.

---

## 2. Servicios y versiones por equipo

### 2.1 172.17.0.3

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 21/tcp vsftpd 2.3.4 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 80/tcp Apache httpd 2.4.58 (Ubuntu) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| FTP anónimo habilitado pero mal configurado | ✅ (CWE-287) | ✅ | ✅ | ✅ | ✅ | Sí |

### 2.2 172.17.0.4

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 22/tcp OpenSSH 7.9p1 Debian 10+deb10u4 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 8009/tcp Apache Jserv (AJP 1.3) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 8080/tcp Apache Tomcat 9.0.30 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |

### 2.3 172.17.0.5

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 80/tcp Apache httpd 2.4.59 (Debian) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 443/tcp Apache httpd 2.4.59 (Debian) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 1883/tcp mqtt (sin versión) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 5672/tcp amqp (sin versión) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 8161/tcp Jetty 9.4.39.v20210325 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 61613/tcp stomp / Apache ActiveMQ (sin versión) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 61614/tcp Jetty 9.4.39.v20210325 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 61616/tcp ActiveMQ OpenWire 5.15.15 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| tcpwrapped (40761/40833/36395/39789) | ✅ (efímero, varía) | ✅ 40761 | ✅ 36395 | ✅ 40833 | ✅ 40763 | ⚠️ todos lo ven, pero **discrepan en el número** |
| Certificado TLS autofirmado y caducado (2025-04-17) | ✅ (Ficha 11) | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente lo recoge** |
| Consola 8161 con Basic Auth (por convención ActiveMQ) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno lo explicita |
| 61614 responde 200 OK / Content-Length: 0 | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno lo recoge |

### 2.4 172.17.0.6

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 22/tcp OpenSSH 9.2p1 Debian 2+deb12u2 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 8080/tcp Jetty 10.0.18 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| Jenkins 2.441 identificado tras el login | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente llega a identificar Jenkins 2.441** |

> **Hallazgo crítico:** los cuatro agentes ven "Jetty 10.0.18" y se quedan ahí; **no identifican Jenkins 2.441**, por lo que se pierden CVE-2024-23897 y CVE-2024-23898 (Ficha 07, en KEV). El equipo sí lo identificó inspeccionando el panel de login.

### 2.5 172.17.0.7

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 22/tcp OpenSSH 7.7 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |

### 2.6 172.17.0.8

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 22/tcp OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 80/tcp Apache httpd 2.4.58 (Ubuntu) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 139/tcp Samba smbd 4 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 445/tcp Samba smbd 4 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| Enumeración anónima: 5 usuarios (administrador, satriani7, usuario1-3) | ✅ (Ficha 10) | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente enumera usuarios SMB** |
| Política de contraseñas débil (min 5, sin complejidad) | ✅ (Ficha 10) | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| Rutas reales expuestas (/srv/samba/myshare, /var/www/html, /tmp) | ✅ (Ficha 10) | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| SMB signing no obligatorio / sin cifrado SMBv3 | ✅ (Ficha 10) | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| `myshare` accesible con `access.txt` (no descargado) | ✅ (Ficha 10) | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |

> **Hallazgo crítico:** los cuatro agentes detectan los puertos Samba pero **no realizan la enumeración anónima** (smbclient/rpcclient) que el equipo sí hizo. Se pierden las 5 cuentas, la política débil y la exposición de `/var/www/html`.

### 2.7 172.17.0.9

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 22/tcp OpenSSH 9.2p1 Debian 2+deb12u10 | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 80/tcp Apache httpd 2.4.68 (Debian) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| `login.php` / título "test page" (whatweb) | ✅ | ➖ | ➖ | ➖ | ➖ | ⚠️ ninguno lo recoge explícitamente |

### 2.8 172.17.0.10

| Puerto / servicio | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| 80/tcp SimpleHTTPServer 0.6 (Python 3.11.2) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí |
| 623/udp asf-rmcp · IPMI 2.0 | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente lo detecta** (solo TCP, no UDP) |
| Métodos de auth débiles (password, md5, md2, null) + IPMI 1.5 | ✅ (Ficha 09) | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |

> **Hallazgo crítico:** los cuatro agentes **omiten el escaneo UDP**, por lo que no ven el puerto 623/UDP ni IPMI 2.0. El equipo sí lo detectó con `nmap -sU --script ipmi-version`.

---

## 3. Cuentas de usuario descubiertas

| Cuenta / dato | Consolidado / inventario.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| `administrador` (RID 0x3ec) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ no detectada |
| `satriani7` (RID 0x3eb) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ no detectada |
| `usuario1` (0x3e8), `usuario2` (0x3e9), `usuario3` (0x3ea) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ no detectadas |
| `min_password_length: 5`, sin complejidad | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ no detectado |
| Dominio `ESEEMEB.DL`, servidor standalone (NULL SID) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ no detectado |
| Usuario `anonymous` FTP habilitado (mal config.) | ✅ | ✅ | ✅ | ✅ | ✅ | Sí (los 4 lo intuyen por el error) |

**Ningún agente enumera cuentas de usuario** (ni SMB ni RPC). El equipo sí las obtuvo con sesión nula.

---

## 4. Vulnerabilidades (CVE) — tabla por tipo de hallazgo

| # | CVE / hallazgo | Activo · servicio | Consolidado / registro-vulns.md | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|---|---|
| 1 | **CVE-2023-46604** (RCE ActiveMQ) | .5 · 61616 | ✅ Ficha 05, prioridad 1 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** |
| 2 | **CVE-2024-38475** (Apache httpd) | .3/.5/.8 · 80/443 | ✅ prioridad 2 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — pero **no está en registro-vulns.md** (el registro usa CVE-2023-38709) ⚠️ |
| 3 | **CVE-2023-44487** (Tomcat) | .4 · 8080 | ✅ prioridad 3 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 4 | **CVE-2025-24813** (Tomcat) | .4 · 8080 | ✅ prioridad 4 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 5 | **CVE-2017-7494** (Samba) | .8 · 139/445 | ✅ prioridad 5 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 6 | **CVE-2020-1472** (Samba) | .8 · 139/445 | ✅ prioridad 6 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 7 | **CVE-2020-1938** (Ghostcat) | .4 · 8009 | ✅ Ficha 03 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** |
| 8 | **CVE-2026-34197** (ActiveMQ) | .5 · 61616 | ✅ prioridad 8 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 9 | **CVE-2026-123158** (OpenSSH, KEV 2026-11-09) | .1/.4/.6/.7/.8/.9 | ✅ prioridad 9 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 10 | **CVE-2024-6387** (OpenSSH) | .6 · 22 | ✅ prioridad 10 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** — no está en registro-vulns.md ⚠️ |
| 11 | **CVE-2011-2523** (vsftpd backdoor) | .3 · 21 | ✅ Ficha 01 | ✅ | ✅ | ✅ | ✅ | **Sí (los 4)** |
| 12 | **CVE-2023-38408** (OpenSSH) | .4/.7 · 22 | ✅ prioridad 12 | ✅ | ➖ | ➖ | ➖ | ⚠️ solo (1); no está en registro-vulns.md |
| 13 | **CVE-2023-38709** (Apache httpd) | .3/.8 · 80 | ✅ Ficha 02 | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente lo recoge** |
| 14 | **CVE-2024-23897** (Jenkins, KEV) | .6 · 8080 | ✅ Ficha 07 | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente lo recoge** (no identifican Jenkins) |
| 15 | **CVE-2024-23898** (Jenkins) | .6 · 8080 | ✅ Ficha 07 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 16 | **CVE-2019-16905** (OpenSSH XMSS) | .7 · 22 | ✅ Ficha 08 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 17 | **CVE-2018-15473** (OpenSSH user enum) | .7 · 22 | ✅ Ficha 08 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 18 | **CVE-2013-4786** (IPMI RAKP) | .10 · 623/udp | ✅ Ficha 09 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno (no escanean UDP) |
| 19 | **CVE-2022-34305** (Tomcat XSS) | .4 · 8080 | ✅ Ficha 04 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 20 | **CVE-2020-1935 / CVE-2019-17569** (Tomcat smuggling) | .4 · 8080 | ✅ Ficha 04 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 21 | **CVE-2020-11022 / -11023** (jQuery) | .5 · 443 | ✅ Ficha 06 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 22 | **CVE-2019-8331** (Bootstrap) | .5 · 443 | ✅ Ficha 06 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 23 | **Config Samba** (sesión nula, signing, cifrado, política) | .8 · 139/445 | ✅ Ficha 10 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 24 | **Certificado TLS autofirmado/caducado + app en desarrollo** | .5 · 443 | ✅ Ficha 11 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 25 | **Servicios de mensajería/consola ActiveMQ expuestos** | .5 · 1883/5672/8161/61613/61614/39789 | ✅ Ficha 12 | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 26 | **Posible FTP anónimo** | .3 · 21 | ✅ Ficha 10 | ✅ | ✅ | ✅ | ✅ | Sí (los 4) |

**Resumen CVE:** los cuatro agentes aciertan el "top 10" KEV/EPSS del informe consolidado (filas 1–11), pero **ninguno cubre** las fichas 02, 04 (parcial), 06, 07, 08, 09, 10, 11 y 12 del `registro-vulns.md`. El registro del equipo incluye 12 fichas; los agentes cubren como máximo 3–4 de ellas (Ficha 01, 03, 05 y parcialmente 10 con FTP anónimo).

**Discrepancia interna del equipo (a resolver):** el informe consolidado prioriza **CVE-2024-38475**, **CVE-2023-44487**, **CVE-2025-24813**, **CVE-2017-7494**, **CVE-2020-1472**, **CVE-2026-34197**, **CVE-2026-123158** y **CVE-2024-6387**, que **no aparecen en `registro-vulns.md`** (que usa CVE-2023-38709, CVE-2022-34305, CVE-2020-1935/17569, CVE-2019-16905, CVE-2018-15473, CVE-2013-4786). Hay que unificar criterio: o el registro incorpora esas CVE, o el consolidado deja de priorizarlas.

---

## 5. Priorización (orden de remediación)

| Aspecto | Equipo (consolidado) | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| Criterio | KEV → EPSS → CVSS → impacto negocio | KEV → EPSS → CVSS | KEV → EPSS → CVSS | KEV → EPSS → CVSS | KEV → EPSS → CVSS | ✅ mismo criterio |
| #1 | ActiveMQ CVE-2023-46604 | ✅ | ✅ | ✅ | ✅ | Sí |
| #2 | OpenSSH CVE-2026-123158 (6 equipos) / Apache CVE-2024-38475 según versión | ⚠️ (1) pone Apache #2 y OpenSSH #9; (2) pone Tomcat #2; (3) pone Apache #2; (4) pone Apache #2 | ⚠️ | ⚠️ | ⚠️ | **No hay consenso**: solo el consolidado pone OpenSSH en #2 |
| #3 | Jenkins CVE-2024-23897 | ➖ (no lo ven) | ➖ | ➖ | ➖ | ❌ **el equipo lo prioriza #3 y ningún agente lo detecta** |
| #4 | Tomcat Ghostcat CVE-2020-1938 | ✅ | ✅ | ✅ | ✅ | Sí (posición varía) |
| #5 | vsftpd CVE-2011-2523 | ✅ | ✅ | ✅ | ✅ | Sí |
| #6 | IPMI CVE-2013-4786 | ➖ | ➖ | ➖ | ➖ | ❌ no detectado |
| #7 | Samba config | ✅ | ✅ | ✅ | ✅ | Sí (pero sin la enumeración real) |
| Plan trimestral (bloque 1) | 5 actuaciones: ActiveMQ, OpenSSH, Jenkins, AJP, vsftpd | ⚠️ propone 5 pero sin OpenSSH ni Jenkins | ⚠️ | ⚠️ | ⚠️ | **Parcial**: los agentes no incluyen Jenkins ni OpenSSH como bloque 1 |

---

## 6. Inventario del cliente (discrepancias)

| Comprobación | `inventario.md` | (1) | (2) | (3) | (4) | Coincide |
|---|---|---|---|---|---|---|
| Inventario del cliente no fiable / desactualizado | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ **ningún agente evalúa el CSV del cliente** |
| 6 de 8 equipos no inventariados | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 172.17.0.9 era `TPV-RECEPCION` (Windows 7/RDP) → ahora Debian 12 | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 172.17.0.10 era `SRV-PORTAL01` (WS2008/IIS) → ahora Python + IPMI | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 8 activos legados que no responden (.2, .11, .20, .30, .50, .100, .254, 172.17.1.6) | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |
| 172.17.1.6 fuera de subred | ✅ | ➖ | ➖ | ➖ | ➖ | ❌ ninguno |

**Ningún agente corrige el inventario del cliente.** Es una omisión total frente al entregable `inventario.md`.

---

## 7. Resumen de coincidencias por agente

| Bloque | (1) | (2) | (3) | (4) | Comentario |
|---|---|---|---|---|---|
| Descubrimiento de hosts (9) | ✅ | ✅ | ✅ | ✅ | Todos correctos |
| Servicios TCP y versiones | ✅ | ✅ | ✅ | ✅ | Todos correctos en lo que ven |
| Puerto tcpwrapped .5 | ⚠️ 40761 | ⚠️ 36395 | ⚠️ 40833 | ⚠️ 40763 | Discrepan en el número |
| Jenkins 2.441 | ❌ | ❌ | ❌ | ❌ | No identificado |
| IPMI 623/UDP | ❌ | ❌ | ❌ | ❌ | No escaneado |
| Enumeración SMB (usuarios/política/rutas) | ❌ | ❌ | ❌ | ❌ | No realizada |
| Certificado TLS .5:443 | ❌ | ❌ | ❌ | ❌ | No recogido |
| Servicios mensajería/consola ActiveMQ | ❌ | ❌ | ❌ | ❌ | No recogido |
| Top 10 CVE KEV/EPSS | ✅ | ✅ | ✅ | ✅ | Coinciden con el consolidado |
| Fichas 02, 04, 06, 07, 08, 09, 11, 12 | ❌ | ❌ | ❌ | ❌ | No cubiertas |
| Priorización general | ⚠️ | ⚠️ | ⚠️ | ⚠️ | Mismo criterio, distinto orden; sin Jenkins/OpenSSH/IPMI |
| Corrección del inventario del cliente | ❌ | ❌ | ❌ | ❌ | No evaluado |

**Conclusión de la comparativa:** los agentes son **consistentes entre sí y con el consolidado en el descubrimiento TCP, versiones y top-10 CVE KEV/EPSS**, pero **omiten sistemáticamente** cuatro áreas que el equipo sí cubrió: (a) identificación de Jenkins 2.441 sobre Jetty 10.0.18, (b) escaneo UDP e IPMI 2.0, (c) enumeración anónima SMB/RPC (cuentas, política, rutas, signing), y (d) análisis de configuración (TLS caducado, servicios de mensajería expuestos, corrección del inventario del cliente). Además, hay **discrepancia interna del equipo** entre el consolidado y `registro-vulns.md` en el conjunto de CVE priorizadas que conviene resolver antes de entregar.

# Los fallos de los agentes

## Cómo se ha hecho

Se ha contrastado el informe final de cada traza (evento `fin`) con lo que devolvieron las herramientas (`nmap_servicios`, `cve_lookup`) en esa misma traza. Los tiempos son segundos desde el evento `inicio`.

La correspondencia entre trazas e informes sale de los puertos `tcpwrapped`, la numeración de evidencias y las fechas de los ficheros:

- **Carlos** = `informe.md` (A), `tcpwrapped` 40761, última evidencia E30.
- **Coraima** = `informe (1).md` (B), `tcpwrapped` 36395, última evidencia E31.
- **Esteban** = `informe (2).md` (C), `tcpwrapped` 40833, última evidencia E31.
- **José** = `informe (3).md` (D), `tcpwrapped` 40763, última evidencia E28.

**Resultado general:** los 4 informes citan solo CVE que aparecen en las salidas de las herramientas. No hay ningún identificador de CVE inventado. Los fallos están en lo que se omite, en cómo se leen los datos y en lo que se afirma sin comprobar.

---

## 1. Omisión (algo que estaba y no lo vio)

- **Jose — Zerologon (CVE-2020-1472).** En 172.17.0.8:445 (t≈282 s, E25) la herramienta devolvió este CVE como segundo resultado, con KEV desde 2021-11-03, ransomware «sí» y EPSS 0.994. El informe de José no lo incluye en el top 10 ni lo menciona. Es la omisión más grave de las cuatro trazas, porque el dato estaba delante.
- **Coraima — CVE-2023-44487 y CVE-2024-6387.** En 172.17.0.4:8080 (t≈383 s) la herramienta devolvió CVE-2023-44487 como primer resultado (KEV, EPSS 1.000). En 172.17.0.6:22 (t≈545 s) devolvió CVE-2024-6387 (EPSS 0.995). Ninguno de los dos entra en su top 10, que sí incluye CVE-2023-38408 con EPSS 0.797.
- **Coraima — equipos afectados por CVE-2023-38408.** La herramienta lo devolvió para 172.17.0.4, .6, .7 y .9. El informe dice «172.17.0.4 (y 172.17.0.7)».
- **Los 4 — Ghostcat (CVE-2020-1938) y el puerto AJP 8009.** Los cuatro informes listan Ghostcat para Tomcat 9.0.30 y también tienen 172.17.0.4:8009 (Apache Jserv, AJP 1.3) abierto. Ninguno relaciona ambos datos. Es justo el vector de Ghostcat y subiría la confianza del hallazgo de «solo versión» a «exposición coherente».
- **Carlos y Esteban — vsftpd 2.3.4 (CVE-2011-2523).** Ambos lo vieron (Carlos t≈148 s, E12; Esteban t≈249 s, E11: CVSS 9.8, EPSS 0.962) y lo dejaron fuera del top 10. Por su criterio KEV → EPSS → CVSS queda en el puesto 11, pero es la puerta trasera clásica y merecía al menos una mención aparte. Coraima y José sí lo incluyen.
- **Carlos — no consulta servicios sin versión.** No llamó a `cve_lookup` para 8009, 1883, 5672 ni 61613. Esteban solo consultó 8009 (t≈895 s). José solo consultó 61613. Coraima consultó los cuatro (t≈336 s y t≈747 s) y es la única que dejó constancia de que la herramienta no busca sin CPE o versión clara.
- **Carlos — backports.** La lista de versiones con posible backport omite Apache httpd 2.4.68 (Debian) de 172.17.0.9.

## 2. Error (un dato mal leído o mal calculado)

- **Recuento de servicios.** El `nmap_servicios` suma 25 puertos abiertos (1+2+3+9+2+1+4+2+1). Tres informes declaran otro número:
    - **Carlos:** «23 servicios».
    - **Coraima:** «22 servicios».
    - **Jose:** «20 servicios TCP con versión», pero su tabla tiene 19 filas más 6 aparte.
    - Esteban es el único que acierta (25).
- **Carlos — justificación de la fila 4.** Dice que CVE-2025-24813 (EPSS 0.999) es «segundo lugar en EPSS entre las KEV». Antes hay tres CVE en KEV con EPSS 1.000 (CVE-2023-46604, CVE-2024-38475 y CVE-2023-44487), así que es el cuarto.
- **Coraima — afirmación sobre EPSS.** Dice que CVE-2023-38408 (EPSS 0.797) es el «siguiente mayor EPSS entre los no-KEV». Las salidas de sus propias herramientas incluyen no-KEV con EPSS mayor: CVE-2024-6387 (0.995), CVE-2018-15473 (0.987), CVE-2023-48795 (0.935), CVE-2024-27316 (0.888) y CVE-2020-13935 (0.866).
- **Coraima — orden contradictorio.** Coloca CVE-2024-38475 (KEV, EPSS 1.000) en la fila 8, por debajo de CVE con EPSS 0.955, y lo justifica con «EPSS 1.000 (el más alto), pero CVSS 9.1». Eso contradice el criterio KEV → EPSS → CVSS que ella misma aplica en las demás filas.
- **Coraima — equipos con OpenSSH.** El resumen dice «OpenSSH en 5 equipos» y la justificación dice 6 (la herramienta confirma 6: .1, .4, .6, .7, .8 y .9). Además incluye 172.17.0.7 entre los «con prefijo Debian/Ubuntu», cuando nmap lo detectó como «OpenSSH 7.7» a secas.
- **José — «sin versión detectada».** Mete en esa lista 61614 (Jetty 9.4.39.v20210325) y 139 (Samba smbd 4), que nmap sí devolvió con versión.

## 3. Invención (afirma algo que ninguna herramienta le dio)

- **Carlos — Jetty «reciente/parcheada».** Para Jetty 9.4.39 y 10.0.18 (t≈193, 206 y 246 s, E18, E19 y E22) la herramienta solo dijo que la búsqueda por palabras clave no encontró candidatos y que esa búsqueda es «más ruido». El informe añade que puede deberse a que «la versión es reciente/parcheada». Ningún dato lo respalda, y Jetty 9.4.39 es una compilación de marzo de 2021. Lo más probable es una limitación del método de búsqueda.
- **«No fue posible buscar CVE» sin haberlo intentado.** Tres agentes lo afirman sin llamar a la herramienta para esos puertos:
    - **Carlos:** mqtt, amqp, stomp (61613) y Jserv.
    - **Esteban:** 1883, 5672 y 61613.
    - **José:** 8009, 1883 y 5672.
    - La conclusión coincide con lo que devolvió después la herramienta a Coraima («sin CPE y sin versión clara»), así que el impacto es bajo. Pero formalmente ningún dato de su traza lo respalda.

## 4. Ruido (falsos positivos que coló sin filtrar)

- **Samba «4» sin versión menor (los 4).** La herramienta buscó por `cpe:2.3:a:samba:samba:4`, que casa con cualquier 4.x y devolvió 44 candidatos. Aun así, todos colocan CVE-2017-7494 en las posiciones 3 a 7 del top 10, y Carlos, Coraima y Esteban añaden Zerologon, sin advertir de que la versión es solo «4». Los banners de 172.17.0.8 (OpenSSH 9.6p1 Ubuntu 3ubuntu13.5, Apache 2.4.58 Ubuntu) apuntan a Ubuntu 24.04, cuyo Samba es muy posterior a las correcciones de ambos CVE. Es muy probable que sean falsos positivos y debieron marcarse como tales.
- **Apache httpd 2.4.58 y 2.4.59 de Ubuntu y Debian (los 4).** CVE-2024-38475 aparece en el puesto 2, 3 u 8 según el informe, y en el resumen ejecutivo como riesgo grave. Todos lo califican como «potencial (solo versión)» y avisan del backport en «A verificar». Está mitigado, pero se priorizó igualmente sin resolver el backport.
- **CVE-2026-123158 en 6 equipos (los 4).** Un único CVE de OpenSSH que casa con versiones desde 7.7 hasta 10.3p1 resulta sospechoso: apunta a un rango de versiones demasiado laxo en el CPE. Además, el identificador es atípico. Nadie lo cuestiona. Yo no puedo verificarlo, así que conviene comprobarlo en NVD y KEV.
- **Coraima — CVE-2023-38408 en la fila 10.** Se explota solo con reenvío de `ssh-agent` hacia un equipo malicioso y la dejó por delante de CVE con mucho mayor EPSS.

## 5. Intento fuera de alcance (lo que el `scope-guard` frenó)

**No hay ningún caso en estas cuatro trazas.**

- Ninguna de las 126 llamadas a herramientas (30 de Carlos, 36 de Coraima, 29 de José, 31 de Esteban) tiene `"bloqueado": true`.
- Todos los argumentos están dentro de 172.17.0.0/24 y solo se usan las tres herramientas autorizadas (`nmap_descubrir`, `nmap_servicios`, `cve_lookup`).

**Fallos de herramienta (no del agente):** Coraima recibió `HTTP 429: Too Many Requests` en 172.17.0.3:80 (t≈288 s) y `HTTP 502: Bad Gateway` en 172.17.0.8:80 (t≈591 s). Reintentó ambas y recuperó los datos (t≈538 s y t≈701 s). Su informe no menciona los reintentos.

---

## 6. Resumen por integrante

- **Carlos (A):** el más limpio en datos de herramienta. Fallos: recuento 23 frente a 25, justificación errónea de la fila 4, hipótesis inventada sobre Jetty y varios servicios sin consultar.
- **Coraima (B):** la más completa en cobertura (consultó todos los servicios y reintentó tras los errores 429 y 502) y la que más errores de razonamiento comete: orden incoherente, afirmación falsa sobre EPSS y dos CVE de alto riesgo que omite.
- **Esteban (C):** el más consistente (recuento correcto, orden coherente), pero omite vsftpd y afirma que no se podía buscar sin intentarlo.
- **Jose (D):** el más rápido y barato, pero omite Zerologon teniéndolo en la salida de la herramienta, además de errores de recuento y de clasificación «sin versión».
- **Común a los 4:** tratar Samba «4» como dato fiable, no enlazar Ghostcat con el puerto AJP abierto y no cuestionar CVE-2026-123158.

# Reflexión: ¿nos fiamos de los agentes?

Nomenclatura: **A** = `Carlos`, **B** = `Coraima`, **C** = `Esteban`, **D** = `Jose`.

Esta reflexión se apoya en las tres comparativas anteriores: agente contra agente, agentes contra equipo y fallos de cada traza.

---

## 1. ¿Daríamos el informe de algún agente al cliente?

**No, ninguno tal cual.** Con matices:

|Agente|¿Se entrega tal cual?|Motivo principal|
|---|---|---|
|Carlos (A)|No|Recuento 23 frente a 25 e hipótesis sin base sobre Jetty|
|Coraima (B)|No|Orden incoherente y omite CVE-2023-44487 y CVE-2024-6387|
|Esteban (C)|No|El más consistente, pero omite vsftpd y no consultó servicios sin versión|
|José (D)|No|Omite Zerologon teniéndolo en la salida de la herramienta|

Cuatro razones de fondo:

1. **Todos tienen errores que el cliente notaría o que le harían actuar mal.** Tres de los cuatro declaran un número de servicios distinto de los 25 puertos reales (23, 22 y 20). José deja fuera Zerologon (en KEV y con ransomware) teniéndolo en la salida de la herramienta. Coraima se salta dos CVE de alto riesgo que había visto.
2. **Todo lo que afirman es «potencial (solo versión)».** Ningún agente verificó nada. Además, parte de lo que priorizan probablemente son falsos positivos: Samba «4» sin versión menor en un equipo que parece Ubuntu 24.04, o Apache de Ubuntu/Debian con parches retroportados.
3. **El informe que recibiría el cliente está incompleto.** No incluye Jenkins 2.441, IPMI en 623/UDP, la enumeración SMB, el certificado TLS caducado, la superficie de ActiveMQ ni la corrección del inventario del cliente. Jenkins, por ejemplo, es nuestra prioridad nº 3 y ningún agente lo ve.
4. **Un informe con contradicciones internas resta credibilidad al resto.** Con un recuento erróneo en la primera página, el cliente duda también de las tablas que sí son correctas.

**Lo que sí aprovecharíamos:** las tablas de equipos y servicios (coinciden con las nuestras en los 4 agentes) y las salidas de las herramientas con su evidencia. Como borrador de esa parte, el informe de Esteban es el más utilizable, tras revisión.

---

## 2. Qué tendríamos que revisar siempre

1. **Cifras contra la fuente.** El número de equipos y servicios del resumen se compara con la salida de nmap, no con lo que escribe el agente.
2. **Orden contra la regla declarada.** Si el criterio es KEV → EPSS → CVSS, se comprueba fila a fila. Coraima y Carlos fallan aquí.
3. **Cada CVE contra el fichero de evidencia.** La herramienta solo muestra los 8 primeros de hasta 77 candidatos por servicio (Tomcat 77, Apache 68). El agente decide sobre una fracción de lo que hay.
4. **Versiones imprecisas y backports.** Samba «4», y todo lo que lleve «(Debian)» o «(Ubuntu)», se contrasta con los trackers de la distribución antes de priorizarlo.
5. **«Sin CVE» no significa seguro.** La búsqueda por palabras clave de Jetty 9.4.39 no encontró nada, pero esa versión es de 2021. Y hay que comprobar que la consulta se hizo de verdad, porque tres agentes dicen «no fue posible» sin intentarlo.
6. **Errores de herramienta.** Coraima recibió un 429 y un 502 y los reintentó. Un agente que no reintente deja un hueco sin avisar, y su informe tampoco lo cuenta.
7. **Explicaciones sin respaldo.** Frases del tipo «la versión es reciente/parcheada» no salen de ninguna herramienta y se eliminan o se verifican.
8. **Lo que el agente no puede ver.** Con tres herramientas (descubrir, servicios y CVE) quedan fuera UDP, la identificación de aplicaciones web, SMB, TLS y el inventario. Eso siempre se completa a mano.
9. **Datos volátiles y atípicos.** EPSS y KEV cambian a diario, así que se revisa la fecha de consulta. CVE-2026-123158 aparece en seis equipos con versiones desde 7.7 hasta 10.3p1 y tiene un identificador atípico. Hay que comprobarlo contra NVD y KEV antes de ponerlo en un informe.
10. **Alcance.** Se verifica que ninguna llamada esté bloqueada y que todas caigan en 172.17.0.0/24. En estas trazas hay 126 llamadas y 0 bloqueos, así que no hemos podido ver actuar al `scope-guard`.

---

## 3. En qué nos ganaron los agentes

- **Rapidez y coste.** Cada ejecución tardó entre 414 s y 949 s, con entre 43.714 y 77.580 tokens. Nuestro tiempo equivalente: [completar con las horas del equipo].
- **Exhaustividad mecánica.** Consultaron todos los servicios con versión, sin cansancio ni saltos. Los 4 coinciden con nuestro inventario en hosts y servicios TCP.
- **Enriquecimiento homogéneo.** EPSS, KEV, CVSS, CWE y marca de ransomware para cada hallazgo, con la fecha de consulta. Los 4 aciertan las filas 1 a 11 del top 10 de nuestro consolidado.
- **Trazabilidad.** Cada afirmación lleva su ID de evidencia, y cada informe se puede contrastar con su traza paso a paso.
- **Coherencia con su propia fuente.** Nuestro consolidado prioriza CVE (CVE-2024-38475, CVE-2023-44487, CVE-2025-24813, CVE-2017-7494, CVE-2020-1472…) que no están en nuestro `registro-vulns.md`. Los agentes no tienen esa discrepancia porque todo sale de la misma fuente.
- **Prudencia en el lenguaje.** Etiquetaron todo como «potencial (solo versión)» y separaron lo que había que verificar.
- **Recuperación ante fallos.** Coraima reintentó dos consultas fallidas y no perdió datos.

## 4. En qué les ganamos nosotros

- **Lo que las herramientas no ven.** Identificamos Jenkins 2.441 tras el login de Jetty, el IPMI 2.0 en 623/UDP, las 5 cuentas SMB con la política de contraseñas débil y las rutas expuestas, el certificado TLS caducado y la superficie de mensajería de ActiveMQ.
- **Interpretación y priorización.** Ponemos Jenkins (en KEV) como tercera prioridad, relacionamos Ghostcat con el AJP abierto y tenemos en cuenta el impacto en el negocio. Ningún agente hizo nada de esto.
- **Contexto del cliente.** Detectamos que su inventario no es fiable: 6 de 8 equipos sin inventariar y los equipos .9 y .10 completamente cambiados.
- **Criterio ante el ruido.** Podemos descartar falsos positivos como Samba «4» y los CVE ya parcheados por la distribución. Los agentes los trataron como dato fiable.
- **Informes sin errores de redacción.** Nuestros recuentos, órdenes y justificaciones los revisó una persona. Los de los agentes fallan en recuento (3 de 4), en orden o en justificaciones.

**Matiz honesto:** parte de nuestra ventaja es de herramientas, no de razonamiento. Con `nmap_descubrir`, `nmap_servicios` y `cve_lookup`, el agente no podía ver Jenkins, IPMI ni SMB. Lo que sí es fallo de razonamiento es Zerologon, el orden, el recuento y el tratamiento de Samba. Y nosotros también tenemos una discrepancia interna por resolver entre el consolidado y el registro de vulnerabilidades.

---

## 5. Conclusión

El modelo que defenderíamos es **híbrido**:

- **El agente** hace la primera pasada: descubrimiento, servicios y CVE con su evidencia.
- **Un script** genera lo mecánico: recuentos, ordenación por KEV/EPSS/CVSS, agrupación por CVE y tablas, más un validador que comprueba que cuadran.
- **Una persona** verifica contra la evidencia, completa lo que el agente no ve (UDP, SMB, aplicaciones web, TLS, inventario), decide la prioridad con el negocio y firma.

Para que el agente rinda más, habría que ampliarle las herramientas (UDP, enumeración SMB, identificación web) y repetir cada ejecución varias veces para medir su variabilidad. Mientras no sea así, el informe del agente es un buen borrador y un mal entregable.
