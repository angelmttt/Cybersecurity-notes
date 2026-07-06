
## NMAP
**¿Qué es Nmap y qué hace ese comando?**

Nmap es una herramienta que **escanea una máquina para ver qué "puertas" (puertos) tiene abiertas** y qué programas están corriendo detrás de cada una.

Imagina una casa con muchas puertas numeradas (los puertos van del 0 al 65535). Cada puerto suele estar asociado a un servicio:

- Puerto **22** → SSH (acceso remoto por terminal)
- Puerto **80** → página web
- Puerto **21** → FTP (transferencia de archivos)
- Puerto **3306** → MySQL

Nmap te dice cuáles de esas puertas están abiertas en la máquina objetivo, y eso es lo primero que necesitas saber porque **cada puerto abierto es una posible vía de entrada**.

---

Cuando termine el escaneo, vas a ver una lista tipo:

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH ...
```

Eso significa "puerto 22 abierto, corriendo SSH". Pégame el resultado completo cuando termine y te explico qué significa cada línea y qué hacer con ella 👍

**Sobre cuál es "mejor", depende del momento:**

**Escaneo por defecto (`nmap -sV <IP>`)** → Lo normal para empezar. Revisa los 1000 puertos más comunes, que cubren el 95% de los casos en máquinas de prácticas. Es rápido (segundos).

**Escaneo completo (`nmap -p- <IP>`)** → Lo usas como "red de seguridad" si el escaneo normal no encuentra nada, por si el servicio está en un puerto raro fuera del top 1000. Es mucho más lento (puede tardar varios minutos).

**Escaneo a puerto concreto (`nmap -sV -p 6379 <IP>`)** → Lo usas cuando **ya sabes** qué puerto te interesa (como ahora, que ya sabías que Redis suele estar en 6379) y solo quieres confirmar/ver detalles rápido, sin perder tiempo escaneando todo lo demás.

**Flujo recomendado en general:**

1. Empiezas con el escaneo normal
2. Si no sale nada (o sospechas que falta algo), repites o usas `-p-`
3. Una vez sabes el puerto exacto que te interesa, lo apuntas directamente


## SSH

SSH significa **Secure Shell**. Es una herramienta que te permite **conectarte de forma remota a otro ordenador y manejarlo por terminal**, como si estuvieras delante de él pero estando a kilómetros de distancia.

**Para qué se usa:**

- Administrar servidores sin estar físicamente delante
- Conectarte a máquinas en HTB para resolver los retos
- Transferir archivos entre equipos

**Por qué es importante en ciberseguridad:**  
Todo va **cifrado**, así que nadie puede espiar lo que escribes (usuario, contraseña, comandos). Por eso es el sucesor de Telnet, que es lo mismo pero sin cifrar — todo viaja en texto plano y cualquiera en la red puede leerlo.

**Cómo se usa (lo verás mucho):**

```
ssh usuario@10.129.56.121
```

Eso te conecta como ese usuario a esa máquina, te pedirá la contraseña y ya estarás "dentro".

Es básicamente la herramienta número 1 que vas a usar una vez consigas credenciales de una máquina

SSH significa **Secure Shell**. Es una herramienta que te permite **conectarte de forma remota a otro ordenador y manejarlo por terminal**, como si estuvieras delante de él pero estando a kilómetros de distancia.

**Para qué se usa:**

- Administrar servidores sin estar físicamente delante
- Conectarte a máquinas en HTB para resolver los retos
- Transferir archivos entre equipos

**Por qué es importante en ciberseguridad:**  
Todo va **cifrado**, así que nadie puede espiar lo que escribes (usuario, contraseña, comandos). Por eso es el sucesor de Telnet, que es lo mismo pero sin cifrar — todo viaja en texto plano y cualquiera en la red puede leerlo.

**Cómo se usa (lo verás mucho):**

```
ssh usuario@10.129.56.121
```

Eso te conecta como ese usuario a esa máquina, te pedirá la contraseña y ya estarás "dentro".

Es básicamente la herramienta número 1 que vas a usar una vez consigas credenciales de una máquina

## SSH y SFTP
**SSH (Secure Shell)** es el protocolo "padre" — te permite conectarte a otra máquina y **ejecutar comandos** de forma remota y cifrada, como si tuvieras una terminal abierta en ese otro ordenador.

**SFTP (SSH File Transfer Protocol)** es como una "extensión" o "modo" de SSH, pero en lugar de servir para ejecutar comandos, sirve específicamente para **transferir archivos** de forma cifrada. Usa el mismo canal seguro que SSH, simplemente con otro propósito.

Es como si SSH fuera la "tubería segura" entre dos ordenadores, y dentro de esa tubería puedes:

- Ejecutar comandos → eso es SSH normal
- Mover archivos → eso es SFTP

Por eso la pregunta decía "una extensión del protocolo SSH" — porque SFTP literalmente se monta encima de SSH para añadir esa función extra de mover archivos de forma segura

## FTP
#### ¿Qué es?

FTP es uno de los protocolos más antiguos de Internet (de los años 70), diseñado específicamente para **transferir archivos entre un cliente y un servidor**. Sigue siendo muy usado hoy en día en entornos de administración de sistemas, aunque ha quedado obsoleto en términos de seguridad.

#### Dos conexiones, no una

A diferencia de la mayoría de protocolos (HTTP, SSH...), FTP usa **dos canales separados**:

- **Canal de control (puerto 21)** → se usa para enviar comandos (login, listar archivos, pedir descargas...)
- **Canal de datos** → se usa exclusivamente para transferir el contenido real de los archivos

Esto es importante porque puede dar problemas con firewalls (al abrir dos conexiones en vez de una).

#### Modo Activo vs Modo Pasivo

- **Activo:** el servidor abre una conexión de vuelta hacia el cliente para enviar los datos (puerto 20). Suele dar problemas si el cliente está detrás de un firewall/NAT.
- **Pasivo (PASV):** el cliente es quien inicia ambas conexiones. Es el modo más usado hoy en día porque evita problemas de firewall. En tu captura viste justo esto: `229 Entering Extended Passive Mode`.

#### Autenticación

FTP soporta dos formas de acceso:

- **Con cuenta real** → usuario y contraseña del sistema
- **Anónimo** → usuario `anonymous`, sin contraseña real (a veces pide un email cualquiera como "contraseña" simbólica)

#### Por qué es inseguro

- **Sin cifrado** → usuario, contraseña y archivos viajan en **texto plano**. Cualquiera que intercepte el tráfico de red (sniffing) puede leerlo todo.
- **Acceso anónimo mal configurado** → muchos administradores dejan acceso anónimo habilitado "por comodidad" y exponen archivos sensibles sin querer.

#### Versiones seguras

|Protocolo|Qué es|
|---|---|
|**FTPS**|FTP + cifrado SSL/TLS|
|**SFTP**|Transferencia de archivos sobre SSH (no es FTP realmente, es otro protocolo distinto que cumple la misma función)|

#### Comandos básicos del cliente FTP

|Comando|Qué hace|
|---|---|
|`ls` / `dir`|Listar archivos del servidor|
|`get archivo`|Descargar un archivo|
|`put archivo`|Subir un archivo|
|`cd carpeta`|Cambiar de directorio remoto|
|`pwd`|Ver directorio actual|
|`bye` / `quit`|Salir del cliente FTP|

#### Códigos de respuesta más comunes

|Código|Significado|
|---|---|
|220|Servicio listo|
|230|Login correcto|
|331|Usuario válido, pide contraseña|
|530|Login incorrecto|
|550|Archivo no encontrado / sin permisos|

## SMB 
#### ¿Qué es?

SMB es un protocolo creado por Microsoft para que ordenadores en una misma red puedan **compartir archivos, carpetas e impresoras** entre sí. Es el corazón de cómo funcionan las redes de oficina tradicionales en Windows ("la carpeta compartida del departamento", "la impresora de la planta 2"...).

#### Puerto y versiones

- Funciona por defecto en el **puerto 445** (versiones modernas) o 139 (versiones antiguas, sobre NetBIOS)
- Ha tenido varias versiones: SMB1 (muy vulnerable, casi en desuso), SMB2, SMB3 (la más segura y usada hoy)

#### Por qué es tan relevante en ciberseguridad

- Es uno de los **protocolos más atacados en entornos corporativos** porque casi todas las empresas con Windows lo usan internamente
- Históricamente ha tenido vulnerabilidades muy graves — la más famosa es **EternalBlue (MS17-010)**, el exploit que usó el ransomware **WannaCry** en 2017 para propagarse por medio mundo
- Muchas veces los administradores dejan **shares (carpetas compartidas) sin contraseña** "por comodidad interna", exponiendo documentos sensibles a cualquiera que esté en la red, como acabas de ver

#### Herramientas típicas para auditar SMB

|Herramienta|Para qué sirve|
|---|---|
|`smbclient`|Conectarte interactivamente a un share, como acabas de hacer|
|`smbmap`|Ver de un vistazo qué permisos tienes en cada share|
|`enum4linux`|Enumeración más completa (usuarios, grupos, política de contraseñas...)|
|`crackmapexec` / `netexec`|Atacar/probar credenciales contra varias máquinas SMB a la vez|

#### Comandos básicos dentro de smbclient

| Comando       | Qué hace                 |
| ------------- | ------------------------ |
| `ls`          | Listar archivos/carpetas |
| `cd carpeta`  | Cambiar de directorio    |
| `get archivo` | Descargar archivo        |
| `put archivo` | Subir archivo            |
| `exit`        | Salir                    |

## Redis

#### ¿Qué es?

**Redis** es una base de datos **NoSQL de tipo clave-valor**, que funciona **en memoria RAM** (no en disco como MySQL), lo que la hace extremadamente rápida. Se usa principalmente como:

- **Caché** — guardar datos que se consultan muy a menudo para no tener que ir cada vez a la base de datos "de verdad"
- **Colas de mensajes** — comunicación entre distintas partes de una aplicación
- **Sesiones de usuario** — guardar temporalmente quién está logueado en una web

#### Cómo se organiza

A diferencia de MySQL (tablas con filas y columnas), Redis guarda todo como pares **clave → valor**, parecido a un diccionario:

```
nombre → "Ángel"
edad → "20"
```

Además tiene varias **bases de datos numeradas** (por defecto del 0 al 15) dentro de la misma instancia, por eso usaste `SELECT 0`.

#### Por qué es inseguro por defecto

- **Sin autenticación de fábrica** → cualquiera que llegue al puerto 6379 puede leer y escribir datos libremente, como acabas de comprobar
- Está pensado para usarse **solo dentro de redes internas de confianza**, nunca expuesto directamente a Internet — pero muchos administradores se olvidan de esto y lo dejan accesible

#### Comandos básicos

|Comando|Qué hace|
|---|---|
|`PING`|Comprobar conexión (responde `PONG`)|
|`INFO`|Ver información del servidor|
|`SELECT n`|Cambiar de base de datos|
|`KEYS *`|Listar todas las claves|
|`GET clave`|Ver el valor de una clave|
|`SET clave valor`|Crear/modificar una clave|
|`DBSIZE`|Contar cuántas claves hay|
|`FLUSHALL`|Borrar todo (¡cuidado con este en producción!)|

#### Relevancia en ciberseguridad

Encontrar un Redis expuesto sin autenticación es un **hallazgo crítico** en una auditoría real — más allá de leer datos, en muchos casos se puede usar para **ejecutar código en el servidor** (escribiendo claves SSH maliciosas, por ejemplo), lo cual ya es una vía directa de compromiso total del sistema.

## SQL INJECTION
#### Qué es?

SQL Injection (inyección SQL) es una vulnerabilidad que ocurre cuando una aplicación web **construye consultas SQL usando directamente lo que escribe el usuario**, sin limpiarlo ni validarlo antes. Esto permite a un atacante "inyectar" código SQL propio dentro de la consulta original, alterando su comportamiento.

#### Cómo funciona en este caso (bypass de autenticación)

La aplicación construye internamente algo así:

sql

```sql
SELECT * FROM users WHERE username='INPUT' AND password='INPUT'
```

Si metes `admin'#` como usuario, la consulta se convierte en:

