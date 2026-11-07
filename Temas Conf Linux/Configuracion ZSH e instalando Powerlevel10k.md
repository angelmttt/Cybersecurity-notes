
Instalamos estos tres recursos en picom
![[Pasted image 20260304121036.png|697]]
![[Pasted image 20260304121452.png]]
Nos metemos en github buscamos abajo del todo el código que pone manual y lo copiamos y pegamos 
https://github.com/romkatv/powerlevel10k
![[Pasted image 20260304121745.png]]

dentro de la ruta /home/angelmt/.zshrc tenemos que cambia la ruta por la nuestro en vez de la del usuario root
![[Pasted image 20260304121957.png]]

Vamos con la instalación de zsh 

[[Insalacion zsh]]

Y ahora lo volvemos a repertir con el usuario root



Salimos cerramos y abrimos consola y nos situamos con el sudo su en el cd /root y accedemos al .p10k.zsh
Una vez terminado filtramos por ROOT_TEMPLATE en .p10k.zsh
![[Pasted image 20260304125210.png]]

En ROOT_TEMPLATE quitamos lo que pone nos metemos en font nerd y buscamos un icono de una llama copiamos y pegamos
En CONTEXT_PREFIX lo borramos

Luego filtramos por dir_anchor y lo ponemos en false
![[Pasted image 20260304130325.png]]


CON ESTO YA ESTARIA LA POWERLVL10K

### ZSH

![[Pasted image 20260304130948.png]]
![[Pasted image 20260304131003.png]]

ahora ya apunta el root al de angelmt

![[Pasted image 20260304131301.png]]

Para no hacer esto siempre lo modificamos dentro del fichero
![[Pasted image 20260304132159.png]]
![[Pasted image 20260304135256.png]]

Añadimos otro plugin
https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/plugins/sudo/sudo.plugin.zsh
![[Pasted image 20260304135750.png]]

Y se añade otra vez en 
``` cd
nano .zshrc
```
![[Pasted image 20260304140237.png]]

Ahora cuando pongamos whoami y esc esc se pone en sudo

Y ahora vamos con history que añadimos justo abajo
![[Pasted image 20260304140654.png]]

Para vaciar el historial
```
echo '' > ~/.zsh history
```


Copiamos y lo pegamos en zshrc para el autocompletado
https://pastebin.com/H87J3nMj


### Definición 

**ZSH (Z Shell)**

ZSH, o Z Shell, es un intérprete de comandos para sistemas Unix, que se utiliza como una alternativa avanzada al tradicional shell Bash. Ofrece muchas mejoras en términos de características, plugins, y temas, lo que lo hace muy popular entre los usuarios avanzados y desarrolladores.

Algunas de las características más notables de ZSH incluyen:

- **Autocompletado**: ZSH proporciona un sistema de autocompletado potente y versátil que puede predecir comandos basados en el contexto, incluyendo opciones y parámetros.
- **Corrección de errores**: Ofrece sugerencias de corrección cuando se escribe un comando erróneamente.
- **Gestión de scripts**: ZSH es compatible con todos los scripts de Bourne Shell y Bash, y añade sus propias mejoras en la programación de scripts.
- **Personalización**: Se puede personalizar profundamente mediante temas y configuraciones que pueden cambiar su apariencia y comportamiento.

**Powerlevel10k**

Powerlevel10k es un tema para ZSH diseñado para ser visualmente atractivo y altamente informativo. Está diseñado para ser una versión más rápida y personalizable del popular tema Powerlevel9k.

Algunas de las características que lo hacen destacar incluyen:

- **Velocidad**: Es significativamente más rápido que otros temas para ZSH, lo que reduce el tiempo de inicio del terminal y la latencia al escribir comandos.
- **Personalizable**: Viene con un asistente de configuración que guía a los usuarios a través del proceso de configuración, permitiendo personalizar el prompt dependiendo de las preferencias del usuario.
- **Información contextual**: Muestra información útil en el prompt según el contexto, como el estado del repositorio git, el usuario actual, el tiempo de ejecución de comandos, y mucho más.
- **Iconos y fuentes**: Utiliza fuentes de Nerd Fonts para mostrar iconos y otros elementos gráficos que hacen que la información sea fácil de leer y visualmente atractiva.

Juntos, ZSH y Powerlevel10k ofrecen una experiencia de terminal rica y eficiente, mejorando tanto la funcionalidad como la apariencia del entorno de línea de comandos. Estas herramientas son particularmente populares entre los desarrolladores y los entusiastas de la personalización de sistemas Unix/Linux.