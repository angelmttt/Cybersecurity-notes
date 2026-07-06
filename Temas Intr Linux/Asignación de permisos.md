https://www.ionos.es/digitalguide/servidores/know-how/asignacion-de-permisos-de-acceso-con-chmod/
https://atareao.es/tutorial/terminal/propietarios-y-permisos/

Gestion de usuarios,grupos y permisos en linux
https://computernewage.com/2016/05/22/gestionar-usuarios-y-permisos-en-linux/

 Gestión de usuarios y grupos en Linux: 
 https://atareao.es/como/gestion-de-usuarios-y-grupos-en-linux/


mkdir ---> creas directorio
rm -r borras tanto el directorio existente como los que hay por detras
Ademas si pones -rf no te pregunta si de verdad lo quieres borrar o no

cmod --> cambias los privilegios

chmod -(o,g,x)

```
cmod o+w prueba
```
asignas a otros que tengan capacidad de write en prueba

```
chmod o-w prueba
```
Ahora le quitas los privilegios

chgrp -----> cambias el grupo de un directorio

```
chgrp angelmt prueba
```
Asignas el nuevo grupo de prueba a angelmt

```
chmod u-x,g-rx,o+w,o-x prueba
```
Asi modificas todos

### Creacion de usuarios

useeradd --->  creas un nuevo usuario
-s indicas el tipo de shell
-d el directorio personal

```
useradd juan -s /bin/bash -d /home/juan
```

Contraseña
passwd ---> asignas una contraseña

```
chgrep pepe pepe
chown pepe pepe
////
chown pepe:pepe pepe
```
Asignas tanto que el grupo y el propietario sea pepe en vez de root que es como viene predefinido


groupadd ----> creacion de nuevo grupo

```
usermod -a -G Alumnos pepe
```

- **`usermod`**: Es la herramienta principal para "modificar" las propiedades de una cuenta de usuario ya existente.
    
- **`-G` (Groups)**: Indica que vas a listar los grupos secundarios (o suplementarios) a los que quieres que pertenezca el usuario.
    
- **`-a` (Append)**: Es la opción más importante. Significa **"añadir"**. Sin esta letra, Linux borraría a `pepe` de todos sus grupos actuales y lo dejaría _solo_ en el grupo `Alumnos`. Con `-a`, simplemente sumas el nuevo grupo a su lista.
    
- **`Alumnos`**: Es el nombre del grupo al que quieres que entre.
    
- **`pepe`**: Es el nombre del usuario que estás modificando.


