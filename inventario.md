# Inventario

El inventario entregado por el cliente se encuentra desactualizado (revisiones de entre **2017 y 2020**) y no refleja la realidad actual de la red: **6 de los 8 equipos activos** en el reconocimiento de la red no figuran en el documento, y los **2 únicos que coinciden en IP** ejecutan sistemas y servicios totalmente distintos a los registrados.
## 1. Discrepancias críticas

| **IP**            | `inventario-cliente.csv`                                     | **Detectado**                                                                                 | **Discrepancia**                                                                                                        |
| ----------------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **`172.17.0.9`**  | `TPV-RECEPCION` — Windows 7, RDP (`3389`)                    | Linux (Debian 12): `22/tcp` (OpenSSH 9.2p1 Debian 2+deb12u10), `80/tcp` (Apache httpd 2.4.68) | **Sistema operativo y servicios distintos.** No es un TPV Windows ni tiene RDP expuesto; es un servidor web/SSH Debian. |
| **`172.17.0.10`** | `SRV-PORTAL01` — Windows Server 2008 R2, IIS 7.5 (`80, 443`) | `80/tcp` (SimpleHTTPServer 0.6 / Python 3.11.2) + `623/udp` (IPMI 2.0)                        | **Servicio y tecnología distintos.** No hay IIS ni HTTPS (`443`); sirve contenido con Python y expone gestión IPMI.     |
## 2. Activos no inventariados

| **IP**           | **Sistema operativo inferido** | **Puertos y servicios reales**                                                                                                                                                  |
| ---------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`172.17.0.3`** | Linux (Ubuntu)                 | `21/tcp` (vsftpd 2.3.4), `80/tcp` (Apache httpd 2.4.58 Ubuntu)                                                                                                                  |
| **`172.17.0.4`** | Linux (Debian 10)              | `22/tcp` (OpenSSH 7.9p1 Debian), `8009/tcp` (Apache Jserv / AJP 1.3), `8080/tcp` (Apache Tomcat 9.0.30)                                                                         |
| **`172.17.0.5`** | Linux (Debian)                 | `80/443` (Apache 2.4.59 Debian), `1883` (MQTT), `5672` (AMQP), `8161/61614` (Jetty 9.4.39), `39789` (tcpwrapped), `61613` (STOMP ActiveMQ), `61616` (ActiveMQ OpenWire 5.15.15) |
| **`172.17.0.6`** | Linux (Debian 12)              | `22/tcp` (OpenSSH 9.2p1 Debian), `8080/tcp` (Jetty 10.0.18 / Jenkins)                                                                                                           |
| **`172.17.0.7`** | Linux / Unix                   | `22/tcp` (OpenSSH 7.7)                                                                                                                                                          |
| **`172.17.0.8`** | Linux (Ubuntu)                 | `22/tcp` (OpenSSH 9.6p1 Ubuntu), `80/tcp` (Apache 2.4.58 Ubuntu), `139/445` (Samba smbd 4)                                                                                      |
## 3. Activos inventariados que no aparecen en el reconocimiento

**8 registros** figuran en el CSV del cliente pero no están presentes en el reconocimiento (pueden haber sido retirados, apagados, reubicados en otro segmento IP o bloqueados por firewall):

- **`172.17.0.2`** (`FTP-ALMACEN` — Windows XP SP3, FileZilla Server 0.9.41)
- **`172.17.0.11`** (`SRV-PORTAL01` — Windows Server 2008, SQL Server 2008 R2 SP1)
- **`172.17.0.20`** (`srv-mensajeria` — Windows Server 2003, MSMQ 3.0)
- **`172.17.0.30`** (`IMPRESORA-MUELLE` — HP LaserJet P2055)
- **`172.17.0.50`** (`NAS-OFICINA` — Synology DSM 5.2)
- **`172.17.0.100`** (`srv-backup` — Debian 7, OpenSSH 5.5)
- **`172.17.0.254`** (`consola sala tecnica` — HP iLO 2)
- **`172.17.1.6`** (`PC-PRUEBAS` — Ubuntu 12.04) → **Pertenece a otra subred (`172.17.1.0/24`) y su nota indica "Lo apago cuando no lo uso".**

---
## Corrección

