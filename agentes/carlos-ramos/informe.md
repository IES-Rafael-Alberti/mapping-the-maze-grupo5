# Informe de enumeración de red 172.17.0.0/24

**Fecha de consulta:** 2026-11-25 (EPSS y KEV cambian a diario)

## 1. Resumen ejecutivo

Se detectaron 9 equipos activos en la red autorizada, con un total de 23 servicios TCP en funcionamiento. Los riesgos más críticos corresponden a vulnerabilidades en el catálogo KEV de CISA presentes en Apache httpd, ActiveMQ, Apache Tomcat y Samba, varias con alto EPSS (probabilidad de explotación >95%) y, en el caso de ActiveMQ y Samba, asociadas a ataques de ransomware. Se recomienda revisar con prioridad los servicios de ActiveMQ en 172.17.0.5, las instancias de Samba en 172.17.0.8 y las versiones de Apache httpd y OpenSSH desplegadas en múltiples equipos.

## 2. Equipos y servicios

| Equipo | Puerto | Servicio y versión | Evidencia |
|---|---|---|---|
| 172.17.0.1 | 22/tcp | OpenSSH 10.3p1 Debian 5 | E02 |
| 172.17.0.3 | 21/tcp | vsftpd 2.3.4 | E03 |
| 172.17.0.3 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E03 |
| 172.17.0.4 | 22/tcp | OpenSSH 7.9p1 Debian 10+deb10u4 | E04 |
| 172.17.0.4 | 8009/tcp | Apache Jserv (Protocol v1.3) | E04 |
| 172.17.0.4 | 8080/tcp | Apache Tomcat 9.0.30 | E04 |
| 172.17.0.5 | 80/tcp | Apache httpd 2.4.59 ((Debian)) | E05 |
| 172.17.0.5 | 443/tcp | Apache httpd 2.4.59 ((Debian)) | E05 |
| 172.17.0.5 | 1883/tcp | mqtt | E05 |
| 172.17.0.5 | 5672/tcp | amqp | E05 |
| 172.17.0.5 | 8161/tcp | Jetty 9.4.39.v20210325 | E05 |
| 172.17.0.5 | 40761/tcp | tcpwrapped | E05 |
| 172.17.0.5 | 61613/tcp | stomp · Apache ActiveMQ | E05 |
| 172.17.0.5 | 61614/tcp | Jetty 9.4.39.v20210325 | E05 |
| 172.17.0.5 | 61616/tcp | apachemq · ActiveMQ OpenWire transport 5.15.15 | E05 |
| 172.17.0.6 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u2 | E06 |
| 172.17.0.6 | 8080/tcp | Jetty 10.0.18 | E06 |
| 172.17.0.7 | 22/tcp | OpenSSH 7.7 | E07 |
| 172.17.0.8 | 22/tcp | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | E08 |
| 172.17.0.8 | 80/tcp | Apache httpd 2.4.58 ((Ubuntu)) | E08 |
| 172.17.0.8 | 139/tcp | netbios-ssn · Samba smbd 4 | E08 |
| 172.17.0.8 | 445/tcp | netbios-ssn · Samba smbd 4 | E08 |
| 172.17.0.9 | 22/tcp | OpenSSH 9.2p1 Debian 2+deb12u10 | E09 |
| 172.17.0.9 | 80/tcp | Apache httpd 2.4.68 ((Debian)) | E09 |
| 172.17.0.10 | 80/tcp | SimpleHTTPServer 0.6 (Python 3.11.2) | E10 |

## 3. Vulnerabilidades potenciales priorizadas

| # | Hallazgo | Equipo y servicio | CVSS (versión) | EPSS | KEV | CWE | Nivel de evidencia | Evidencia |
|---|---|---|---|---|---|---|---|---|
| 1 | CVE-2023-46604 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 9.8 (3.1) | 1.000 | desde 2023-11-02 · ransomware | CWE-502 | potencial (solo versión) | E20 |
| 2 | CVE-2024-38475 | 172.17.0.3 · 80/tcp Apache httpd 2.4.58 | 9.1 (3.1) | 1.000 | desde 2025-05-01 | CWE-116 | potencial (solo versión) | E13 |
| 3 | CVE-2023-44487 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 7.5 (3.1) | 1.000 | desde 2023-10-10 | CWE-400 | potencial (solo versión) | E15 |
| 4 | CVE-2025-24813 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.999 | desde 2025-04-01 | CWE-44, CWE-502, CWE-706 | potencial (solo versión) | E15 |
| 5 | CVE-2017-7494 | 172.17.0.8 · 139/tcp Samba smbd 4 | 9.8 (3.1) | 0.994 | desde 2023-03-30 · ransomware | CWE-94 | potencial (solo versión) | E26 |
| 6 | CVE-2020-1472 | 172.17.0.8 · 139/tcp Samba smbd 4 | 5.5 (3.1) | 0.994 | desde 2021-11-03 · ransomware | sin dato | potencial (solo versión) | E26 |
| 7 | CVE-2020-1938 | 172.17.0.4 · 8080/tcp Apache Tomcat 9.0.30 | 9.8 (3.1) | 0.993 | desde 2022-03-03 | sin dato | potencial (solo versión) | E15 |
| 8 | CVE-2026-34197 | 172.17.0.5 · 61616/tcp ActiveMQ OpenWire transport 5.15.15 | 8.8 (3.1) | 0.983 | desde 2026-04-16 | CWE-20, CWE-78, CWE-94 | potencial (solo versión) | E20 |
| 9 | CVE-2026-123158 | 172.17.0.1 · 22/tcp OpenSSH 10.3p1 Debian 5 | 8.1 (3.1) | 0.955 | desde 2026-11-09 | CWE-394 | potencial (solo versión) | E11 |
| 10 | CVE-2024-6387 | 172.17.0.6 · 22/tcp OpenSSH 9.2p1 Debian 2+deb12u2 | 8.1 (3.1) | 0.995 | no | CWE-362, CWE-364 | potencial (solo versión) | E21 |