sql

```sql
SELECT * FROM users WHERE username='admin'#' AND password='cualquier_cosa'
```

El símbolo `#` (en MySQL) le dice a la base de datos: **"ignora todo lo que viene después de aquí"**. Por tanto, la comprobación de la contraseña desaparece por completo, y la consulta solo verifica que exista un usuario llamado `admin` — que sí existe, así que el login se acepta.

#### Símbolos de comentario según el motor de base de datos

|Motor|Comentario|
|---|---|
|MySQL|`#` o `--` (con espacio)|
|PostgreSQL / SQL Server|`--`|
|Oracle|`--`|

#### Por qué es tan peligrosa

No se queda solo en bypass de login — con SQL Injection un atacante puede llegar a:

- Leer **toda la base de datos** (contraseñas, datos personales, tarjetas...)
- Modificar o borrar información
- En casos avanzados, incluso **ejecutar comandos en el servidor**

#### Cómo se previene (importante para tus apuntes de cara a auditorías)

- **Consultas parametrizadas / prepared statements** → la forma correcta y estándar de programar consultas, separando el código SQL de los datos del usuario
- **Validación y sanitización de entradas** → filtrar caracteres peligrosos como comillas
- **Principio de mínimo privilegio** → que el usuario de la base de datos que usa la web no tenga más permisos de los necesarios
- **WAF (Web Application Firewall)** → como capa extra de defensa, aunque no sustituye a una buena programación

