# Herramientas
# Procedimiento
Escaneamos con nmap y solamente encontramos un puerto abierto, el 80. 
![[Pasted image 20250716200350.png]]

Vemos que se listan los directorios y nos vamos a las más interesantes: administrator y robots.txt
![[Pasted image 20250716200442.png]]

En robots encontramos esto, podemos ocupar estas credenciales más adelante.

Mientras que en administrator vemos esto:
![[Pasted image 20250716200518.png]]

Ocupamos las credenciales de robots :) pero...
![[Pasted image 20250716200550.png]]

No funciona, pero notamos que la contraseña es muy rara, por lo que podría estar encriptada... Así que vamos a desencriptarla con el siguiente comando:
```
echo 'c2FubHVpczEyMzQ1' | base64 --decode

sanluis12345
```

Efectivamente, ¡estaba encriptada! Así que ahora sí entramos con estas credenciales...