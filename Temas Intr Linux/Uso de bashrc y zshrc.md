¿Qué es Bashrc en Linux?
https://www.compuhoy.com/que-es-bashrc-en-linux/
¿Por qué deberías usar ZSH?
https://respontodo.com/que-es-zsh-y-por-que-deberia-usarlo-en-lugar-de-bash/


~/.zshrc  ---> Ruta para acceder
/home/angelmt/.zshrc

Archivo para crear nuevas funciones que te merezca la pena. Se modifica aqui lo de cat por batcat el ls por lsd

awk ---> puedes printear el primer argumento
```
hostname -I | awk '{print $1}'
```

EJEMPLO DE FUNCIÓN
dentro de .zshrc
```
function vermiip(){
	echo "Tu Ip privada es: $(hostname -I | awk '{print $1}')"
}
```


Para añadir un parámetro de output hay que ponerlo ----> $()
ls -a --> muestra archivos ocultos (empieza por un .)