## MySql
#### ¿Qué es MySQL?

MySQL es el **sistema de gestión de bases de datos relacionales (RDBMS)** más usado del mundo. Guarda información en **tablas** con filas y columnas, y se consulta con el lenguaje **SQL**.

**MariaDB** es un fork (copia) open source de MySQL creada por la comunidad cuando Oracle compró MySQL en 2010. Es prácticamente idéntica en uso pero con mejoras de rendimiento y totalmente gratuita. Hoy en día muchos servidores Linux usan MariaDB en lugar de MySQL aunque lo llamen igual.

#### Puerto y uso

- Puerto por defecto: **3306**
- Usado en prácticamente toda aplicación web con base de datos (WordPress, Joomla, tiendas online...)
- Las bases de datos que siempre existen en cualquier instalación MySQL/MariaDB son:

|Base de datos|Para qué sirve|
|---|---|
|`information_schema`|Metadatos del servidor (qué tablas, columnas, usuarios existen...)|
|`mysql`|Configuración interna y usuarios del propio MySQL|
|`performance_schema`|Estadísticas de rendimiento del servidor|

#### Por qué es relevante en ciberseguridad

- Un MySQL expuesto en el puerto 3306 **sin contraseña** (como esta máquina) es un hallazgo crítico en auditorías reales
- Acceder a MySQL da acceso a **todos los datos** de todas las aplicaciones que usen esa base de datos — contraseñas, datos personales, información de negocio...
- Es habitual encontrar MySQL expuesto en entornos mal configurados o en pruebas de desarrollo que nadie cerró

