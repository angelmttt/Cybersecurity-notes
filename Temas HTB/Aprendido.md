### Resumen, Lo aprendido hoy 20-06-2026

 Conceptos de conexión

- Cómo conectarte a una VPN con OpenVPN para acceder a una red de prácticas
- Diferencia entre tu propia IP/Pwnbox y la IP del objetivo

 Herramientas

- Nmap (`-sV`) → escanear puertos abiertos y detectar versión de los servicios
- Telnet → conexión remota sin cifrar (puerto 23)
- FTP → transferencia de archivos sin cifrar (puerto 21)

 Vulnerabilidades explotadas

- Meow: usuario `root` con contraseña en blanco vía Telnet → acceso total como administrador
- Fawn: acceso anónimo (`anonymous`) habilitado en un servidor FTP → archivos expuestos sin autenticación real

 Conclusión
Ambas máquinas comparten el mismo fallo de fondo: credenciales débiles o inexistentes en servicios sin cifrado. Es de las vulnerabilidades más básicas pero también de las más comunes en sistemas mal configurados en el mundo real.

 Comandos usados

```
nmap -sV <IP>
telnet <IP>
ftp <IP>
get archivo
cat archivo
```

#### Conceptos de conexión

- Cómo conectarte a una VPN con OpenVPN para acceder a una red de prácticas
- Diferencia entre tu propia IP/Pwnbox y la IP del objetivo
- Uso de `/etc/hosts` para resolver dominios manualmente sin DNS real
- Virtual Hosting → un mismo servidor puede alojar varias webs distintas bajo la misma IP

#### Herramientas

- Nmap (`-sV`, `-sC`, `-p-`, `-p <puerto>`) → escanear puertos, detectar versiones y ejecutar scripts por defecto
- Telnet → conexión remota sin cifrar (puerto 23)
- FTP → transferencia de archivos sin cifrar (puerto 21)
- SMB / smbclient → carpetas compartidas en red, típico de Windows (puerto 445)
- Redis / redis-cli → base de datos clave-valor en memoria (puerto 6379)
- MySQL/MariaDB → bases de datos relacionales (puerto 3306)
- Gobuster (`dir`, `vhost`) → fuerza bruta de directorios web y de subdominios (vhosts)
- Responder → captura credenciales NTLM haciéndose pasar por servidores legítimos en la red
- John the Ripper → crackeo de hashes con wordlists
- Evil-WinRM → acceso remoto a máquinas Windows (puerto 5985)
- AWS CLI → interactuar con servicios de Amazon (S3 y otros)

#### Vulnerabilidades explotadas

- Meow: usuario `root` con contraseña en blanco vía Telnet → acceso total como administrador
- Fawn: acceso anónimo (`anonymous`) habilitado en un servidor FTP → archivos expuestos sin autenticación real
- Dancing: carpeta SMB compartida sin contraseña → acceso a archivos sensibles
- Redeemer: servidor Redis expuesto sin autenticación → lectura libre de datos
- Appointment: SQL Injection en login (`admin'#`) → bypass de autenticación
- Sequel: MySQL root sin contraseña → acceso completo a bases de datos
- Crocodile: credenciales en FTP anónimo + página de login oculta descubierta por fuerza bruta → encadenamiento de vulnerabilidades
- Responder: RFI (Remote File Inclusion) → captura de hash NTLM con Responder → crackeo con John → acceso remoto con Evil-WinRM
- Three: Bucket S3 mal configurado → subida de webshell PHP → RCE (ejecución remota de código)

#### Conclusión

La mayoría de estas máquinas comparten un patrón de fondo: servicios mal configurados, sin autenticación o con credenciales por defecto/vacías. A medida que avanzas, los retos pasan de "un solo fallo evidente" a encadenar varias vulnerabilidades (ej. Crocodile, Responder, Three), la habilidad clave en pentesting real no es solo encontrar un fallo, sino saber combinar varios pasos hasta lograr el objetivo final.

#### Comandos usados

```
nmap -sV -sC -p- <IP>
telnet <IP>
ftp <IP>
smbclient -L <IP>
redis-cli -h <IP>
mysql -h <IP> -u root --skip-ssl
gobuster dir -u <URL> -w <wordlist> -x php
gobuster vhost -u <URL> -w <wordlist> --append-domain
sudo responder -I tun0
john hash.txt --wordlist=<wordlist> --format=netntlmv2
evil-winrm -i <IP> -u <usuario> -p <contraseña>
aws --endpoint=<URL> s3 ls s3://<bucket>
```

