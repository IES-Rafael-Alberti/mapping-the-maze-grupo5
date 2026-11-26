# Informe de enumeración — Red 172.17.0.0/24

## 1. Resumen ejecutivo

Se identificaron 9 equipos activos en la red autorizada, ejecutando un total de 25 servicios TCP. Los riesgos más graves concentrados en tres servicios: Apache ActiveMQ 5.15.15 (172.17.0.5), Samba 4 (172.17.0.8) y Apache httpd 2.4.58/2.4.59 (172.17.0.3, 172.17.0.5, 172.17.0.8), todos con vulnerabilidades presentes en el catálogo KEV de CISA y, en el caso de ActiveMQ y Samba, con actividad de ransomware confirmada en campo. Se recomienda revisar con prioridad la exposición de ActiveMQ y Samba, y verificar si las versiones de Apache httpd y OpenSSH han sido parcheadas mediante backport en Debian/Ubuntu.

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
| 172.17.0.5 | 1883/tcp | mqtt | E04 |
| 172.17.0.5 | 5672/tcp | amqp | E04 |
| 172.17.0.5 | 8161/tcp | Jetty 9.4.39.v20210325 | E04 |
| 172.17.0.5 | 40833/tcp | tcpwrapped | E04 |
| 172.17.0.5 | 61613/tcp | Apache ActiveMQ | E04 |
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

Fecha de la consulta: 2026-11-25 (EPSS y KEV cambian a diario).

| # | Hallazgo | Equipo y servicio | CVSS (versión) | EPSS | KEV | CWE | Nivel de evidencia | Evidencia |
|---|---|---|---|---|---|---|---|---|
| 1 | CVE-2023-46604 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 9.8 (3.1) | 1.000 | desde 2023-11-02 · ransomware | CWE-502 | potencial (solo versión) | E19 |
| 2 | CVE-2024-38475 | 172.17.0.3 · 80/tcp Apache httpd 2.4.58 | 9.1 (3.1) | 1.000 | desde 2025-05-01 | CWE-116 | potencial (solo versión) | E12 |
| 3 | CVE-2023-44487 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 7.5 (3.1) | 1.000 | desde 2023-10-10 | CWE-400 | potencial (solo versión) | E14 |
| 4 | CVE-2025-24813 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.999 | desde 2025-04-01 | CWE-44, CWE-502, CWE-706 | potencial (solo versión) | E14 |
| 5 | CVE-2017-7494 | 172.17.0.8 · 139/tcp Samba smbd 4 | 9.8 (3.1) | 0.994 | desde 2023-03-30 · ransomware | CWE-94 | potencial (solo versión) | E25 |
| 6 | CVE-2020-1472 | 172.17.0.8 · 139/tcp Samba smbd 4 | 5.5 (3.1) | 0.994 | desde 2021-11-03 · ransomware | sin dato | potencial (solo versión) | E25 |
| 7 | CVE-2020-1938 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.993 | desde 2022-03-03 | sin dato | potencial (solo versión) | E14 |
| 8 | CVE-2026-34197 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 8.8 (3.1) | 0.983 | desde 2026-04-16 | CWE-20, CWE-78, CWE-94 | potencial (solo versión) | E19 |
| 9 | CVE-2026-123158 | 172.17.0.4 · 22/tcp OpenSSH 7.9p1 | 8.1 (3.1) | 0.955 | desde 2026-11-09 | CWE-394 | potencial (solo versión) | E13 |
| 10 | CVE-2024-6387 | 172.17.0.6 · 22/tcp OpenSSH 9.2p1 | 8.1 (3.1) | 0.995 | no | CWE-362, CWE-364 | potencial (solo versión) | E20 |

**Justificación del orden:**

1. **CVE-2023-46604**: En KEV desde 2023-11-02 con ransomware confirmado; EPSS 1.000, CVSS 9.8.
2. **CVE-2024-38475**: En KEV desde 2025-05-01; EPSS 1.000, CVSS 9.1.
3. **CVE-2023-44487**: En KEV desde 2023-10-10; EPSS 1.000, CVSS 7.5.
4. **CVE-2025-24813**: En KEV desde 2025-04-01; EPSS 0.999, CVSS 9.8.
5. **CVE-2017-7494**: En KEV desde 2023-03-30 con ransomware confirmado; EPSS 0.994, CVSS 9.8.
6. **CVE-2020-1472**: En KEV desde 2021-11-03 con ransomware confirmado; EPSS 0.994, CVSS 5.5.
7. **CVE-2020-1938**: En KEV desde 2022-03-03; EPSS 0.993, CVSS 9.8.
8. **CVE-2026-34197**: En KEV desde 2026-04-16; EPSS 0.983, CVSS 8.8.
9. **CVE-2026-123158**: En KEV desde 2026-11-09; EPSS 0.955, CVSS 8.1.
10. **CVE-2024-6387**: No en KEV; EPSS 0.995, CVSS 8.1 — mayor EPSS entre las vulnerabilidades fuera de KEV.

## 4. A verificar

- **Versiones con posible backport (Debian/Ubuntu):** Apache httpd 2.4.58 ((Ubuntu)) en 172.17.0.3 y 172.17.0.8; Apache httpd 2.4.59 ((Debian)) en 172.17.0.5; OpenSSH 7.9p1 Debian 10+deb10u4 en 172.17.0.4; OpenSSH 9.2p1 Debian 2+deb12u2 en 172.17.0.6; OpenSSH 9.2p1 Debian 2+deb12u10 en 172.17.0.9; OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 en 172.17.0.8; OpenSSH 10.3p1 Debian 5 en 172.17.0.1; Apache httpd 2.4.68 ((Debian)) en 172.17.0.9. Estos sistemas podrían tener parches retroportados que no reflejan la versión upstream.
- **Búsquedas por palabras clave (más ruido):** vsftpd 2.3.4 (E11), Jetty 9.4.39.v20210325 (E17, E18), Jetty 10.0.18 (E21), SimpleHTTPServer 0.6 (E29). Las coincidencias pueden incluir CVE de versiones vecinas.
- **Servicios sin versión detectada:** 172.17.0.4:8009 (Apache Jserv), 172.17.0.5:1883 (mqtt), 172.17.0.5:5672 (amqp), 172.17.0.5:40833 (tcpwrapped), 172.17.0.5:61613 (Apache ActiveMQ). Sin versión no fue posible buscar CVE con precisión.
- **Consultas sin CVE encontrados:** Jetty 9.4.39.v20210325 (E17, E18), Jetty 10.0.18 (E21), SimpleHTTPServer 0.6 (E29), Apache Jserv (E31). No encontrar CVE no demuestra que el servicio sea seguro; puede deberse a limitaciones del método de búsqueda por palabras clave o a un CPE no reconocido.