#### Comandos básicos SQL que ya conoces (repaso)

|Comando|Qué hace|
|---|---|
|`SHOW DATABASES;`|Listar todas las bases de datos|
|`USE nombre;`|Entrar en una base de datos|
|`SHOW TABLES;`|Listar tablas de la base de datos actual|
|`DESCRIBE tabla;`|Ver la estructura de una tabla (columnas, tipos...)|
|`SELECT * FROM tabla;`|Ver todo el contenido de una tabla|
|`SELECT columna FROM tabla;`|Ver solo una columna concreta|

#### Diferencia entre MySQL y lo que viste en la uni

En la uni usaste MySQL haciendo consultas desde una interfaz gráfica (phpMyAdmin, MySQL Workbench...). Aquí lo haces desde la **línea de comandos**, que es exactamente cómo se hace en auditorías reales cuando no tienes acceso a ninguna interfaz gráfica.

## Gobuster y Directory Brute Force

**Gobuster** es una herramienta que prueba miles de nombres de directorios y archivos contra un servidor web para descubrir contenido oculto que no está enlazado públicamente.

Los switches más importantes:

|Switch|Para qué sirve|
|---|---|
|`dir`|Modo búsqueda de directorios|
|`-u`|URL objetivo|
|`-w`|Wordlist (lista de nombres a probar)|
|`-x`|Extensiones de archivo a buscar (php, html...)|

