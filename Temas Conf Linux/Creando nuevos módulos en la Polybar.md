
Seguimos como root en /home/angelmt/.config/polybar

Creación de la barra Target
![[Pasted image 20260306183352.png]]

Nos metemos ahora en current ini filtramos por secondary copiamos y pegamos el código hasta que quede así 
![[Pasted image 20260307103414.png]]

Bajamos abajo del todo y lo mismo copiamos una y ponemos los valores de victim
![[Pasted image 20260307104248.png]]

Creamos carpeta bin y fichero target para ahora usarlo
![[Pasted image 20260307105204.png]]

Vamos a la ruta de bspwm y scripts creamos con touch victim_to_target.sh le damos permisos y copiamos este paste modificando la ruta y poniendo el icono
https://pastebin.com/Z0s6zy7W

![[Pasted image 20260307104947.png]]


Ahora accedemos al directorio .zshrc y metemos las funciones settarget y cleartarge copiadas de estos dos links
https://pastebin.com/z0Hy7PUB (settarget)
https://pastebin.com/JnHSrq3Y (cleartarget)
![[Pasted image 20260307110259.png]]

Y por último configuramos los colores que se encuentran en el archivo workspace.ini en polybar
Filtramos por occupied y cambiamos a estos colores
![[Pasted image 20260307111813.png]]

Por último instalamos Fzf
Lo hacemos tanto en el no privilegiado como en el privilegiado
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
![[Pasted image 20260307112429.png]]

Lo usamos con ctrl + t
[[Comandos Terminal]]


