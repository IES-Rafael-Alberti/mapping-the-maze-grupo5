# Informe de revisión de seguridad interna — Gadir Logística, S.L.

**Red revisada:** `172.17.0.0/24`  
**Fecha del informe:** 24 de noviembre de 2026  
**Tipo de trabajo:** descubrimiento de equipos, enumeración sin autenticación y análisis no intrusivo. No se ha explotado ninguna vulnerabilidad, ni se han probado contraseñas ni realizado escaneos agresivos.

---

## 1. Resumen para Gerencia

1. **Sobre el aviso de «sshole» (CVE-2026-123158):** con la información de que disponemos, **no hemos encontrado ningún servidor de Gadir que sea claramente vulnerable**. Cuatro de los cinco servidores con acceso SSH usan Debian o Ubuntu, que según la propia noticia **no están afectados**. **Queda un servidor (`172.17.0.7`) que no podemos descartar** hasta que el administrador haga una comprobación de unos minutos (ver sección 2).
2. **El problema real de la red no es sshole.** La revisión ha detectado **vulnerabilidades más graves y más fáciles de aprovechar** que ese fallo, en especial en el servidor `172.17.0.5` (mensajería ActiveMQ), en el servidor de ficheros `172.17.0.8` (acceso anónimo a información y a un recurso compartido) y en `172.17.0.3` (servicio FTP con una versión conocida por llevar una puerta trasera).
3. **El inventario de equipos está muy desactualizado** (revisiones de 2017 a 2020). De los 8 equipos que responden en la red, **6 no figuran** en él y los 2 restantes son distintos de lo que dice el documento. Sin saber qué equipos hay, no se pueden mantener actualizados.
4. **Qué hacer primero:** corregir los cinco puntos de la sección 3 (prioridad 1) y, en paralelo, hacer la comprobación del servidor `172.17.0.7`.

---

## 2. Respuesta al aviso recibido («sshole»)

> **Asunto: Re: Ojo con esto — fallo «sshole» (CVE-2026-123158)**
> 
> Buenas,
> 
> No hace falta preocuparse en exceso, pero sí hay que cerrar una comprobación. Os resumimos lo importante.
> 
> **¿Qué es?** Un fallo en OpenSSH (el programa con el que se entra en remoto a los servidores). Permite entrar en ciertas cuentas enviando una contraseña extremadamente larga, sin conocer la verdadera. OpenSSH publicó la corrección el 5 de noviembre de 2026 (versión 10.6p2).
> 
> **¿Nos afecta?** El fallo solo se da si se cumplen **a la vez** varias condiciones: el servidor usa el sistema Alpine Linux u otro basado en «musl» (OpenWrt, Buildroot, Yocto), no usa el sistema de autenticación PAM, permite entrar con contraseña, y existe alguna cuenta sin contraseña (por ejemplo, de acceso solo con clave). **Debian, Ubuntu y Fedora no están afectados.**
> 
> En vuestra red hay cinco servidores con acceso SSH:
> 
>|Servidor|Sistema detectado|¿Afectado?|
> |---|---|---|
> |`172.17.0.4`|Debian 10|No, según los criterios de la noticia|
> |`172.17.0.6`|Debian 12|No, según los criterios de la noticia|
> |`172.17.0.8`|Ubuntu 24.04|No, según los criterios de la noticia|
> |`172.17.0.9`|Debian 12|No, según los criterios de la noticia|
> |`172.17.0.7`|**No identificado** (OpenSSH 7.7)|**No se puede descartar todavía**|
> 
> Sobre `172.17.0.7`: no sabemos qué sistema lleva, y no podemos comprobarlo desde fuera sin intentar entrar, que no está permitido en esta revisión. Por eso necesitamos que quien administre ese equipo haga las comprobaciones del apartado «Qué hacer» más abajo.
> 
> **Importante:** nosotros **no hemos probado el fallo** en vuestros servidores. La conclusión se basa en el sistema operativo y la versión que cada servidor muestra a la red. Y que Juanma entrase «por SSH» es normal: SSH es una herramienta habitual y segura; lo que falla es solo esta combinación concreta de configuración.
> 
> **Sobre lo que cuenta Paco:** la noticia que nos habéis reenviado no menciona que el Gobierno de EE. UU. lo haya incluido en su lista de fallos explotados activamente, por lo que no podemos confirmarlo con ese material. Conviene tratarlo como si fuera cierto y actuar con rapidez, pero sin alarma.
> 
> **Qué hacer (para el administrador de sistemas):**
> 
> 1. **En `172.17.0.7`**, comprobar: el sistema (`cat /etc/os-release`); si usa musl (`ls /lib/ld-musl*`); si SSH usa PAM (`grep -i UsePAM /etc/ssh/sshd_config`); y si existen cuentas sin contraseña (`awk -F: '$2=="*"{print $1}' /etc/shadow`).
> 2. **Si resulta ser Alpine/musl sin PAM:** actualizar OpenSSH (`apk upgrade -U -a`) y **reiniciar el servicio** (`rc-service sshd restart`), porque actualizar el paquete sin reiniciar no basta. Si no se puede actualizar ya, desactivar el acceso por contraseña (`PasswordAuthentication no`), comprobando antes que funciona la entrada con clave.
> 3. **Revisar los registros** buscando entradas «Accepted password for» sobre cuentas que no tienen contraseña. Si aparece alguna, tratar el equipo como comprometido.
> 4. **En el resto de servidores:** no hace falta acción por este fallo, pero recomendamos desactivar el acceso SSH con contraseña si ya se usan claves.
> 5. **No descargar ni ejecutar** «comprobadores» o pruebas de concepto de repositorios públicos: es una vía habitual de infección.
> 
> **Para el seguro:** guardad por escrito la fecha de este aviso, nuestra respuesta, el resultado de las comprobaciones y la fecha de cada actualización. Eso demuestra que se actuó con diligencia.
> 
> Un saludo.

