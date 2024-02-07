# Walkthrough de HackTheBox Machine - SAU
Este repositorio contiene una detallada guía paso a paso sobre cómo resolví la máquina "Sau" en Hack The Box. Mi objetivo es proporcionar tanto a los entusiastas de la ciberseguridad como a los profesionales, una referencia útil y educativa que puedan seguir para entender y replicar el proceso de resolución.

## Escaneo Inicial

Comenzamos el proceso de análisis de la máquina SAU mediante un escaneo Nmap, revelando que los puertos 22, 80 y 55555 están abiertos.

nmap -p 22,80,55555 <IP>
