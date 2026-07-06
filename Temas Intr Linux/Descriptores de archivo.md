https://hack4u.io/wp-content/uploads/2022/05/bash-redirections-cheat-sheet.pdf

```
exec 3<> file
```
Crea un descriptor de archivo con el numero 3 se guarda en formato append (lo mete debajo)
	<+>//<-> 
		-  solo lectura
		+  escribir 

```
whoami >&3
```
Mandamos el output a ese descriptor de archivo

 Cerrar el descriptor de archivo

```
exec 3>&-
```
Cerrar el descriptor de archivos pero no pierde el contenido. De cara a un futuro output no funionaría

Copiar descriptor de archivo

```
exec 8>&3
```
Se copia en el descriptor de archivo 3 en el 8

Mientras que creas la copia cierra el otro descriptor de archivos
```
exec 5<> example
whoami >&5
exec 6>&5-  // Creas el 6 con los datos del 5 y además se cierra el 5
```




