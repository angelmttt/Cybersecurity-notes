![[Pasted image 20260713172744.png]]


Nos metemos en la  http://10.129.48.148:3000/  no vemos nada así que buscamos directorios con la herramienta gobuster
```
gobuster dir -u http://10.129.48.148:3000/ -w /usr/share/dirb/wordlists/common.txt
```
![[Pasted image 20260713173728.png]]
solo `/cgi-bin/` con un código 308 (redirección), que es una carpeta estándar de Apache/nginx, prácticamente nunca contiene nada útil por defecto en este tipo de máquinas

Así que probamos con otra lista de palabras en el código había algo de next.js 
Vamos a usar `raft-medium-directories.txt` — buen equilibrio entre cobertura y tiempo de escaneo:
```
gobuster dir -u http://10.129.48.148:3000/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x js,json,txt
```
### Por qué añadir extensiones con `-x`

Hasta ahora, gobuster solo probaba nombres de **carpeta** (`/algo`, sin nada después). Pero en aplicaciones modernas, muchas rutas interesantes son **archivos concretos**, no carpetas — por ejemplo:

- `/config.json` (un archivo de configuración expuesto)
- `/api.js`
- `/backup.zip`