#### Wordlists

Son archivos de texto con miles de nombres comunes de directorios y archivos. En Parrot OS ya vienen preinstaladas en `/usr/share/wordlists/`. La más básica y usada para empezar es `dirb/common.txt`.

#### Encadenamiento de vulnerabilidades

Esta es la lección más importante de Crocodile — por primera vez **encadenaste dos servicios**:

```
FTP (credenciales) → Web (login)
```

En pentesting real esto es muy habitual: lo que encuentras en un sitio (credenciales, archivos, información) te sirve para atacar otro. Rara vez un ataque real es de un solo paso.

#### Nmap -sC (NSE Scripts)

El **Nmap Scripting Engine (NSE)** permite a Nmap ir más allá del escaneo básico y ejecutar scripts automatizados que obtienen información extra. Con `-sC` usas los scripts "safe" por defecto. Combinado con `-sV` es el escaneo estándar en cualquier auditoría:

```
nmap -sC -sV <IP>
```

## LFI, RFI, NTLM, John the Ripper y Evil-WinRM

LFI vs RFI
![[Pasted image 20260630181309.png]]

Ambas ocurren cuando una web usa directamente la entrada del usuario en funciones como `include()` de PHP sin validarla.

#### NTLM y autenticación de Windows

**NTLM (New Technology LAN Manager)** es el protocolo de autenticación de Windows. Funciona por **desafío-respuesta**: en vez de enviar la contraseña por la red, el sistema calcula un valor (hash) combinando la contraseña con un "reto" aleatorio, y solo ese resultado viaja por la red.

Esto significa que la contraseña real **nunca se transmite**, pero el hash capturado puede ser **crackeado offline** probando millones de contraseñas hasta encontrar cuál genera ese mismo resultado.

#### Responder

Herramienta que **se hace pasar por servidores legítimos** (SMB, HTTP, DNS...) en una red para capturar credenciales cuando una máquina intenta autenticarse contra ella de forma automática (como pasa con las rutas de red de Windows).

#### John the Ripper — formato netntlmv2

Cada tipo de hash tiene un formato distinto, por eso a veces hay que especificarlo manualmente con `--format` si John no lo detecta bien automáticamente:

```
john hash.txt --wordlist=<wordlist> --format=netntlmv2
```

#### Evil-WinRM

Cliente de Linux para conectarte remotamente a máquinas Windows a través del protocolo **WinRM (puerto 5985)**, usando usuario y contraseña válidos. Es el equivalente de `ssh` pero para entornos Windows.

```
evil-winrm -i <IP> -u <usuario> -p <contraseña>
```

---

#### 💡 La lección de fondo

Esta es la máquina más completa que has hecho hasta ahora — combina **vulnerabilidad web (RFI) + ataque de red (captura NTLM) + crackeo de contraseñas + acceso remoto**. Es un ejemplo perfecto de cómo en pentesting real rara vez hay un solo paso: encadenas varias técnicas hasta lograr acceso completo al sistema.



## AWS
#### Amazon S3 (Simple Storage Service)

S3 es el servicio de **almacenamiento de objetos en la nube** de AWS, uno de los servicios más usados de Amazon. Permite guardar cualquier tipo de archivo (imágenes, documentos, backups, código...) en contenedores llamados **buckets**.