**Criterios de ordenación:**

1. **CVE-2023-46604:** En KEV con ransomware, EPSS 1.000 (máximo), CVSS 9.8. Prioridad absoluta por presencia en KEV y probabilidad de explotación máxima.
2. **CVE-2024-38475:** En KEV, EPSS 1.000 (máximo), CVSS 9.1. Desempate con CVE-2023-46604 por CVSS (9.8 > 9.1).
3. **CVE-2023-44487:** En KEV, EPSS 1.000 (máximo), CVSS 7.5. Desempate con las anteriores por CVSS (9.1 > 7.5).
4. **CVE-2025-24813:** En KEV, EPSS 0.999, CVSS 9.8. Segundo lugar en EPSS entre las KEV.
5. **CVE-2017-7494:** En KEV con ransomware, EPSS 0.994, CVSS 9.8. Empate en EPSS con CVE-2020-1472, desempate por CVSS (9.8 > 5.5).
6. **CVE-2020-1472:** En KEV con ransomware, EPSS 0.994, CVSS 5.5.
7. **CVE-2020-1938:** En KEV, EPSS 0.993, CVSS 9.8.
8. **CVE-2026-34197:** En KEV, EPSS 0.983, CVSS 8.8.
9. **CVE-2026-123158:** En KEV, EPSS 0.955, CVSS 8.1. Afecta también a 172.17.0.4, 172.17.0.6, 172.17.0.7, 172.17.0.8 y 172.17.0.9 (todos con OpenSSH).
10. **CVE-2024-6387:** Fuera de KEV, EPSS 0.995, CVSS 8.1. Mayor EPSS entre las no-KEV, por eso cierra el top 10.

## 4. A verificar

- **Versiones con backport de distribuciones:** OpenSSH 10.3p1 **Debian 5** (172.17.0.1), OpenSSH 7.9p1 **Debian 10+deb10u4** (172.17.0.4), OpenSSH 9.2p1 **Debian 2+deb12u2** (172.17.0.6), OpenSSH 9.2p1 **Debian 2+deb12u10** (172.17.0.9), OpenSSH 9.6p1 **Ubuntu 3ubuntu13.5** (172.17.0.8), Apache httpd 2.4.58 **(Ubuntu)** (172.17.0.3 y 172.17.0.8), Apache httpd 2.4.59 **(Debian)** (172.17.0.5). Las versiones de los proveedores pueden incluir parches no reflejados en el número base del software.
- **Búsquedas por palabras clave (más ruido):** vsftpd 2.3.4 (E12), Jetty 9.4.39.v20210325 (E18, E19), Jetty 10.0.18 (E22), SimpleHTTPServer 0.6 (E30). Los resultados pueden incluir falsos positivos.
- **Servicios sin versión detectada:** mqtt (172.17.0.5:1883), amqp (172.17.0.5:5672), tcpwrapped (172.17.0.5:40761), stomp · Apache ActiveMQ (172.17.0.5:61613). No se pudo consultar CVEs específicas por falta de versión numérica.
- **Servicios sin CVEs encontradas:** Jetty 9.4.39.v20210325 (172.17.0.5:8161 y 61614), Jetty 10.0.18 (172.17.0.6:8080), SimpleHTTPServer 0.6 (172.17.0.10:80). Puede deberse a búsqueda por palabras clave o a que la versión es reciente/parcheada.
- **CVEs duplicadas por servicio:** CVE-2024-38475 aparece en 172.17.0.3:80, 172.17.0.5:80, 172.17.0.5:443 y 172.17.0.8:80; CVE-2026-123158 en múltiples equipos con OpenSSH; CVEs de Samba en ambos puertos 139 y 445 de 172.17.0.8. Una persona debería consolidarlas.
