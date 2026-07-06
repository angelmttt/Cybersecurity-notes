https://rm-rf.es/chattr-y-lsattr-visualizar-y-modificar-atributos-en-sistemas-de-ficheros-linux/#:~:text=El%20primer%20comando%2C%20lsattr%20permite,chmod%2C%20chown%2Csetfacl%E2%80%A6)

https://programmerclick.com/article/5604675172/

cp --> para hacer copias

chattr sirve para asignar una flag especial

```
chattr +i +v prueba
```
+i de inmutable qasi no se puede modificar renombrear o borrar (con -i lo quitas)

+v verbose ---> forma al ejecutar un comando de forma descriptiva que esta pasando por detras

+A conseguimos que cuando se acceda al fichero no se modifique el registro atime. De este modo no quedarán la fecha del último acceso al fichero

+c cuando se leaw el ifchero se servira descomprimido

+u cuando el fichero dse borre, sus datos permanezcan guardados y permitan al usuario su recuperación

+e implica que cuando un archivo sea borrado, automaticamente los bloques utilizados por el mismo sean sobreesceito con ceros

+S fichero modificado, los cambios se aplican a disco de forma sincrona y directa. Es lo mismo que tener la flag sync en el punto de montaje de filesystem

```
chattr +uiAc prueba
```

Así se asignan todos a la vez

lsattr ---> puedes ver las flags de un archivo