https://blog.desdelinux.net/permisos-y-derechos-en-linux/
https://www.profesionalreview.com/2017/01/28/permisos-basicos-linux-ubuntu-chmod/
https://www.softzone.es/linux/tutoriales/permisos-archivos-directorios-linux/
https://www.hostinger.com/es/tutoriales/cambiar-permisos-y-propietarios-linux-linea-de-comandos


touch --> creación de un archivo vacío
nano ---> creacion de un archivo y se puede escribir al momento

```
echo "Hola Mundo" > file.txt
echo "Adios Mundo" > file.txt
```
Se sobreescribiria el segundo sobre el primero porque solo se usa un >

```
echo "Hola Mundo" >> file.txt
echo "Adios Mundo" >> file.txt
```
Ahora si que aparecería los dos y de forma append

rm ---> eliminar el archivo

### PERMISOS

![[Pasted image 20260308143851.png]]
d (directorio)
. (ejecutable)
.rw-r--r
	 Tres partes rwx 
	 Primera propietario
	 Segundo grupo
	 Tercero otros
		 r Read
		 w Escritura
		 x 
			 x --- Directorio se puede meter
			 x --- Ejecutable el grupo se puede meter
	Los siguientes parámetros
	El primero es quien lo ha creado (propietario)
	El segundo es el grupo que lo pertenece (grupo)

