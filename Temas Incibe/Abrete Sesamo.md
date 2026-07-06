## Cómo analizar un binario ELF — Guía paso a paso

---

### 1️⃣ Reconocer el archivo

Lo primero siempre es saber con qué tipo de archivo estás tratando:

bash

```bash
file flag
```

Nos dice que es un **ELF 64-bit** — es decir, un ejecutable de Linux de 64 bits. También nos dice si está "stripped" (sin símbolos de debug) o "not stripped" (con ellos). En este caso estaba **not stripped**, lo que nos facilita mucho el trabajo.

---

### 2️⃣ Extraer strings visibles

bash

````bash
strings flag
```

Este comando extrae todas las cadenas de texto legibles del binario. Aquí encontramos cosas muy útiles:
```
Introduce la cadena mágica como argumento
tienes la flag:
Lo siento, %s no es correcto
````

Esto ya nos dice que el programa **espera un argumento** y que si es correcto imprime la flag. También vimos strings raros como `P10=zuie`, `cmJ5,3za`... eso son los **buffers cifrados** que veremos luego.

---

### 3️⃣ Desensamblar — leer el código máquina

bash

```bash
objdump -d flag
```

Este comando convierte los bytes del binario en **instrucciones ensamblador** legibles. Nos centramos en la función `main`. Lo que encontramos:

#### 🔐 Dos buffers hardcodeados

El programa carga dos bloques de bytes directamente en la pila con instrucciones `movabs` y `movl`:

asm

```asm
movabs $0x6569757a3d303150, %rax   ; → buf_key
movabs $0x617a332c354a6d63, %rdx
movl   $0x726b332d, ...

movabs $0x3a0819015a515d36, %rax   ; → buf_cipher
movabs $0x323b6c4d59280c17, %rdx
movl   $0x0f227a6e, ...
```

> ⚠️ Importante: x86-64 usa **little-endian**, así que los bytes se invierten al leerlos de memoria. Usamos `struct.pack('<Q', valor)` en Python para ordenarlos correctamente.

#### ✅ Dos condiciones de validación

asm

```asm
cmpl $0x15, -0x8(%rbp)    ; len(input) == 21
cmpl $0x929, -0xc(%rbp)   ; suma ASCII de todos los chars == 2345
```

Si las dos se cumplen → pasa. Si no → "Lo siento, X no es correcto".

#### 🔑 Generación de la flag por XOR

Si el input es válido, el programa hace un bucle XOR:

asm

```asm
movzbl -0x50(%rbp,%rax,1), %edx   ; byte de buf_cipher
movzbl -0x30(%rbp,%rax,1), %eax   ; byte de buf_key
xor    %edx, %eax                  ; XOR entre ellos
```

Es decir: `flag[i] = buf_cipher[i] XOR buf_key[i]`

---

### 4️⃣ Calcular la flag en Python

Con los buffers extraídos del ensamblador, calculamos el XOR:

python

````python
import struct

buf_key    = struct.pack('<Q', 0x6569757a3d303150) + \
             struct.pack('<Q', 0x617a332c354a6d63) + \
             struct.pack('<I', 0x726b332d)

buf_cipher = struct.pack('<Q', 0x3a0819015a515d36) + \
             struct.pack('<Q', 0x323b6c4d59280c17) + \
             struct.pack('<I', 0x0f227a6e)

flag = bytes([c ^ k for c, k in zip(buf_cipher, buf_key)])
print(flag.decode())  # → flag{la_tabla_ASCII}
```

---

### 5️⃣ Construir un input válido para ejecutar el binario

Para verificarlo ejecutando el propio binario, necesitamos un string de **21 chars y suma ASCII = 2345**. El más simple:
```
20 × 'o' (valor 111)  +  '}' (valor 125)
= 20×111 + 125 = 2220 + 125 = 2345 ✓
````

bash

````bash
./flag "oooooooooooooooooooo}"
# → Aquí tienes la flag: flag{la_tabla_ASCII}
```

---

### 🧠 Resumen del flujo
```
file        →  ¿qué tipo de binario es?
strings     →  ¿qué texto hay dentro? ¿pistas sobre la lógica?
objdump -d  →  ¿cómo funciona por dentro? (condiciones, operaciones)
Python      →  replicar la lógica y calcular la flag sin ejecutar nada
````

Este flujo sirve para la mayoría de retos de **reversing básico** en CTFs. La clave está en identificar: las **condiciones de validación** y las **operaciones sobre datos** (XOR, suma, restas...).