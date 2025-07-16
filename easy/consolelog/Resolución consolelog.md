# Herramientas
# Procedimiento
Escaneamos los puertos con nmap y vemos que hay 3 abiertos: el 80, 3000 y 5000.
![[Pasted image 20250716192233.png]]

Revisamos la página y vemos tan solo un botón
![[Pasted image 20250716192406.png]]

Empleamos la herramienta de gobuster con el siguiente comando:
```
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://172.1  
7.0.2 -x .php,.py,.txt,.html
```
![[Pasted image 20250716192502.png]]

Y vemos que hay una ruta de backend y javascript. Exploramos el primer directorio y en el archivo `server.js` está lo siguiente:
![[Pasted image 20250716192550.png]]

Vemos una contraseña y la ocupamos con hydra para hacer fuerza bruta, debemos hallar el usuario por lo que debemos ejecutar el siguiente comando:
```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p lapassworddebackupmaschingonadetodas  
ssh://172.17.0.2:5000 -s 22 -t 64
```
![[Pasted image 20250716192851.png]]

Esperamos un rato y... ¡voilà! Tenemos el usuario, por lo que entramos por ssh con el siguiente comando (recordar que no hay puerto 22, pero ocupamos el puerto 5000):
```
ssh lovely@172.17.0.2 -p 5000
```

Al ingresar usamos `sudo -l` y vemos que se puede ocupar `/usr/bin/nano`. Para escalar privilegios con esto, debemos hacer lo siguiente:
![[Pasted image 20250716193237.png]]

PERO esto no nos servirá, tremenda F. Así que usando pensamiento lateral podemos decir: wait, pero podemos ocupar nano en modo root... y tenemos acceso a /etc/passwd! (Donde estan los usuarios del sistema!) así que vamos a cambiar eso :)
Borramos la `x` que había acá (ya la borré, sorry):
![[Pasted image 20250716194110.png]]

Y así podremos acceder como root sin necesidad de credenciales lol.
![[Pasted image 20250716194151.png]]

Con esto, ya hemos podido escalar privilegios en esta máquina lol