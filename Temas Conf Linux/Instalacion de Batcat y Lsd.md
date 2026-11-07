Descargar las ultimas versiones de estos dos repositorios 
https://github.com/lsd-rs/lsd/releases/tag/v1.2.0
https://github.com/sharkdp/bat/releases/tag/v0.26.1

El primer .deb que aparece

"**`dpkg`** es el comando que coges cuando ya tienes el "paquete" físico (el archivo `.deb`) sentado en tu carpeta de Descargas y quieres que pase a formar parte de tu sistema."

![[Pasted image 20260304171336.png]]


Tenemos que cambiar el ls colors en desktop Y cambiar el valor de la variable de entorno

![[Pasted image 20260304200536.png]]



Metemos también el export en el .zshrc y buscas el echo Path del usuario normal y también lo pegas

![[Pasted image 20260304201412.png]]


Ahora vamos a hacer que mi ls sea lsd y mi cat sea bat dentro de .zshrc

![[Pasted image 20260304202120.png]]







Ahora abrimos una nueva nos metemos como sudo su a descargar y tenemos que instalar y seguir paso a paso todo esto 
https://medium.com/@fidjolakoka/burpsuite-no-longer-launches-after-parrot-upgrade-d1c6b17cb70d

Y le damos a burpsuite en la terminal (Recomendable fuera de root) y descargamos



#### Como usarlos

Bat: bat /etc/hosts
Lsd: lsd -l 