# Herramientas
- Nmap
- Hydra
# Procedimiento
Primero, escaneamos los puertos y podemos ver que hay 3 puertos abiertos: 21 (FTP), 22 (SSH) y 80 (HTTP). Además, hay una carpeta llamada `backup` que puede ser interesante de ver. 

![[Pasted image 20250715164933.png]]

Entramos al ftp con el comando `ftp 172.17.0.2`, el nombre de usuario sería `anonymous`y sin contraseña, pudiendo acceder perfectamente. 

Una vez dentro, listamos los archivos existentes con el comando `ls` y vemos lo siguiente:
![[Pasted image 20250715164719.png]]

Para obtener estos archivos solamente debemos ocupar `get <nombre_archivo>` y se guardará en `home` de nuestra máquina.
![[Pasted image 20250715164836.png]]

Podemos ver que russoski está realmente funado... para pensar, señores

Luego, nos dirigimos a la paǵina y ponemos en la URL `/backup/` para ver qué es lo que hay. 
![[Pasted image 20250715165112.png]]

Tenemos este usuario, por lo que procederemos a ocupar hydra con este usuario y... ¡lo conseguimos!
![[Pasted image 20250715165328.png]]
Entramos por ssh con estas credenciales y voilà. Ocupamos el comando `sudo -l` para ver qué podemos hacer, y vemos que vim tiene permisos para ejecutarse como root. Ejecutamos el comando `sudo /usr/bin/vim`, y luego `:!/bin/bash` para acceder al panel del root.
![[Pasted image 20250715165648.png]]

Listo, hemos completado esta máquina :)