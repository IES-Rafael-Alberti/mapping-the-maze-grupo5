# Informe de enumeración de red interna — 172.17.0.0/24

## 1. Resumen ejecutivo

Se identificaron 9 equipos activos en la red autorizada, ejecutando un total de 20 servicios TCP con versiones detectables. Los riesgos más graves corresponden a Apache ActiveMQ 5.15.15 (con vulnerabilidades críticas en KEV asociadas a ransomware), Apache httpd 2.4.58/2.4.59 en tres equipos con una vulnerabilidad crítica en KEV, y Apache Tomcat 9.0.30 con dos vulnerabilidades en KEV. Conviene revisar primero ActiveMQ en 172.17.0.5, luego los servidores Apache y Tomcat en 172.17.0.3, 172.17.0.5 y 172.17.0.4.

## 2. Equipos y servicios

| Equipo | Puerto | Servicio y versión | Evidencia |
|---|---|---|---|
| 172.17.0.3 | 21/tcp | vsftpd 2.3.4 | E02 |
| 172.17.0.3 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E02 |
| 172.17.0.4 | 22/tcp | OpenSSH 7.9p1 Debian 10+deb10u4 | E03 |
| 172.17.0.4 | 8009/tcp | Apache Jserv (Protocol v1.3) | E03 |
| 172.17.0.4 | 8080/tcp | Apache Tomcat 9.0.30 | E03 |
| 172.17.0.5 | 80/tcp | Apache httpd 2.4.59 ((Debian)) | E04 |
| 172.17.0.5 | 443/tcp | Apache httpd 2.4.59 ((Debian)) | E04 |
| 172.17.0.5 | 8161/tcp | Jetty 9.4.39.v20210325 | E04 |
| 172.17.0.5 | 61616/tcp | ActiveMQ OpenWire transport 5.15.15 | E04 |
| 172.17.0.6 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u2 | E05 |
| 172.17.0.6 | 8080/tcp | Jetty 10.0.18 | E05 |
| 172.17.0.7 | 22/tcp | OpenSSH 7.7 | E06 |
| 172.17.0.8 | 22/tcp | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | E07 |
| 172.17.0.8 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E07 |
| 172.17.0.8 | 445/tcp | Samba smbd 4 | E07 |
| 172.17.0.9 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u10 | E08 |
| 172.17.0.9 | 80/tcp | Apache httpd 2.4.68 ((Debian)) | E08 |
| 172.17.0.10 | 80/tcp | SimpleHTTPServer 0.6 (Python 3.11.2) | E09 |
| 172.17.0.1 | 22/tcp | OpenSSH 10.3p1 Debian 5 | E10 |

Servicios sin versión detectada (omitidos de la tabla pero presentes): 172.17.0.5 puertos 1883 (mqtt), 5672 (amqp), 40763 (tcpwrapped), 61613 (stomp), 61614 (Jetty 9.4.39.v20210325); 172.17.0.8 puerto 139 (Samba smbd 4).

## 3. Vulnerabilidades potenciales priorizadas

Consulta de EPSS y KEV: 2026-11-25.

