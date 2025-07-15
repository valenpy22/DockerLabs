# Herramientas
- Nmap
- Python3
# Procedimientos
Lo primero, como siempre, es realizar el escaneo con nmap. En este caso ocupamos el comando `nmap -p- --script=vuln 172.17.0.2` y podemos ver que existe una vulnerabilidad en el servicio de ftp donde se pude colocar un backdoor.

```
Nmap scan report for 172.17.0.2  
Host is up (0.0000070s latency).  
Not shown: 65533 closed tcp ports (reset)  
PORT   STATE SERVICE  
21/tcp open  ftp  
| ftp-vsftpd-backdoor:    
|   VULNERABLE:  
|   vsFTPd version 2.3.4 backdoor  
|     State: VULNERABLE (Exploitable)  
|     IDs:  BID:48539  CVE:CVE-2011-2523  
|       vsFTPd version 2.3.4 backdoor, this was reported on 2011-07-04.  
|     Disclosure date: 2011-07-03  
|     Exploit results:  
|       Shell command: id  
|       Results: uid=0(root) gid=0(root) groups=0(root)  
|     References:  
|       http://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html  
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-2523  
|       https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/unix/ftp/vsftpd_234_backdoor.rb  
|_      https://www.securityfocus.com/bid/48539  
80/tcp open  http  
|_http-csrf: Couldn't find any CSRF vulnerabilities.  
|_http-dombased-xss: Couldn't find any DOM based XSS.  
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.  
MAC Address: 02:42:AC:11:00:02 (Unknown)  
  
Nmap done: 1 IP address (1 host up) scanned in 58.40 seconds
```

Buscamos en google un exploit para esta versión y especificamos que esté en github (ocupamos el mismo exploit que para la máquina [[Resolución firsthacking]]). 

Usamos el script `python3 exploit.py 172.17.0.2` (ya habiendo activado el entorno virtual), y comprobamos que somos root con el comando `whoami`.

![[Pasted image 20250715172404.png]]

Es decir, hemos completado esta máquina de forma rápida :)