# Comparativa agente vs agente

Nomenclatura: **A** = `Carlos`, **B** = `Coraima`, **C** = `Esteban`, **D** = `Jose`.

> **Coste y tiempo:** ninguno de los ficheros contiene tokens, número de llamadas, duración ni coste en dinero, así que no es posible compararlos con datos reales. El único indicador indirecto es la numeración de evidencias, que refleja cuántas consultas hizo cada agente.

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
