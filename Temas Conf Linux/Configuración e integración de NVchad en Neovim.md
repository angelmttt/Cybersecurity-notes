
### NEOVIM
Primero desintalamos el neovim que tenemos como sudo 
![[Pasted image 20260307113411.png]]

nos salimos del root nos ponemos como angelmt y ejecutamos este comando 
```
git clone https://github.com/NvChad/starter ~/.config/nvim
```
![[Pasted image 20260307113615.png]]

Nos metemos en neovim y descargamos el ultimo release el archivo siguiente y lo guardamos en descargas
![[Pasted image 20260307114037.png]]
https://github.com/neovim/neovim/releases/tag/v0.11.6


Creamos la carpetas desomprimir y todo 
![[Pasted image 20260307114326.png]]

Dentro de /home/angelmt/.zshrc añadimos donde export PATH la nueva ruta 
![[Pasted image 20260307114557.png]]

Ahora dentro de /opt ejecutamos nvim y una vez instalado para salir escribrimos :q!
![[Pasted image 20260307114933.png]]

Accedemso a nano ~/.config/nvim/init.lua y añadimos esta linea
https://pastebin.com/Lbk9vQLZ

![[Pasted image 20260307115318.png]]

Instlamos la siguiente utilidad y luego escribimos (si nos da permiso denegado simplemente ponemos umount y lo que estaba dando denegado)
```
sudo updatedb
```

![[Pasted image 20260307115456.png]]

![[Pasted image 20260307115809.png]]

##### Nuevo comando
locate .py (lo que quieras y te aparece todo)

Nos metemos con nvim init.lua y escribimos el siguiente comando
```
esc + space + t + h (Para cambiar el color del fondo)
```

![[Pasted image 20260307120838.png]]

Tambien ctrl + n
![[Pasted image 20260307121005.png]]

[[Comandos Terminal]]

### ROFI
![[Pasted image 20260307122633.png]]
Y clonamos este repositorio
```
sudo git clone https://github.com/newmanls/rofi-themes-collection
```

![[Pasted image 20260307122854.png]]

Copiamos los documentos .rasi
![[Pasted image 20260307123609.png]]

Salimos del root vamos al rofi-themes-selector buscamos el squared-material-red le damos al enter para ir probando y luego para guardar
```
alt + a
```
![[Pasted image 20260307123910.png]]
Entonces ahora cerramos y cada vez que le demos a windows + d tendra que aparecer así
![[Pasted image 20260307123956.png]]

### i3loock-fancy
Primero instlaamos los paquetes en la terminal
![[Pasted image 20260307124202.png]]

Vamos a la carpeta opt y clonamos este repositorio
```
git clone https://github.com/meskarune/i3lock-fancy.git
```
![[Pasted image 20260307124345.png]]Accedemos a la carpeta y hacemos make install
![[Pasted image 20260307124433.png]]

Y ahora ya podemos ejecutarlo con i3lock-fancy IMPORTANTE NO HACERLO EN ROOT PQ NO TIENE CONTRASEÑA

Vamos a crear la ruta i3lock con un atajo 
Nos metemos con nvim en sxhkdrc (puede ser que necesitemos privilegios) y añadimos el atajo y salimos con
```
:wq
```
![[Pasted image 20260307125524.png]]

![[Pasted image 20260307125501.png]]

windows + escape para recargar


Accedemos a este init.lua con nvim
![[Pasted image 20260308115623.png]]

Y borramos desde -- load luasnips + cmp related in insert mode only hasta que se cierre el parentesis
![[Pasted image 20260308120322.png]]


AHORA LO MISMO CON ROOT
![[Pasted image 20260308121031.png]]
![[Pasted image 20260308121742.png]]


CAMBIOS EN FIREFOX

Nos metemos en firefox asi no se queda nada guardado 
![[Pasted image 20260308121916.png]]

Ahora para que no haya problemas para conectar a Mantis nos metemos a about:config
![[Pasted image 20260308122153.png]]

Y añadimos esta
![[Pasted image 20260308122309.png]]

Descargamos las siguientes extensiones
https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/
https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/

Dentro de foxyproxy creamos uno nuevo 
Iniciamos burpsuite
![[Pasted image 20260308122951.png]]

Lo unico es que ahora si buscamos no nos va a dejar por lo que tenemos que buscar http://burp/ y descargamos el CA
![[Pasted image 20260308123738.png]]

Ahora nos vamos a settings
![[Pasted image 20260308123833.png|211]]
![[Pasted image 20260308123908.png]]

He importamos el certificado que nos acabamos de instalar y marcando la primera casilla que nos aparece y Ok

Ultimas instalaciones

```
apt install ranger
```
Para poder navegar por directorios comodamente
Como usarlo en la terminal escribimos
```
	ranger
```

Y YA ESTARIA TODO EL ENTORNO CONFIGURADO

### Definicion
**NVChad**

NVChad es un framework de configuración para Neovim que ofrece una experiencia de usuario mejorada y una suite de características prediseñadas. Es conocido por su interfaz de usuario estéticamente agradable y por su enfoque en la simplicidad y la eficiencia. NVChad viene preconfigurado con una serie de plugins cuidadosamente seleccionados y configuraciones optimizadas que abordan tanto la funcionalidad como la visualización. Esto incluye soporte para temas de color, gestión de buffers mejorada, autocompletado inteligente, y mucho más, todo ello manteniendo un rendimiento rápido. NVChad está diseñado para ser fácil de instalar y configurar, ofreciendo a los usuarios una plataforma robusta sobre la cual pueden construir o adaptar según sus necesidades específicas.

Ambas herramientas están diseñadas para complementarse entre sí, donde Neovim proporciona el entorno base y NVChad aprovecha y amplía sus capacidades, facilitando una experiencia de usuario altamente personalizada y eficiente para programadores y editores de texto.