**Conceptos clave:**

- **Bucket** → el "contenedor" donde se guardan los archivos, identificado por un nombre único
- **Objeto** → cada archivo individual dentro de un bucket
- Las webs y aplicaciones suelen usar S3 para servir contenido estático (imágenes, vídeos, archivos descargables) en lugar de guardarlo en su propio servidor, porque es más barato y escalable

#### Virtual Hosting y subdominios

Un mismo servidor puede alojar **varias webs distintas bajo la misma IP**, diferenciándolas por el nombre de dominio que se usa en la petición. Esto se llama **Virtual Hosting**.

Para descubrir estos subdominios "ocultos" se usa:

```
gobuster vhost -u http://dominio -w <wordlist> --append-domain
```

A diferencia del modo `dir` (busca carpetas/archivos) o `dns` (necesita un servidor DNS real), el modo `vhost` prueba distintos nombres de host contra el mismo servidor web directamente.

#### AWS CLI

Herramienta oficial de línea de comandos de Amazon para interactuar con cualquier servicio de AWS, incluido S3.

| Comando                         | Qué hace                                                                                                  |
| ------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `aws configure`                 | Configura credenciales (obligatorio antes de usar cualquier comando)                                      |
| `aws s3 ls s3://bucket`         | Lista el contenido de un bucket                                                                           |
| `aws s3 cp archivo s3://bucket` | Sube un archivo al bucket                                                                                 |
| `--endpoint=URL`                | Apunta la herramienta a un servidor distinto al de Amazon real (clave para usarla contra máquinas de HTB) |

#### Webshell

Un **webshell** es un archivo de código (normalmente PHP, ASP o JSP) que, una vez subido a un servidor vulnerable, permite **ejecutar comandos del sistema operativo a través del navegador**. Es una de las herramientas más usadas tras conseguir subir archivos a un servidor.

Ejemplo mínimo en PHP:

php

```php
<?php system($_GET['cmd']); ?>
```

#### RCE (Remote Code Execution)

**Ejecución remota de código** — el nivel de compromiso más alto que se puede lograr en una máquina. Significa que puedes ejecutar **cualquier comando** en el sistema objetivo como si tuvieras una terminal abierta ahí mismo, sin necesidad de tener acceso físico ni credenciales de login tradicionales.

#### Cadena de explotación de esta máquina (encadenamiento)

```
Web con subdominio oculto (vhost)
        ↓
Subdominio resulta ser un bucket S3 mal configurado
        ↓
El bucket permite subida de archivos sin autenticación real
        ↓
Subes un webshell PHP al bucket
        ↓
El bucket está sincronizado con la carpeta web del servidor
        ↓
Tu webshell queda accesible públicamente
        ↓
Ejecutas comandos = RCE = control total del servidor
```

---

#### 💡 Relevancia en el mundo real

Los buckets de S3 mal configurados son una de las **causas más comunes de filtraciones de datos masivas** en empresas reales — múltiples brechas de seguridad conocidas (incluyendo grandes empresas) han ocurrido por dejar buckets S3 con permisos de lectura/escritura públicos sin querer. Por eso esta máquina, aunque parezca un ejercicio, refleja un fallo real y frecuente en entornos cloud empresariales.

## SQLi
### 1. SQL Injection (SQLi)

**Qué es**: una vulnerabilidad que ocurre cuando una aplicación web mete directamente lo que escribe el usuario dentro de una consulta SQL, sin filtrarlo ni validarlo. Esto permite que un atacante "rompa" la lógica de esa consulta e inserte código SQL propio.

**Por qué pasa aquí**: el parámetro `search=` de `dashboard.php` construye la consulta concatenando directamente tu texto: `Select * from cars where name ilike '%TU_TEXTO%'`. Si tu texto incluye comillas o palabras clave SQL, puedes alterar completamente la consulta.

**Tipos de técnica de SQLi** (las dos que nos afectaron):

