
Tutorial para cambiar a Arch Linux S4vitar:
https://www.youtube.com/watch?v=fshLf6u8B-w

 Comandos Find y Locate en Linux:
 https://www.hostinger.com/es/tutoriales/como-usar-comando-find-locate-en-linux

find ---> busqueda para lo que quieras 

```
find / -name passwd | xargs ls -l 
```

Te busca los archivos que tengan de nombre passwd

```
find / -perm 4000
```
Buscas que archivos tengan permisos SUID

```
find / -group angelmt
//
find / -group angelmt -type d (directorio) f (archivo/file)
```
Archivos y directorios que formen del grupo angelmt 

```
find / -user root -writeable
```
Archivos del usuario root con permiso de escritura

```
fint / -user root -type f -executable
```
Archivos del usuario root con permiso ejecutable. Aunque para directorios la x significa que podemos atravesar

```
find / -name dex\*
find / -name dex\*.sh
find / -name \*exdum\*
```
Cuando no te sabes el nombre entero te sirve para decir que empieza dex... y abajo es para busca ...exdum...

```
find --help 
```
Te aparecen mas comandos