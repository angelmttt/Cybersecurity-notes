### Preguntas
![[Pasted image 20260621131538.png]]
La respuesta es: A03:2021 – Injection

(El OWASP Top 10 es una lista de las 10 vulnerabilidades más críticas en aplicaciones web. La inyección SQL entra dentro de la categoría general "Injection", que ocupa el puesto A03 en la edición de 2021)

![[Pasted image 20260621131756.png]]
![[Pasted image 20260621131816.png]]
La respuesta es: 443

(El puerto 80 es para HTTP normal sin cifrar, y el 443 es su versión segura con cifrado TLS/SSL, HTTPS)

![[Pasted image 20260621132115.png]]
Gobuster funciona con distintos "modos" según lo que quieras buscar, y se específica como primer argumento después de `gobuster`:

```
gobuster dir -u <URL> -w <wordlist>
```

- `dir` → modo para buscar directorios/archivos
- `dns` → modo para buscar subdominios
- `vhost` → modo para buscar virtual hosts

Como en esta máquina buscas carpetas escondidas en el servidor web, usas el modo `dir`

![[Pasted image 20260621133534.png]]

En SQL, el símbolo `#` (o `--` en algunas bases de datos) significa "todo lo que viene después de aquí en esta línea es un comentario, ignóralo", exactamente igual que el `#` en Python o Bash para comentar una línea de código.

Ahora, mira la consulta completa que se construye cuando metes `admin'#` en el campo de usuario (dejando el campo de contraseña vacío

![[Pasted image 20260621133550.png]]![[Pasted image 20260621133455.png]]

Respuesta a la pregunta de la primera palabra: `Congratulations`

Con el acceso como `admin` se obtiene la flag de la máquina (no se pública).

_"Si la entrada del usuario no se gestiona con cuidado, podría interpretarse como un comentario. Usa un comentario para iniciar sesión como admin sin saber la contraseña. ¿Cuál es la primera palabra que aparece en la página devuelta?"_
Explicación de lo que está pasando:

Cuando una web hace login contra una base de datos, normalmente construye una consulta SQL así por detrás:

sql

```sql
SELECT * FROM usuarios WHERE username = 'lo_que_escribas' AND password = 'lo_que_escribas'
```

El problema es que si la web no filtra correctamente lo que escribes, puedes "romper" esa consulta metiendo caracteres especiales de SQL.

El truco del comentario:
En SQL, los símbolos `--` (dos guiones) o `#` significan "todo lo que viene después es un comentario, ignóralo".

Entonces, si en el campo de usuario escribes:

```
admin'--
```

La consulta que se ejecuta realmente queda así:

sql

```sql
SELECT * FROM usuarios WHERE username = 'admin'--' AND password = 'lo_que_sea'
```

Como todo después de `--` se ignora, la base de datos solo comprueba si existe el usuario `admin`, sin comprobar la contraseña en absoluto. Por eso entras sin saberla.

|Base de datos|Símbolo de comentario|
|---|---|
|MySQL|`#` o `--` (con espacio después)|
|PostgreSQL / SQL Server|`--`|
|Oracle|`--`|

Para esta máquina (Appointment), que usa MySQL, lo correcto es:

```
admin'#
```

Nota importante para tus apuntes: en MySQL, si usas `--` en vez de `#`, tiene que llevar un espacio después para funcionar correctamente:

```
admin'--
```

(fíjate en el espacio después de los guiones, es obligatorio en MySQL)

Por eso normalmente en MySQL se prefiere usar `#`, porque es más simple y no tienes ese problema de "se me olvida el espacio".

Resumen para apuntar:

- MySQL → `'#` (más simple y fiable)
- Otros motores → `'--` (con espacio si es MySQL)

### Resumen, Máquina "Appointment" (HTB Starting Point, Tier 1)

#### 1. Reconocimiento con Nmap

Escaneaste la máquina y encontraste el puerto 80 (HTTP) abierto, un servidor web.

#### 2. Acceso al servicio web

A diferencia de Telnet/FTP/SMB (que se manejan por terminal), un servicio HTTP se accede directamente desde el navegador, escribiendo la IP en la barra de direcciones:

```
http://<IP>
```

Esto te llevó a un formulario de login de una aplicación web.

#### 3. Identificar la vulnerabilidad

Las preguntas te guiaron por el concepto de SQL Injection, una de las vulnerabilidades más comunes (clasificada como A03:2021 – Injection en el OWASP Top 10).

#### 4. Explotación: bypass de login con un comentario SQL

En el campo de usuario escribiste:

```
admin'#
```

Y cualquier cosa en contraseña. Esto rompió la consulta SQL interna del servidor, comentando la parte que comprobaba la contraseña, y te dejó entrar directamente como `admin` sin necesidad de saber la contraseña real.

#### 5. Resultado

La propia página te devolvió la flag directamente en pantalla tras el login exitoso.