- **UNION-based**: combinas tu propia consulta con la original para leer datos adicionales de la base de datos
- **Stacked queries**: encadenas múltiples sentencias SQL separadas por `;`, lo que en bases de datos mal configuradas permite ejecutar comandos del sistema operativo, no solo leer datos

### 2. sqlmap

Herramienta que automatiza la detección y explotación de SQLi. Prueba automáticamente docenas de payloads y técnicas para encontrar la que funciona. La función `--os-shell` es una de sus capacidades más potentes: si detecta stacked queries, puede convertir la inyección SQL en una vía para ejecutar comandos arbitrarios en el servidor.

### 3. HTTP, sesiones y cookies

HTTP es **stateless** — cada petición es independiente. Las **cookies de sesión** (`PHPSESSID`) son el mecanismo que usan las aplicaciones web para "recordar" que ya te autenticaste, sin tener que volver a pedir usuario/contraseña en cada página. Cuando una herramienta externa (como sqlmap) necesita interactuar con una página protegida, hay que pasarle esa misma cookie manualmente para que el servidor la trate como una sesión ya autenticada.

### 4. Reverse shells

Una **shell** es simplemente un intérprete de comandos. Una **reverse shell** es cuando la máquina víctima inicia la conexión hacia ti (en vez de tú conectarte a ella), típicamente porque hay un firewall que bloquea conexiones entrantes pero no salientes. Tú te pones en modo "escucha" (`nc -lvnp`) y la víctima "te llama" ejecutando un comando que redirige su entrada/salida de shell hacia esa conexión de red.

### 5. TTY y PTY

**TTY** (teletype) es el concepto histórico de "terminal" — el mecanismo que gestiona la interacción en tiempo real entre un usuario y un programa (entrada de teclado, salida en pantalla, control de cursor, señales como Ctrl+C). Muchos programas interactivos (como `vi`, `top`, `sudo` pidiendo contraseña) necesitan una tty real para funcionar.

Una reverse shell básica no tiene tty. **PTY** (pseudo-terminal) es una tty simulada por software que el sistema operativo trata como si fuera real. El truco `python3 -c 'import pty; pty.spawn("/bin/bash")'` crea una de estas pseudo-terminales para que tu shell remota se comporte como una terminal completa.

### 6. Credenciales hardcodeadas

Un fallo de seguridad extremadamente común en el mundo real: desarrolladores que escriben contraseñas directamente en el código fuente (en vez de usar variables de entorno o gestores de secretos). Cualquiera con acceso de lectura al código puede extraerlas. Además, la **reutilización de contraseñas** (la misma contraseña sirve para la base de datos y para el usuario del sistema) multiplica el impacto de este fallo.

### 7. sudo y sus reglas (`/etc/sudoers`)

`sudo` permite a un usuario ejecutar comandos específicos como otro usuario (normalmente root), según reglas definidas por el administrador. Estas reglas pueden ser tan amplias como "puede ejecutar cualquier cosa" o tan estrechas como "puede ejecutar este binario concreto, con este argumento exacto, y nada más". `sudo -l` te muestra qué reglas aplican a tu usuario.

### 8. GTFOBins — el concepto detrás de la escalada final

Lo que hicimos con `vi` (`:!/bin/bash`) es un ejemplo del concepto conocido en la comunidad de pentesting como **"living off the land"** o técnicas de **GTFOBins**: aprovechar funcionalidades legítimas de programas del sistema (editores, paginadores, intérpretes) para escapar a una shell con los privilegios del programa que las ejecuta. Muchísimos binarios de Unix (`vi`, `less`, `find`, `python`, `awk`...) tienen esta capacidad de "ejecutar comandos externos" desde dentro de sí mismos — y cuando ese binario corre con privilegios elevados (root, vía sudo), el escape hereda esos mismos privilegios. Existe incluso una web (**gtfobins.github.io**) que documenta, binario por binario, cómo abusar de cada uno si tienes permisos de sudo sobre él — vale la pena que la explores, te vas a encontrar esta técnica muchísimas veces en HTB.



