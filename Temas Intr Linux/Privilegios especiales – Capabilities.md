¿Que son las Linux Capabilities?
https://dcollao.pages.dev/CPTS/D/1/4/4/

getcap  --> permite listar las capabilities del sistema

```
getcap -r / 2>/dev/null
```
Lista las capabilities del sistema


cap_setuid=ep // Permite cambiarte el uid

```
setcap -r /usr/bin/python3.9
```
Ahora ya no dejas la capabilitie de python