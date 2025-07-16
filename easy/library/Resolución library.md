# Herramientas
- Nmap
- Gobuster
- Hydra
# Procedimiento
Escaneamos los puertos y vemos que están abiertos el puerto 22 y 80. 
![[Pasted image 20250716184251.png]]

Visitamos la página pero es el archivo predeterminado de Apache, por lo que procedemos a utilizar gobuster.
![[Pasted image 20250716184503.png]]

Vemos que hay una directorio llamado `index.php`:
![[Pasted image 20250716184202.png]]

Esto podría ser una contraseña, así que se utiliza hydra para encontrar el usuario de esta contraseña.
Los parámetros son:
- -L: Listado de usuarios
- -p: Una sola contraseña
```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p JIFGHDS87GYDFIGD ssh://172.17.0.3 -t  
20
```

Con esto, logramos encontrar el usuario en cuestión:
![[Pasted image 20250716184444.png]]

Una vez ingresando a la máquina, ocupamos `sudo -l` para ver qué permisos tenemos.
![[Pasted image 20250716185400.png]]

Vemos que hay un script, le hacemos `cat` y...
![[Pasted image 20250716185432.png]]

Hay un código en python que importa una librería... Entonces podríamos crear un archivo en ese mismo directorio que se llame `shutil` para reemplazar esa librería y le pondemos código malicioso (este tipo de ataque se le dice Path Hijacking):
```python
import os
os.system("bash")
```

Con esto, volvemos a nuestro directorio inicial y colocamos lo siguiente:
```bash
sudo /usr/bin/python3 /opt/script.py
```

Y ya con esto, hemos conseguido elevar privilegios en esta máquina :3
![[Pasted image 20250716185649.png]]