## PATH hijacking

### 1. Enumeración de código fuente / comentarios ocultos

Mirar el HTML/JS de una página puede revelar rutas, endpoints o pistas que no están enlazadas visualmente. Complementa (no sustituye) a herramientas como gobuster.

### 2. Broken Access Control

Categoría muy amplia de vulnerabilidades donde el sistema no verifica correctamente si un usuario tiene permiso para hacer algo o ver algo. Aquí se manifestó de dos formas concretas:

- **Manipulación de cookies de rol**: el servidor confiaba ciegamente en cookies (`role`, `user`) que el cliente puede modificar libremente, en vez de validar el rol real contra una sesión segura en el servidor.
- **IDOR (Insecure Direct Object Reference)**: usar un identificador visible y predecible (como `id=2` en una URL) para acceder a recursos de otros usuarios sin comprobación de permisos.

Ambos son de las vulnerabilidades más comunes en aplicaciones web reales, catalogadas en el **OWASP Top 10** bajo "Broken Access Control".

### 3. Subida insegura de archivos (Unrestricted File Upload)

Cuando una aplicación permite subir archivos sin validar su tipo real (extensión, contenido, tipo MIME), un atacante puede subir código ejecutable (como un `.php`) camuflado como si fuera un archivo inofensivo (una imagen). Si el servidor luego ejecuta ese archivo al visitarlo (porque Apache/PHP interpreta cualquier `.php` que le llegue), se convierte en ejecución remota de código.

### 4. Reverse shell en PHP

Mismo concepto que la reverse shell en bash de Vaccine, pero implementada en un lenguaje distinto porque el vector de entrada era distinto: aquí subíamos un archivo que el servidor iba a ejecutar como PHP, no teníamos ejecución directa de comandos de sistema como con sqlmap.

### 5. Credenciales hardcodeadas + reutilización de contraseñas

Igual que en Vaccine: escribir contraseñas directamente en el código fuente es un fallo de seguridad grave, y se agrava cuando esa misma contraseña se reutiliza para cuentas de sistema operativo, permitiendo movimiento lateral entre usuarios.

### 6. SUID (Set owner User ID)

Bit especial de permisos en sistemas Unix/Linux que hace que un programa se ejecute siempre con los privilegios de su **propietario** (frecuentemente root), sin importar qué usuario lo lance. Es legítimo y necesario para muchos programas del sistema (como `passwd`), pero es peligroso si el binario que lo tiene activado hace algo que un atacante pueda manipular.

### 7. PATH hijacking (secuestro de variable de entorno)

Cuando un programa llama a otro comando **sin especificar su ruta absoluta** (por ejemplo, `cat` en vez de `/bin/cat`), el sistema operativo depende de la variable de entorno **`$PATH`** para decidir dónde buscar ese comando, y lo busca en el orden en que las carpetas aparecen listadas ahí.

Si un atacante puede:

1. Crear su propio ejecutable con el mismo nombre que el comando esperado
2. Modificar su `$PATH` para que su carpeta se revise primero

...puede conseguir que el programa vulnerable ejecute su propio código en lugar del comando legítimo. Si ese programa vulnerable tiene privilegios elevados (por SUID), el código del atacante también se ejecuta con esos privilegios elevados.

Esta es una técnica **distinta pero relacionada** con el escape de `vi` en Vaccine (GTFOBins): en Vaccine, abusábamos de una funcionalidad _interna_ del programa (el `:!` de vi). Aquí, abusamos de cómo el programa busca _externamente_ otro programa, manipulando el entorno del sistema (`$PATH`) en el que se ejecuta.

### 8. `strings` como herramienta de análisis de binarios

Comando que extrae todas las cadenas de texto legibles dentro de un archivo binario (que por lo demás es código máquina, ilegible directamente). Muy útil para hacer un primer análisis rápido de qué hace un programa sin necesidad de ingeniería inversa completa — a veces revela comandos que ejecuta, mensajes de error, rutas de archivos, contraseñas olvidadas, etc.