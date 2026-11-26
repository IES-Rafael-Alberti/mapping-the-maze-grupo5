# Informe de enumeración de la red 172.17.0.0/24

## 1. Resumen ejecutivo

Se identificaron 9 equipos activos en la red autorizada, ejecutando un total de 22 servicios TCP. Los riesgos más graves corresponden a un servidor ActiveMQ con vulnerabilidad de ejecución remota de código en el catálogo KEV con historial de ransomware, e instancias de Apache httpd 2.4.58/2.4.59 con un fallo crítico también en KEV. Se recomienda revisar primero estos servicios, junto con la actualización de OpenSSH en 5 equipos, ya que comparten una vulnerabilidad reciente añadida al KEV en noviembre de 2026.

## 2. Equipos y servicios

| Equipo | Puerto | Servicio y versión | Evidencia |
|---|---|---|---|
| 172.17.0.1 | 22/tcp | OpenSSH 10.3p1 Debian 5 | E10 |
| 172.17.0.3 | 21/tcp | vsftpd 2.3.4 | E02 |
| 172.17.0.3 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E02 |
| 172.17.0.4 | 22/tcp | OpenSSH 7.9p1 Debian 10+deb10u4 | E03 |
| 172.17.0.4 | 8009/tcp | Apache Jserv (Protocol v1.3) | E03 |
| 172.17.0.4 | 8080/tcp | Apache Tomcat 9.0.30 | E03 |
| 172.17.0.5 | 80/tcp | Apache httpd 2.4.59 ((Debian)) | E04 |
| 172.17.0.5 | 443/tcp | Apache httpd 2.4.59 ((Debian)) | E04 |
| 172.17.0.5 | 1883/tcp | mqtt (sin versión) | E04 |
| 172.17.0.5 | 5672/tcp | amqp (sin versión) | E04 |
| 172.17.0.5 | 8161/tcp | Jetty 9.4.39.v20210325 | E04 |
| 172.17.0.5 | 36395/tcp | tcpwrapped (sin versión) | E04 |
| 172.17.0.5 | 61613/tcp | Apache ActiveMQ (sin versión) | E04 |
| 172.17.0.5 | 61614/tcp | Jetty 9.4.39.v20210325 | E04 |
| 172.17.0.5 | 61616/tcp | ActiveMQ OpenWire transport 5.15.15 | E04 |
| 172.17.0.6 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u2 | E05 |
| 172.17.0.6 | 8080/tcp | Jetty 10.0.18 | E05 |
| 172.17.0.7 | 22/tcp | OpenSSH 7.7 | E06 |
| 172.17.0.8 | 22/tcp | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | E07 |
| 172.17.0.8 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E07 |
| 172.17.0.8 | 139/tcp | Samba smbd 4 | E07 |
| 172.17.0.8 | 445/tcp | Samba smbd 4 | E07 |
| 172.17.0.9 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u10 | E08 |
| 172.17.0.9 | 80/tcp | Apache httpd 2.4.68 ((Debian)) | E08 |
| 172.17.0.10 | 80/tcp | SimpleHTTPServer 0.6 (Python 3.11.2) | E09 |

## 3. Vulnerabilidades potenciales priorizadas

Fecha de consulta de EPSS y KEV: 2026-11-25 (cambian a diario).