| IP                | Equipo / Hostname                       | Dirección MAC       | Sistema Operativo (Inferido) | Puertos                                                                                                             | Servicios y Versiones Detectadas                                                                                                                                       | Estado respecto al inventario previo                                      |
| :---------------- | :-------------------------------------- | :------------------ | :--------------------------- | :------------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| **`172.17.0.3`**  | *Sin asignar*                           | `EA:FB:EF:D2:DD:F1` | Linux (Ubuntu)               | `21/tcp`<br>`80/tcp`                                                                                                | `vsftpd 2.3.4`<br>`Apache httpd 2.4.58 ((Ubuntu))`                                                                                                                     | **No inventariado**                                                       |
| **`172.17.0.4`**  | *Sin asignar*                           | `72:83:A3:FF:A6:D3` | Linux (Debian 10)            | `22/tcp`<br>`8009/tcp`<br>`8080/tcp`                                                                                | `OpenSSH 7.9p1 Debian 10+deb10u4`<br>`Apache Jserv (Protocol v1.3 / AJP13)`<br>`Apache Tomcat 9.0.30`                                                                  | **No inventariado**                                                       |
| **`172.17.0.5`**  | *Sin asignar*                           | `C2:EB:1A:D6:E7:04` | Linux (Debian)               | `80/tcp, 443/tcp`<br>`1883/tcp`<br>`5672/tcp`<br>`8161/tcp, 61614/tcp`<br>`39789/tcp`<br>`61613/tcp`<br>`61616/tcp` | `Apache httpd 2.4.59 ((Debian))`<br>`MQTT`<br>`AMQP`<br>`Jetty 9.4.39.v20210325`<br>`tcpwrapped`<br>`Apache ActiveMQ (STOMP)`<br>`ActiveMQ OpenWire transport 5.15.15` | **No inventariado**                                                       |
| **`172.17.0.6`**  | *Sin asignar*                           | `3A:0D:4E:B3:B3:92` | Linux (Debian 12)            | `22/tcp`<br>`8080/tcp`                                                                                              | `OpenSSH 9.2p1 Debian 2+deb12u2`<br>`Jetty 10.0.18 (Jenkins 2.441)`                                                                                                    | **No inventariado**                                                       |
| **`172.17.0.7`**  | *Sin asignar*                           | `3E:8E:8B:DE:39:B7` | Linux / Unix                 | `22/tcp`                                                                                                            | `OpenSSH 7.7`                                                                                                                                                          | **No inventariado**                                                       |
| **`172.17.0.8`**  | `SAMBASERVER` (`ESEEMEB.DL`)            | `DA:41:B3:D0:CB:9D` | Linux (Ubuntu 24.04)         | `22/tcp`<br>`80/tcp`<br>`139/tcp, 445/tcp`                                                                          | `OpenSSH 9.6p1 Ubuntu 3ubuntu13.5`<br>`Apache httpd 2.4.58 ((Ubuntu))`<br>`Samba smbd 4`                                                                               | **No inventariado**                                                       |
| **`172.17.0.9`**  | *Sin asignar* *(antes `TPV-RECEPCION`)* | `06:FE:85:80:3E:A0` | Linux (Debian 12)            | `22/tcp`<br>`80/tcp`                                                                                                | `OpenSSH 9.2p1 Debian 2+deb12u10`<br>`Apache httpd 2.4.68 ((Debian))`                                                                                                  | **Discrepancia total** (figuraba como Windows 7 con RDP `3389`)           |
| **`172.17.0.10`** | *Sin asignar* *(antes `SRV-PORTAL01`)*  | `12:4F:2B:31:4F:86` | Linux / Unix + BMC           | `80/tcp`<br>`623/udp`                                                                                               | `SimpleHTTPServer 0.6 (Python 3.11.2)`<br>`IPMI 2.0 (asf-rmcp)`                                                                                                        | **Discrepancia total** (figuraba como Windows Server 2008 R2 con IIS 7.5) |

---

### Activos del inventario legado pendientes de baja o verificación

Los siguientes equipos figuran en `inventario-cliente.csv` pero **no respondieron** durante el reconocimiento en `172.17.0.0/24`:

| IP | Equipo registrado | Sistema / Servicio registrado | Acción recomendada |
| :--- | :--- | :--- | :--- |
| `172.17.0.2` | `FTP-ALMACEN` | Windows XP SP3 / FileZilla Server 0.9.41 | Confirmar si fue retirado o reubicado |
| `172.17.0.11` | `SRV-PORTAL01` | Windows Server 2008 / SQL Server 2008 R2 | Confirmar baja (registro duplicado con `.10`) |
| `172.17.0.20` | `srv-mensajeria` | Windows Server 2003 / MSMQ 3.0 | Confirmar si fue retirado o filtrado |
| `172.17.0.30` | `IMPRESORA-MUELLE` | HP LaserJet P2055 (`9100`) | Verificar si sigue activa o cambió de IP |
| `172.17.0.50` | `NAS-OFICINA` | Synology DSM 5.2 (`445`) | Confirmar baja o cambio de segmento |
| `172.17.0.100` | `srv-backup` | Debian 7 / OpenSSH 5.5 | Confirmar baja (nota previa: *"Creo que ya no funciona"*) |
| `172.17.0.254` | `consola sala tecnica` | HP iLO 2 (`443`) | Verificar acceso físico/VLAN de gestión |
| `172.17.1.6` | `PC-PRUEBAS` | Ubuntu 12.04 (`3000, 5000, 9090`) | Fuera de subred (`172.17.1.0/24`) / equipo intermitente |
