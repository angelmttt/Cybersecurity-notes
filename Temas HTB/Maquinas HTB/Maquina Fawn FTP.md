![[Pasted image 20260620150632.png]]
FTP significa File Transfer Protocol (Protocolo de Transferencia de Archivos).

Es un protocolo que se usa para subir y descargar archivos entre dos ordenadores a través de una red. Suele funcionar en el puerto 21, y al igual que Telnet, en su versión clásica no cifra los datos, por lo que usuario y contraseña viajan en texto plano.

![[Pasted image 20260620150747.png]]
Te pregunta qué protocolo seguro, basado en SSH, hace lo mismo que FTP pero cifrando los datos.

La respuesta es: SFTP (Secure File Transfer Protocol / SSH File Transfer Protocol)

![[Pasted image 20260620151154.png]]
![[Pasted image 20260620151316.png]]Esta sí requiere usar la terminal. Vamos a escanear la máquina con Nmap para ver la versión exacta del FTP.

En tu terminal (no la de la VPN) escribe:

```
nmap -sV 10.129.X.X
```

(sustituye por la IP que te haya dado HTB para Fawn)

El parámetro `-sV` le dice a Nmap que detecte la versión de cada servicio que encuentre, que es justo lo que te piden.

![[Pasted image 20260620151807.png]]
![[Pasted image 20260620151746.png]]

![[Pasted image 20260620152206.png]]
![[Pasted image 20260620152150.png]]

![[Pasted image 20260620152424.png]]

### Resumen, Máquina "Fawn" (HTB Starting Point)

#### 1. Concepto: FTP

FTP (File Transfer Protocol) es un protocolo para transferir archivos entre cliente y servidor. Funciona por defecto en el puerto 21. Al igual que Telnet, en su versión clásica no cifra los datos, usuario, contraseña y archivos viajan en texto plano por la red.

Su versión segura es SFTP, que usa el cifrado de SSH para proteger esos datos.

#### 2. Reconocimiento con Nmap

```
nmap -sV 10.129.56.134
```

El resultado mostró:

```
21/tcp open  ftp   vsftpd 3.0.3
Service Info: OS: Unix
```

Esto te dio dos datos clave: el software exacto (vsftpd 3.0.3) y el sistema operativo (Unix).

#### 3. El fallo de seguridad: acceso anónimo

Muchos servidores FTP, sobre todo en entornos antiguos o mal configurados, permiten el acceso anónimo, un usuario genérico llamado `anonymous` que no requiere contraseña real. Esto es una vulnerabilidad clásica porque cualquiera en la red puede entrar sin credenciales.

#### 4. Conectarse al FTP

```
ftp 10.129.56.134
```

- Usuario: `anonymous`
- Contraseña: en blanco (Enter)

El servidor respondió con el código:

```
230 Login successful.
```

Ese 230 es el código estándar de FTP que confirma un login correcto (parecido a un "200 OK" en una web).

#### 5. Listar y descargar archivos

Dentro del modo FTP:

```
ls          # listar archivos disponibles
get flag.txt  # descargar el archivo a tu máquina local
```

#### 6. Leer la flag

Una vez descargado, saliste del modo FTP (`bye`) y leíste el archivo ya en tu terminal normal con `cat flag.txt`.
#### Conclusión

Esta máquina enseña otra vulnerabilidad muy básica pero real: servicios sin cifrado + acceso anónimo habilitado = exposición de archivos sensibles. En una auditoría real, encontrar FTP anónimo accesible desde fuera sería un hallazgo a reportar inmediatamente.