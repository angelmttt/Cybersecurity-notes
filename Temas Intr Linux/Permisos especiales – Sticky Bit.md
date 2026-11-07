https://keepcoding.io/blog/que-es-el-sticky-bit-y-como-configurarlo/
https://www.fpgenred.es/GNU-Linux/el_bit_sticky.html

```
chmod +t pruebaspepito
```
Ahora con el t aplicas el Sticky Bit
Así no puedes borrar un archivo aunque el directorio anterior permita borrar YA NO TE DEJA
("cláusula de seguridad" que se impone por encima de las reglas normales del directorio.)

ejemplo de sticky bit
![[Pasted image 20260308202647.png]]

### Definición del Sticky Bit

Es un permiso especial aplicado a directorios que permite que todos los usuarios puedan crear, leer y editar archivos, pero restringe la eliminación.

- Su función principal: Solo el dueño del archivo (o el usuario `root`) puede borrar o renombrar un archivo dentro de esa carpeta.

- Para qué sirve: Evita que un usuario "A" borre por accidente (o por maldad) los archivos del usuario "B" en una carpeta donde ambos tienen permisos de escritura.
