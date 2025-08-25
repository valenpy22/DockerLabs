# Herramientas
# Procedimiento
Lo primero que debemos hacer es escanear los puertos con nmap. Encontramos que hay un puerto 5000.

![[Pasted image 20250825093137.png]]
Lo visitamos y nos encontramos con esto:
![[Pasted image 20250825093222.png]]
Nos vamos a la ruta:
![[Pasted image 20250825093240.png]]

Debemos probar con atributos que podrían pertenecer a users. 
![[Pasted image 20250825093321.png]]

Para ello, ocupamos wfuzz para fuzzear los parámetros.
```
wfuzz -c --hc=404 --hw 169 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt 172.17.0.2:5000/users?  
FUZZ=
```

![[Pasted image 20250825093528.png]]

Nos encontramos con esto:
![[Pasted image 20250825093753.png]]

Con esto, nos metemos a ssh con el usuario pingu y contraseña pinguinasio. Una vez dentro exploramos y encontramos lo siguiente:
![[Pasted image 20250825094848.png]]

Introducimos la contraseña y ya tenemos privilegios de usuario.