# Herramientas
- Nmap
- Hydra
- GTFOBins
# Procedimiento
Como lo solemos hacer, escaneamos los puertos con nmap y podemos ver que hay 2 puertos abiertos: el 22 y el 80. Por lo que procederemos a visitar la página.
![[Pasted image 20250715173127.png]]

Podemos sacar conclusiones de que `a` es el usuario, por lo que se procede a hacer una fuerza bruta de este usuario y encontramos que la contraseña es `secret`.

Al ingresar con las credenciales y colocar `sudo -l`, nos percatamos que no se puede correr `sudo`. Hacemos un simple `cd ../` y podemos notar que hay otro usuario llamado `spencer`. Hacemos otro ataque de fuerza bruta con este usuario y la contraseña es `password1`. Accedemos a la máquina con este último usuario, corremos `sudo -l` y vemos que `/usr/bin/python3`, buscamos en GTFOBins python y nos vamos a la parte de sudo.
```
sudo python -c 'import os; os.system("/bin/sh")'
```

Pero el comando nos dice que es python3, solamente debemos agregar un 3 después de python y nos servirá el comando.

```
sudo python3 -c 'import os; os.system("/bin/sh")'
```

Una vez hecho esto, comprobamos con  `whoami` y comprobamos que tenemos acceso root.

Con esto, damos por finalizada esta máquina :)