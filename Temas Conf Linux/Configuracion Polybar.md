
### Modulo de Ip

Dentro de .config/polybar, existen varias carpetas donde se puede modificar la poly

Accedemos en current.ini filtramos por log (ctrl w) y camiamos el background por bg
![[Pasted image 20260305093821.png]]
Ademas lo hacemos con primary lo ponemos en bg
![[Pasted image 20260305115025.png]]

Tambien cambiamos el my-text-label por lo que filtramos por nombre buscamos en nerdfonts el icono de parrot copiamos y lo pegamos
https://www.nerdfonts.com/cheat-sheet
![[Pasted image 20260305094319.png]]

Ahora tenemos que crear una font 7 para que aparezca igual que antes filtramos por font-
![[Pasted image 20260305094944.png|624]]

Creamos nuestro propio modulo para que aparezca nuestra ip privada vamos a launch.sh y cambiamos el nombre a ethernet_bar para posteriormente hacerlo
![[Pasted image 20260305095215.png]]

Nos vamos a launch.sh filtramos por secondary lo copiamos y lo pegamos arriba todo para modificarlo cambiando el nombre por ethernet
![[Pasted image 20260305095607.png]]

Cambiamos tanto el nombre como el width como el module-center por ethernet_status por lo que ahora tenemos que volver a filtrar por module/date que era el anterior y copiamos las dos primeras partes y lo modificamos casi al final del todo

![[Pasted image 20260305101857.png]]

Ahora creamos el archivo le damos permisos y modificamos
![[Pasted image 20260305100221.png]]

Nos metemos con nano en el .zshrc y filtramos por PATH y añadimos esto
![[Pasted image 20260305100657.png]]

Copiamos el pastebin y pegamos en el touch
https://pastebin.com/HcKxU3tD
Ponemos el icono que querramos
![[Pasted image 20260305100828.png]]


### Modulo VPN


![[Pasted image 20260305105720.png]]
Nos metemos en launch.sh  creamos el nuevo modulo para la vpn
![[Pasted image 20260305105749.png]]

Nos volvemos a meter en current.ini filtramos por ethernet_bar copiamos todo el bloque cambiando el nombre por vpn_bar y poniendo estos valores 
![[Pasted image 20260305110210.png]]

Filtramos por ethernet_status copiamos y pegamos justo abajo cambiando el nombre y la ruta
![[Pasted image 20260305110438.png]]

Ahora solo queda crear el archivo y le pegamos el raw de vpn 
https://pastebin.com/sUk5hB4Q
![[Pasted image 20260305110845.png]]

![[Pasted image 20260305110758.png]]

Ahora para acceder a hackthebox. Nos meteriamos en htb pondriamos con la vpn y lo guardamos
![[Pasted image 20260305114146.png]]



Creamos carpeta htb y vpn ejecutamos el comando vpn lo descargamos 
![[Pasted image 20260305114648.png]]

Y ya tendria que estar configurado 
![[Pasted image 20260305114723.png]]