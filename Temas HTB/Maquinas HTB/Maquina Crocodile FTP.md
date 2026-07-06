![[Pasted image 20260623120912.png]]
- **`-sV`** → detecta la **versión** de los servicios en cada puerto
- **`-sC`** → ejecuta los **scripts por defecto** de Nmap (default scripts)

La respuesta a la pregunta es: **-sC**

---

**¿Qué son los scripts de Nmap?**

Nmap tiene un motor de scripts llamado **NSE (Nmap Scripting Engine)** que puede hacer cosas más avanzadas que un escaneo normal, como:

- Detectar vulnerabilidades conocidas
- Obtener información extra de los servicios (banners, versiones, configuraciones...)
- Intentar logins por defecto

Con `-sC` ejecuta automáticamente los scripts más comunes y seguros. Normalmente se combina con `-sV`:

```
nmap -sC -sV <IP>
```

Eso es el escaneo "estándar" que verás en casi todos los writeups de HTB

![[Pasted image 20260623121134.png]]
![[Pasted image 20260623121200.png]]
![[Pasted image 20260623121639.png]]
![[Pasted image 20260623121416.png]]
![[Pasted image 20260623121548.png]]

![[Pasted image 20260623122344.png]]

![[Pasted image 20260623122651.png]]


### 📝 Resumen — Máquina "Crocodile" (HTB Starting Point, Tier 1)

#### 1. Reconocimiento con Nmap

```
nmap -sC -sV 10.129.59.80
```

Novedad respecto a antes: usaste **`-sC`** además de `-sV`, que ejecuta los **scripts por defecto** de Nmap para obtener más información de cada servicio automáticamente.

Encontraste dos puertos abiertos:

- **Puerto 21** → FTP
- **Puerto 80** → HTTP (servidor web)

#### 2. Conectarse al FTP y descargar credenciales

```
ftp 10.129.59.80
```

Usuario: `anonymous`, contraseña en blanco. Encontraste dos archivos clave:

- `allowed.userlist` → lista de usuarios
- `allowed.userlist.passwd` → lista de contraseñas

Los descargaste con `get` y los leíste con `cat`. El usuario más privilegiado era **admin**.

#### 3. Buscar páginas ocultas con Gobuster

```
gobuster dir -u http://10.129.59.80 -w /usr/share/wordlists/dirb/common.txt -x php
```

La web no tenía el login enlazado visiblemente — había que descubrirlo por fuerza bruta de directorios. Gobuster encontró **login.php**.

#### 4. Login con las credenciales del FTP

Entraste en `http://10.129.59.80/login.php` y usaste las credenciales obtenidas del FTP para autenticarte como **admin**, consiguiendo la flag