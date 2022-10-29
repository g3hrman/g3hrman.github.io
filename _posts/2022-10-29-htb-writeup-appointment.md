---
layout: single
title: Appointment - Hack The Box
excerpt: "Appointment es una máquina muy fácil que forma parte del Tier 1 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando SQL Injection, una vulnerabilidad de seguridad web que permite a un atacante realizar consultas no deseadas a una base de datos."
date: 2022-10-29
classes: wide
header:
  teaser: /assets/images/htb-writeup-appointment/appointment-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - CTFs
  - Linux
tags:  
  - sqli
---

![](/assets/images/htb-writeup-appointment/appointment-logo.png)

Appointment es una máquina muy fácil que forma parte del Tier 1 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando `SQL Injection`, una vulnerabilidad de seguridad web que permite a un atacante realizar consultas no deseadas a una base de datos.

## Reconocimiento Inicial

```
ping -c 1 10.129.16.212

PING 10.129.16.212 (10.129.16.212) 56(84) bytes of data.
64 bytes from 10.129.16.212: icmp_seq=1 ttl=63 time=327 ms

--- 10.129.16.212 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 326.683/326.683/326.683/0.000 ms
```

## Escaneo de Puertos

```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.16.212 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2022-10-25 17:56 CEST
Initiating SYN Stealth Scan at 17:56
Scanning 10.129.16.212 [65535 ports]
Discovered open port 80/tcp on 10.129.16.212
Completed SYN Stealth Scan at 17:56, 13.73s elapsed (65535 total ports)
Nmap scan report for 10.129.16.212
Host is up, received user-set (0.073s latency).
Scanned at 2022-10-25 17:56:46 CEST for 13s
Not shown: 63566 closed tcp ports (reset), 1968 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.86 seconds
           Raw packets sent: 69066 (3.039MB) | Rcvd: 63612 (2.544MB)
```

**Escaneo inicial de puertos abiertos con Nmap**
- **-p-** Escanea todo el rango de puertos [1-65535]
- **--open** Muestro solo los puertos abiertos
- **-sS** Aplica un TCP SYN Port Scan
- **--min-rate** Indica el número de paquetes por segundo a emitir
- **-vvv** Muestra por pantalla la información descubierta durante el escaneo
- **-n** Indica que no se aplicará resolución DNS durante el escaneo 
- **-Pn** Indica que no se aplicará el protocolo ARP durante el escaneo
- **-oG** Exporta el output del escaneo a un fichero en formato grepeable

![](/assets/images/htb-writeup-appointment/allPorts.png)

```
sudo nmap -sCV -p80 10.129.16.212 -oN targeted
Starting Nmap 7.93 ( https://nmap.org ) at 2022-10-25 18:23 CEST
Nmap scan report for 10.129.16.212
Host is up (0.44s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.15 seconds
```

**Escaneo avanzado de detección de versiones y servicios con Nmap**
- **-p** Escanea los puertos especificados
- **-sV** Activa la detección de versiones y servicios
- **-sC** Lanza scripts básicos de reconocimiento y enumeración
- **-oG** Exporta el output del escaneo a un fichero en formato normal

![](/assets/images/htb-writeup-appointment/targeted.png)

## Detección de vulnerabilidades

Como podemos ver en el escaneo, hemos identificado que el puerto `80/tcp` está en estado abierto, ejecutando la versión `Apache httpd versión 2.4.38` del servicio `HTTP`. HTTP, que significa `Hypertext Transfer Protocol` o `Protocolo de transferencia de hipertexto` en español, es un protocolo de capa de aplicación utilizado para transmitir información a través de archivos HTML en la World Wide Web. Por lo tanto, en este caso, nos estaremos enfrentando a una `página web`.

**Enumeración Web**

Con la herramienta `whatweb` podemos tratarnos de hacer una idea de las tecnlogías que se pueden estar utilizando por detrás del servicio web, como el gestor de contenidos entre otras cosas. En este caso, volvemos a ver la versión del codename `Apache[2.4.38]`, pero no encontramos información más relevante. Alternativamente, podemos utilizar la herramienta `Wappalyzer` dentro del navegador para obtener la misma información.

```
whatweb 10.129.16.212

http://10.129.16.212 [200 OK] Apache[2.4.38], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)], IP[10.129.16.212], JQuery[3.2.1], PasswordField[password], Script, Title[Login]
```

**Página Web**

Si accedemos a la página web con la `ip de la máquina víctima`, podemos ver que necesitamos unas credenciales válidas para poder acceder a nuestro objetivo. En este caso, podemos intentar probar distintas combinaciones como `admin::admin`, `root::root`, `user::user`, `administrator::password`, etc. Como ninguna de las combinaciones ha sido efectiva, podemos probar si nos enfrenatmos ante un `Inyecicción SQL`.

![](/assets/images/htb-writeup-appointment/web.png)


## Explotación de vulnerabilidades

**SQL Injection**

En el caso de que estemos ante una `Inyecicción SQL`, por detrás de esta página web habrá una base de datos utilizando una sentencia de este estilo:

```
SELECT * FROM users WHERE username = ‘$username’ AND password = ‘$password’;
```

Por lo tanto, podemos probar si nos efrenatmos ante una `SQL injection vulnerability allowing login bypass` con el username `admin' or 1=1-- -` y cualquier contraseña:

![](/assets/images/htb-writeup-appointment/sqli.png)

Finalmente, habremos obtenido la `flag`, ya que la sentencia anterior no se encuentra sanitizada porque los carácteres `-- -` permiten omitir la contraseña.

![](/assets/images/htb-writeup-appointment/flag.png)