| # | Hallazgo | Equipo y servicio | CVSS (versión) | EPSS | KEV | CWE | Nivel de evidencia | Evidencia |
|---|---|---|---|---|---|---|---|---|
| 1 | CVE-2023-46604 | 172.17.0.5 · 61616/tcp ActiveMQ 5.15.15 | 9.8 (3.1) | 1.000 | desde 2023-11-02 · ransomware | CWE-502 | potencial (solo versión) | E19 |
| 2 | CVE-2025-24813 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.999 | desde 2025-04-01 | CWE-44, CWE-502, CWE-706 | potencial (solo versión) | E14 |
| 3 | CVE-2017-7494 | 172.17.0.8 · 139/tcp Samba smbd 4 | 9.8 (3.1) | 0.994 | desde 2023-03-30 · ransomware | CWE-94 | potencial (solo versión) | E25 |
| 4 | CVE-2020-1472 | 172.17.0.8 · 139/tcp Samba smbd 4 | 5.5 (3.1) | 0.994 | desde 2021-11-03 · ransomware | sin dato | potencial (solo versión) | E25 |
| 5 | CVE-2020-1938 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.993 | desde 2022-03-03 | sin dato | potencial (solo versión) | E14 |
| 6 | CVE-2026-34197 | 172.17.0.5 · 61616/tcp ActiveMQ 5.15.15 | 8.8 (3.1) | 0.983 | desde 2026-04-16 | CWE-20, CWE-78, CWE-94 | potencial (solo versión) | E19 |
| 7 | CVE-2026-123158 | 172.17.0.1 · 22/tcp OpenSSH 10.3p1 (y 5 equipos más) | 8.1 (3.1) | 0.955 | desde 2026-11-09 | CWE-394 | potencial (solo versión) | E31 |
| 8 | CVE-2024-38475 | 172.17.0.3 · 80/tcp Apache httpd 2.4.58 (y 3 equipos más) | 9.1 (3.1) | 1.000 | desde 2025-05-01 | CWE-116 | potencial (solo versión) | E20 |
| 9 | CVE-2011-2523 | 172.17.0.3 · 21/tcp vsftpd 2.3.4 | 9.8 (3.1) | 0.962 | no | CWE-78 | potencial (solo versión) | E11 |
| 10 | CVE-2023-38408 | 172.17.0.4 · 22/tcp OpenSSH 7.9p1 (y 172.17.0.7) | 9.8 (3.1) | 0.797 | no | CWE-428 | potencial (solo versión) | E12 |

**Justificación del orden:**
- **Fila 1:** KEV desde 2023-11-02 con ransomware confirmado; EPSS 1.000 y CVSS 9.8 → máxima prioridad.
- **Fila 2:** KEV desde 2025-04-01; EPSS 0.999 y CVSS 9.8.
- **Fila 3:** KEV desde 2023-03-30 con ransomware confirmado; EPSS 0.994 y CVSS 9.8.
- **Fila 4:** KEV desde 2021-11-03 con ransomware confirmado; EPSS 0.994 (mismo que fila 3, pero CVSS 5.5 → desempate a favor de 9.8 en fila 3).
- **Fila 5:** KEV desde 2022-03-03; EPSS 0.993 y CVSS 9.8.
- **Fila 6:** KEV desde 2026-04-16; EPSS 0.983 y CVSS 8.8.
- **Fila 7:** KEV desde 2026-11-09; EPSS 0.955 y CVSS 8.1. Afecta a 6 equipos simultáneamente.
- **Fila 8:** KEV desde 2025-05-01; EPSS 1.000 (el más alto), pero CVSS 9.1. Afecta a 4 equipos simultáneamente.
- **Fila 9:** No en KEV; EPSS 0.962 y CVSS 9.8, por encima del resto no-KEV.
- **Fila 10:** No en KEV; EPSS 0.797 y CVSS 9.8, siguiente mayor EPSS entre los no-KEV.

## 4. A verificar

- **Versiones con backport de distribución:** Apache httpd 2.4.58 «(Ubuntu)» en 172.17.0.3 y 172.17.0.8; Apache httpd 2.4.59 «(Debian)» en 172.17.0.5; Apache httpd 2.4.68 «(Debian)» en 172.17.0.9; OpenSSH con prefijo «Debian» o «Ubuntu» en 172.17.0.1, .4, .6, .7, .8, .9. Estas distribuciones pueden haber aplicado parches de seguridad sin cambiar el número de versión upstream, por lo que algunos CVE podrían no aplicarse.
- **Búsquedas por palabras clave (más ruido):** vsftpd 2.3.4 (E11), Jetty 9.4.39.v20210325 (E17, E18), Jetty 10.0.18 (E22), SimpleHTTPServer 0.6 (E30). Pueden incluir CVE de otras versiones del producto.
- **Servicios sin versión detectada:** 1883/tcp (mqtt), 5672/tcp (amqp), 36395/tcp (tcpwrapped) en 172.17.0.5; Apache ActiveMQ en 61613/tcp (E04, sin versión numérica). Sin versión no fue posible buscar CVE.
- **Servicio sin CPE claro:** Apache Jserv en 172.17.0.4:8009 (E13) — no se buscaron CVE por falta de versión clara.
- **Mismo CVE en múltiples equipos:** CVE-2026-123158 aparece en 6 equipos con OpenSSH; CVE-2024-38475 en 4 equipos con Apache httpd. Una persona debe verificar si cada instancia está realmente afectada según su configuración.