| # | Hallazgo | Equipo y servicio | CVSS (versión) | EPSS | KEV | CWE | Nivel de evidencia | Evidencia |
|---|---|---|---|---|---|---|---|---|
| 1 | CVE-2023-46604 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 9.8 (3.1) | 1.000 | desde 2023-11-02 · ransomware | CWE-502 | potencial (solo versión) | E19 |
| 2 | CVE-2024-38475 | 172.17.0.3 · 80/tcp Apache httpd 2.4.58 | 9.1 (3.1) | 1.000 | desde 2025-05-01 | CWE-116 | potencial (solo versión) | E12 |
| 3 | CVE-2023-44487 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 7.5 (3.1) | 1.000 | desde 2023-10-10 | CWE-400 | potencial (solo versión) | E14 |
| 4 | CVE-2025-24813 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.999 | desde 2025-04-01 | CWE-44, CWE-502, CWE-706 | potencial (solo versión) | E14 |
| 5 | CVE-2017-7494 | 172.17.0.8 · 445/tcp Samba smbd 4 | 9.8 (3.1) | 0.994 | desde 2023-03-30 · ransomware | CWE-94 | potencial (solo versión) | E25 |
| 6 | CVE-2020-1938 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.993 | desde 2022-03-03 | sin dato | potencial (solo versión) | E14 |
| 7 | CVE-2026-34197 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 8.8 (3.1) | 0.983 | desde 2026-04-16 | CWE-20, CWE-78, CWE-94 | potencial (solo versión) | E19 |
| 8 | CVE-2026-123158 | 172.17.0.4 · 22/tcp OpenSSH 7.9p1 Debian 10+deb10u4 | 8.1 (3.1) | 0.955 | desde 2026-11-09 | CWE-394 | potencial (solo versión) | E13 |
| 9 | CVE-2024-6387 | 172.17.0.6 · 22/tcp OpenSSH 9.2p1 Debian 2+deb12u2 | 8.1 (3.1) | 0.995 | no | CWE-362, CWE-364 | potencial (solo versión) | E20 |
| 10 | CVE-2011-2523 | 172.17.0.3 · 21/tcp vsftpd 2.3.4 | 9.8 (3.1) | 0.962 | no | CWE-78 | potencial (solo versión) | E11 |

**Justificación del orden:**

1. **Fila 1:** En KEV, EPSS 1.000 (máximo), CVSS 9.8 (máximo). Además, asociada a ransomware.
2. **Fila 2:** En KEV, EPSS 1.000 (máximo), CVSS 9.1 (segundo más alto dentro de EPSS 1.000).
3. **Fila 3:** En KEV, EPSS 1.000 (máximo), CVSS 7.5 (tercero dentro de EPSS 1.000).
4. **Fila 4:** En KEV, EPSS 0.999, CVSS 9.8.
5. **Fila 5:** En KEV, EPSS 0.994, CVSS 9.8. Asociada a ransomware.
6. **Fila 6:** En KEV, EPSS 0.993, CVSS 9.8.
7. **Fila 7:** En KEV, EPSS 0.983, CVSS 8.8.
8. **Fila 8:** En KEV, EPSS 0.955, CVSS 8.1. Múltiples equipos OpenSSH afectados (172.17.0.1, .4, .6, .7, .8, .9).
9. **Fila 9:** No en KEV. EPSS 0.995, CVSS 8.1. Primera fuera de KEV por mayor EPSS.
10. **Fila 10:** No en KEV. EPSS 0.962, CVSS 9.8.

## 4. A verificar

- **Versiones con distribución (posible backport):** OpenSSH 7.9p1 Debian 10+deb10u4 (172.17.0.4), OpenSSH 9.2p1 Debian 2+deb12u2 (172.17.0.6), OpenSSH 9.2p1 Debian 2+deb12u10 (172.17.0.9), OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (172.17.0.8), OpenSSH 10.3p1 Debian 5 (172.17.0.1), Apache httpd 2.4.58 (Ubuntu) (172.17.0.3 y 172.17.0.8), Apache httpd 2.4.59 (Debian) (172.17.0.5), Apache httpd 2.4.68 (Debian) (172.17.0.9). Las vulnerabilidades listadas pueden estar ya parcheadas a nivel de distribución sin cambiar el número de versión del producto upstream.
- **Búsquedas por palabras clave (más ruido):** vsftpd 2.3.4 en 172.17.0.3:21 — la búsqueda fue por palabras clave, no por CPE, por lo que los resultados pueden incluir falsos positivos.
- **Servicios sin versión detectada:** 172.17.0.5 puertos 1883 (mqtt), 5672 (amqp), 40763 (tcpwrapped); 172.17.0.4 puerto 8009 (ajp13). No se pudieron buscar CVEs sin versión.
- **Consultas que no arrojaron CVEs:** Jetty 9.4.39 (172.17.0.5:8161, E17), ActiveMQ sin versión 172.17.0.5:61613 (E18), Jetty 10.0.18 (172.17.0.6:8080, E21), SimpleHTTPServer 0.6 (172.17.0.10:80, E28).
