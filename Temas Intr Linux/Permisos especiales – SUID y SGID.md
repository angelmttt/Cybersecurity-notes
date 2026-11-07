Permisos SGID, SUID y Sticky Bit:
https://blog.deephacking.tech/es/posts/permisos-sgid-suid-stickybit/
Permisos especiales en Linux:
https://www.ochobitshacenunbyte.com/2019/06/17/permisos-especiales-en-linux-sticky-bit-suid-y-sgid/
Los bits SUID, SGID y Sticky
https://www.ibiblio.org/pub/linux/docs/LuCaS/Manuales-LuCAS/SEGUNIX/unixsec-2.1-html/node56.html

| xargs ls -l
	 xargs así no hace falta al output y lo hace directamente

Privilegio SUID

```
chmod u + s /usr/bin/python3.9
//
chmod 4755
```

Como descubrir binarios SUID

```
find / -type f -perm -4000 #(SUID) 2>/dev/null
```

Cuando un binario es un binario SUID puedo ejecutar python 3.9 lo ejecuto como el usuario

SGID
Igual a sgid pero a nivel de grupo

```
chmod g + s /usr/bin/python3.9
//
chmod 2755
```
En vez del 4 es con el 2

Como descubrir binarios SGID
```
find / -type f -perm -2000 #SGID 2>/dev/null
```

EJEMPLO DE SUID Y SGI
![[Pasted image 20260308202852.png]]

Se ve como el SGI afecta al grupo

## Definición de Gemini
### SUID (Set User ID)

Es un permiso especial que se aplica exclusivamente a archivos ejecutables.

- ¿Cómo funciona?: Cuando un usuario ejecuta un archivo con SUID, el proceso no corre con los permisos del usuario que lo lanza, sino con los permisos del dueño del archivo.

- El ejemplo clásico: El comando `passwd`. Para cambiar tu contraseña, el sistema debe escribir en `/etc/shadow`, un archivo que solo `root` puede tocar. Gracias al SUID, cuando tú usas `passwd`, el sistema te "presta" los poderes de `root` solo durante ese segundo para que puedas actualizar tu clave.

- Identificación visual: Verás una **`s`** (minúscula) en la posición de ejecución del dueño.

    - Normal: `-rwx------`

    - Con SUID: `-rws------`### 2. SGID (Set Group ID)

Este permiso tiene dos funciones dependiendo de dónde se aplique:

#### A. En Archivos (Igual que el SUID)

- El archivo se ejecuta con los privilegios del grupo dueño del archivo, en lugar del grupo del usuario que lo ejecuta.

#### B. En Directorios (El más usado)

- ¿Cómo funciona?: Cualquier archivo o subcarpeta que se cree dentro de ese directorio heredará automáticamente el grupo de la carpeta "madre".

- Para qué sirve: Es vital para el trabajo en equipo. Si tú (`angelmt`) y Pepe estáis en el grupo `Alumnos`, y creáis una carpeta con SGID para ese grupo, todos los archivos que hagáis allí serán del grupo `Alumnos` por defecto, facilitando que ambos podáis editarlos sin andar cambiando permisos manualmente.

- Identificación visual: Verás una **`s`** en la posición de ejecución del grupo.

    - Normal: `drwxr-xr-x`

    - Con SGID: `drwxr-sr-x`### Resumen para comparar (Octal)

Si usas números para `chmod`, estos bits especiales ocupan una cuarta posición al principio:

|Permiso Especial|Valor Octal|Efecto Principal|
|---|---|---|
|SUID|4|Ejecutas como el dueño.|
|SGID|2|Heredas el grupo del directorio.|
|Sticky Bit|1|Solo el dueño borra su archivo.|

