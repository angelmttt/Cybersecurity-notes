Ejercicio 3 bandit
Lectura de un archivo oculto
```
cat ./...Hiding-From-You
cat ./hidden
cd..
find . -type f | grep -vE "bashrc|profile|logout" | xargs cat
	(nos busca los archivos)
```
El comando busca todos los **ficheros** (no carpetas) dentro del directorio actual y sus subcarpetas, **descarta aquellos cuyo nombre** contenga 'bashrc', 'profile' o 'logout', y finalmente **vuelca el contenido** de todos los archivos restantes en la pantalla.

contraseña 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