---

## 3. Qué corregir primero

Orden de actuación según gravedad, facilidad de aprovechamiento y certeza de la evidencia. Se distingue entre lo **confirmado** (observado directamente) y lo **por confirmar** (deducido de la versión del programa; puede estar ya parcheado).

### Prioridad 1 — Esta semana

|#|Equipo|Problema|Certeza|Acción|
|---|---|---|---|---|
|1|`172.17.0.5`|**Apache ActiveMQ 5.15.15**: permite ejecutar código de forma remota sin credenciales (`CVE-2023-46604`). Además expone otros servicios de mensajería y la consola de administración (Fichas 05 y 12).|Versión detectada; sin parche manual conocido|Actualizar a 5.15.16 o superior (recomendable una versión vigente). Restringir por cortafuegos los puertos de mensajería y la consola `8161` solo a quienes los necesiten.|
|2|`172.17.0.8`|**Servidor de ficheros con acceso anónimo**: se obtiene la lista de 5 usuarios (incluido `administrador`), la política de contraseñas es débil (mínimo 5 caracteres, sin complejidad) y el recurso `myshare` es accesible sin credenciales, con un archivo `access.txt` de contenido desconocido (Ficha 10).|**Confirmado**|Desactivar sesiones anónimas y el acceso anónimo a `myshare`. Exigir firma SMB. Endurecer la política de contraseñas. Revisar el contenido de `access.txt` y cambiar lo que contenga.|
|3|`172.17.0.3`|**vsftpd 2.3.4**: esta versión se distribuyó en 2011 con una puerta trasera (`CVE-2011-2523`). Posible acceso FTP anónimo (Fichas 01 y 10).|Versión detectada; no se ha activado la puerta trasera|Verificar el origen y la integridad del programa. Lo más seguro es sustituirlo por una versión actual o retirar el servicio. Desactivar el acceso anónimo.|
|4|`172.17.0.6`|**Jenkins 2.441**: permite leer archivos arbitrarios del servidor (`CVE-2024-23897`). Jenkins suele guardar claves y credenciales de otros sistemas (Ficha 07).|Versión detectada|Actualizar Jenkins a una versión corregida. Comprobar si el acceso anónimo está activo y si el CLI puede desactivarse.|
|5|`172.17.0.4`|**Tomcat 9.0.30 con el puerto AJP (8009) expuesto** («Ghostcat», `CVE-2020-1938`), más fallos adicionales de la misma versión (Fichas 03 y 04). El sistema, Debian 10, ya no recibe actualizaciones de seguridad.|Versión detectada|Actualizar Tomcat. Cerrar o restringir el puerto `8009` y, si se necesita, configurar secreto compartido (`requiredSecret`). Planificar la migración del sistema operativo.|

### Prioridad 2 — En las próximas semanas

|#|Equipo|Problema|Acción|
|---|---|---|---|
|6|`172.17.0.7`|**Comprobación de sshole** (sección 2) y **OpenSSH 7.7**, antiguo, que permite averiguar qué usuarios existen (`CVE-2018-15473`) (Ficha 08).|Hacer la comprobación de la sección 2. Identificar el sistema, inventariarlo y actualizar OpenSSH.|
|7|`172.17.0.10`|**IPMI 2.0 expuesto** (`623/udp`): permite capturar hashes de contraseñas de la gestión remota de hardware y descifrarlos después (`CVE-2013-4786`). Admite métodos de autenticación débiles, entre ellos el nulo (Ficha 09).|Aislar el puerto en una red de gestión. Desactivar IPMI 1.5 y la autenticación nula. Usar contraseñas largas y únicas.|
|8|`172.17.0.5`|**Certificado HTTPS autofirmado y caducado** desde el 17/04/2025, y **aplicación web de un restaurante en desarrollo** que no tiene relación con el negocio (Ficha 11).|Retirar la aplicación si no se necesita. Si se mantiene, usar un certificado válido y un proceso de renovación.|
|9|`172.17.0.3` y `172.17.0.8`|**Apache 2.4.58** con un fallo conocido de división de respuestas HTTP (`CVE-2023-38709`). Ubuntu suele corregirlo sin cambiar el número de versión, por lo que puede estar ya resuelto (Ficha 02).|Comprobar la versión exacta del paquete `apache2` y aplicar actualizaciones pendientes.|

