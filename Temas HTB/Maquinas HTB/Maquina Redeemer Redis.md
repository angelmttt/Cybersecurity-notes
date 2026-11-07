![[Pasted image 20260620184627.png]]
Cuando el escaneo normal no encuentra nada, no significa que no haya nada abierto, simplemente puede estar fuera del top 1000. Ahí es cuando usas:

```
nmap -p- <IP>
```

para escanear todos los puertos sin excepción y no perderte nada.
![[Pasted image 20260620184616.png]]

![[Pasted image 20260620184928.png]]
![[Pasted image 20260620185120.png]]![[Pasted image 20260620185054.png]]
![[Pasted image 20260620185546.png]]
![[Pasted image 20260620185458.png]]
Vale, lo de `SELECT 0` ya está bien hecho, ese `OK` solo confirma que cambiaste a la base de datos 0 correctamente

Task 9 Con DBSIZE
![[Pasted image 20260620185635.png]]
![[Pasted image 20260620185826.png]]
![[Pasted image 20260620185841.png]]

### Resumen, Máquina "Redeemer" (HTB Starting Point)

#### 1. Reconocimiento con Nmap

El escaneo por defecto no encontró nada porque Redis no está en el top 1000 de puertos que Nmap revisa por defecto. Tuviste que especificar el puerto manualmente:

```
nmap -sV -p 6379 <IP>
```

Resultado:

```
6379/tcp open  redis   Redis key-value store 5.0.7
```

#### 2. Conectarse a Redis

```
redis-cli -h <IP>
```

- `redis-cli` → el cliente de línea de comandos para interactuar con Redis
- `-h` → específica el host (la IP) al que conectarte

#### 3. El fallo de seguridad: sin autenticación

Te conectaste directamente, sin usuario ni contraseña. Mismo patrón que en Meow, Fawn y Dancing, un servicio expuesto sin ningún tipo de protección.

#### 4. Explorar el servidor

```
INFO          → información y estadísticas del servidor
SELECT 0      → cambiar a la base de datos número 0 (Redis tiene varias, numeradas)
KEYS *        → listar todas las claves guardadas en esa base de datos
GET <clave>   → ver el valor guardado en una clave concreta
```

#### 5. Conseguir la flag

Con `KEYS *` localizaste el nombre de la clave que contenía la flag, y con `GET <clave>` obtuviste directamente su valor.