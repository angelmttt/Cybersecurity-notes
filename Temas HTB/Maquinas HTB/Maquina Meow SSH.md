![[Pasted image 20260620144119.png]]Terminal
![[Pasted image 20260620144259.png]]![[Pasted image 20260620144457.png]]El puerto 23 es siempre el mismo servicio, es uno de los "puertos clásicos" que merece la pena memorizar:

**Telnet**

(Es un servicio antiguo de acceso remoto, parecido a SSH pero sin cifrado — por eso es inseguro y suele aparecer en máquinas fáciles de HTB

![[Pasted image 20260620144659.png]]root

### Resumen — Máquina "Meow" (HTB Starting Point)

#### 1. Conexión VPN

Antes de poder atacar cualquier máquina de HTB, necesitas estar **dentro de su red privada**. Por eso te descargaste un archivo `.ovpn` y lo lanzaste con OpenVPN. Cuando viste `Initialization Sequence Completed`, significaba que ya estabas conectado a la red de HTB y podías "ver" las máquinas (10.129.X.X son direcciones internas de esa red).

#### 2. Conseguir el objetivo

Al darle a "spawn" en Starting Point, HTB levanta una máquina virtual solo para ti y te da su IP (`10.129.56.121`). Esa es tu objetivo.

#### 3. Identificar servicios (puerto 23 = Telnet)

En un escenario real aquí harías un escaneo con **Nmap** para ver qué puertos están abiertos. En este caso las preguntas ya te decían que el puerto **23/tcp** estaba abierto y corriendo **Telnet**.

**Telnet** es un protocolo antiguo para conectarse remotamente a otra máquina por consola — el problema es que **no cifra nada**, todo lo que escribes (usuario, contraseña, comandos) viaja en texto plano por la red. Por eso hoy en día casi nadie lo usa en producción, pero aparece mucho en máquinas de prácticas porque es muy fácil de explotar.

#### 4. Conectarse por Telnet

```
telnet 10.129.56.121
```

Esto abre una conexión al puerto 23 de la máquina objetivo.

#### 5. El fallo de seguridad: login sin contraseña

La máquina te pedía usuario y contraseña, pero estaba mal configurada: el usuario **root** (el administrador con todos los permisos en Linux) **no tenía contraseña asignada**. Por eso al dejar el campo de contraseña en blanco y dar a Enter, te dejó entrar directamente como root — el usuario con más privilegios del sistema.

Esto es una vulnerabilidad real y muy común: **credenciales débiles o inexistentes**.

#### 6. Encontrar la flag

Una vez dentro, como ya eras root, tenías acceso a todo el sistema. La flag estaba guardada en el directorio personal de root:

```
cat /root/flag.txt
```

Eso te mostró una cadena de 32 caracteres hexadecimales (la "flag"), que es la prueba de que lograste comprometer la máquina.

#### 7. Submit Flag

Pegaste esa cadena en HTB y se valida automáticamente que la máquina fue resuelta correctamente.

---

#### 💡 La lección de fondo

Esta máquina enseña la vulnerabilidad más básica que existe: **un servicio inseguro (Telnet) + credenciales por defecto/vacías = acceso total al sistema**. Es justo lo que en una auditoría real estaría documentando como "hallazgo crítico".