### Prioridad 3 — Mantenimiento

|#|Equipo|Problema|Acción|
|---|---|---|---|
|10|`172.17.0.5`|Librerías web antiguas: jQuery 3.4.1 y Bootstrap 3.4.1 con fallos de XSS (Ficha 06).|Actualizar las librerías o retirar la aplicación (ver punto 8).|

### Medidas generales

- **Actualizar el inventario** con los equipos reales (sección 4), asignando un responsable a cada uno.
- **Segmentar la red:** los servicios de gestión (IPMI, consolas de administración, Jenkins, ActiveMQ) no deberían ser accesibles desde toda la red interna.
- **Ordenar los accesos tras la salida de Juanma.** Según el mensaje recibido, ya no está en la empresa. Si tenía claves SSH o contraseñas de administración, conviene **revocarlas y cambiarlas**, y dejar documentado quién administra ahora los servidores.
- **Política de actualizaciones:** definir quién revisa avisos como este y con qué frecuencia se aplican parches.

---

## 4. Estado del inventario

**Equipos activos detectados: 8** (`172.17.0.3` a `172.17.0.10`).

|Situación|Equipos|
|---|---|
|**No figuran en el inventario**|`172.17.0.3`, `.4`, `.5`, `.6`, `.7`, `.8`|
|**Figuran, pero son otra cosa**|`172.17.0.9` (el inventario dice TPV con Windows 7; es un servidor Debian 12 con SSH y web) y `172.17.0.10` (el inventario dice Windows Server 2008 R2 con IIS; es un equipo Linux con un servidor web en Python e IPMI)|
|**Figuran, pero no responden**|`172.17.0.2`, `.11`, `.20`, `.30`, `.50`, `.100`, `.254`|

Los equipos del último grupo pueden haberse retirado, apagado, movido de red o estar bloqueados por un cortafuegos. **Hay que confirmarlo con quien corresponda.** Se trata de sistemas muy antiguos (Windows XP, Windows Server 2003, Debian 7…) que, si siguieran en uso, serían un riesgo importante.

El equipo `172.17.1.6` (`PC-PRUEBAS`) pertenece a la subred `172.17.1.0/24`, **fuera del alcance autorizado**, por lo que no se ha revisado. Su nota indica que se apaga cuando no se usa; si sigue existiendo, conviene revisarlo por separado.

---

## 5. Alcance y limitaciones

- Solo se ha analizado `172.17.0.0/24`, con técnicas no intrusivas. **No se ha explotado ninguna vulnerabilidad, probado contraseñas ni descargado archivos** (por ejemplo, `access.txt` no se ha abierto).
- Muchos hallazgos se basan en la **versión que el servicio muestra a la red**. Algunos sistemas (como Ubuntu) corrigen fallos sin cambiar ese número, por lo que algunos puntos pueden estar ya resueltos. Cada ficha indica qué falta por comprobar.
- El estado de **sshole** en `172.17.0.7` está **sin determinar** hasta que se hagan las comprobaciones indicadas.
- La red se revisó en un momento concreto: los equipos que se apagan o cambian de red pueden no haber aparecido.

---

## Anexo — Resumen técnico por equipo

|IP|Sistema (inferido)|Servicios principales|Hallazgos|
|---|---|---|---|
|`172.17.0.3`|Ubuntu|FTP (vsftpd 2.3.4), HTTP (Apache 2.4.58)|Fichas 01, 02, 10|
|`172.17.0.4`|Debian 10|SSH, AJP, Tomcat 9.0.30|Fichas 03, 04|
|`172.17.0.5`|Debian|Apache 2.4.59, ActiveMQ 5.15.15 y mensajería (MQTT, AMQP, STOMP), consola `8161`|Fichas 05, 06, 11, 12|
|`172.17.0.6`|Debian 12|SSH, Jenkins 2.441|Ficha 07|
|`172.17.0.7`|No identificado|SSH (OpenSSH 7.7)|Ficha 08 y comprobación de sshole|
|`172.17.0.8`|Ubuntu 24.04|SSH, Apache 2.4.58, Samba|Fichas 02, 10|
|`172.17.0.9`|Debian 12|SSH, Apache 2.4.68|Sin hallazgos registrados; no coincide con el inventario|
|`172.17.0.10`|Linux + gestión remota|Servidor web Python, IPMI 2.0|Ficha 09|
