## Paso 1: Identificar qué teníamos

Del bytecode descompilado extraje estas piezas clave:

```
"uB3GbV45fdz6YMWfZ"              ← clave AES (hardcodeada)
"3z,XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U="  ← ciphertext
"Y29tb3R1"                        ← base64("comotu") = nombre mascota
Rot13, XOREncryption, Base64      ← métodos del script
AES MODE_ECB                      ← modo de cifrado
```

---

## 🔍 Paso 2: Entender el problema con el ciphertext

El string original era:

```
3z,XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U=
```

Los primeros caracteres `3z,` **no son Base64 válido**. Eso pasa porque el descompilador de `.pyc` a veces añade **basura al principio** de las strings.

```
3z,XPDfIRDa09P...   ← basura + base64 real
   ↑
   saltar 2-3 chars
```

Lo comprobé intentando decodificar saltando 0, 1, 2 y 3 caracteres:

python

```python
for skip in range(0, 4):
    ct = base64.b64decode(raw_cipher[skip:])  # probar cada offset
```

Con `skip=2` → obtenía exactamente **32 bytes** (múltiplo de 16 ✓ válido para AES)

---

## 🔍 Paso 3: El problema con la clave

La clave `uB3GbV45fdz6YMWfZ` tiene **17 caracteres**, pero AES necesita exactamente **16, 24 o 32 bytes**.

Probé varias formas de derivarla:

python

````python
key_variants = {
    'md5':     hashlib.md5(key_raw).digest(),   # hash MD5 → 16 bytes
    'raw_16':  key_raw[:16],                     # truncar → 16 bytes ✅ ESTO FUNCIONÓ
    'sha256':  hashlib.sha256(key_raw).digest(), # hash SHA256 → 32 bytes
    'comotu':  hashlib.md5(b'comotu').digest(),  # quizás era el nombre
}
```

La que funcionó fue **truncar directamente a 16 bytes**:
```
uB3GbV45fdz6YMWf  ← solo los primeros 16 chars, se descarta la 'Z' final
````

---

## 🔍 Paso 4: Probar todas las combinaciones

Como no sabía exactamente cuál era la clave correcta ni el ciphertext correcto, hice un **ataque de fuerza bruta inteligente** — probé todas las combinaciones posibles:

python

````python
for skip, ct in ct_variants.items():       # 4 versiones del ciphertext
    for kname, key in key_variants.items(): # 7 versiones de la clave
        result = aes_ecb_decrypt(key, ct)
        
        # Filtro: ¿el resultado parece texto legible?
        printable = sum(1 for b in result if 32 <= b <= 126)
        if printable > len(result) * 0.6:   # más del 60% son caracteres imprimibles
            print(result)
```

El filtro del 60% es clave — descarta resultados que son basura binaria y solo muestra los que parecen texto real.

---

## 🔍 Paso 5: El resultado
```
flag{pyth0n_decompile_OK}\x07\x07\x07\x07\x07\x07\x07
```

Los `\x07` del final son el **padding PKCS#7** — AES trabaja en bloques de 16 bytes, y si el mensaje no llena el bloque, se rellena con bytes que indican cuánto padding hay (`\x07` = 7 bytes de padding).
```
flag{pyth0n_decompile_OK} = 25 chars
25 + 7 = 32 = 2 bloques AES de 16  ✓
```

---

## 📊 Resumen visual
```
Bytecode .pyc
    │
    ├── strings extraídas
    │       │
    │       ├── ciphertext "3z,XPDf..."
    │       │       └── skip 2 chars basura
    │       │               └── base64 decode → 32 bytes
    │       │
    │       └── clave "uB3GbV45fdz6YMWfZ"
    │               └── truncar a 16 bytes
    │
    └── AES-ECB decrypt
            │
            └── flag{pyth0n_decompile_OK} + padding \x07
````

---

## 💡 Lección aprendida

> Cuando el descompilador da strings "raras", **no te fíes ciegamente** — puede haber bytes de basura. La solución es probar múltiples offsets y múltiples formas de derivar la clave, y usar un **filtro de legibilidad** para detectar automáticamente cuál combinación produce texto real.

como se puede ver el archivo descompilado

10:02

Hay varias formas dependiendo de si tienes el `.py` o el `.pyc`:

---

## Si tienes el `.pyc` (bytecode compilado)

### Opción 1 — `uncompyle6` (la más común)

bash

```bash
# Instalar
pip install uncompyle6

# Descompilar
uncompyle6 script.pyc > script_decompilado.py

# Ver el resultado
cat script_decompilado.py
```

### Opción 2 — `decompile3` (para Python 3.9+)

bash

```bash
pip install decompile3

pycdc script.pyc
```

### Opción 3 — `pycdas` (ver el bytecode directamente)

bash

```bash
pip install pycdc

pycdas script.pyc
```

### Opción 4 — con Python puro (sin instalar nada)

bash

```bash
python3 -c "
import dis, marshal

with open('script.pyc', 'rb') as f:
    f.read(16)  # saltar cabecera
    code = marshal.loads(f.read())
    dis.dis(code)
"
```

---

## Si tienes el `.py` pero ofuscado

bash

````bash
# Ver directamente
cat script.py

# Si tiene eval/exec anidados, sustituir por print
sed 's/exec(/print(/g' script.py > script_claro.py
python3 script_claro.py
```

---

## Herramientas online (sin instalar nada)

| Web | Para qué |
|---|---|
| [pylingual.io](https://pylingual.io) | Descompilar .pyc online |
| [decompiler.com](https://www.decompiler.com) | Subir .pyc y ver el código |
| [python-decompiler.com](http://www.python-decompiler.com) | Alternativa online |

---

## Flujo recomendado en CTF
```
archivo.pyc
     │
     ├── uncompyle6 archivo.pyc    ← primero probar esto
     │         │
     │         └── ¿funciona? → leer el .py resultante
     │
     └── ¿falla? → probar decompile3 o web online
````

> 💡 **Truco**: si el `.pyc` está en Python 3.11+ ninguna herramienta lo descompila bien todavía — en ese caso usa `dis` para ver el bytecode o analiza las strings con `strings script.pyc` como hicimos nosotros.

¿Tienes el fichero `.pyc` del reto? Si lo subes lo descompilo yo directamente. 🐍

lo puedes darme el resultado descompilado para analizarlo

10:03

¡Claro! Sube el fichero `.pyc` y lo descompilo aquí mismo. 📁

Acepto:

- `script.pyc`
- `script.py` (si está ofuscado)
- `Python.py` (que mencionaste antes)

2 / 2

  

Sonnet 4.6

[Claude es IA y puede cometer errores. Por favor, verifi](https://support.anthropic.com/en/articles/8525154-claude-is-providing-incorrect-or-misleading-responses-what-s-going-on)


## ¿Qué es ese string?

Es la **flag cifrada** — lo que el script guarda dentro para que no se pueda leer directamente.

El script lo usaba así:

python

````python
# Dentro del script original
ciphertext = "3z,XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U="
# Si el usuario acierta el nombre de la mascota → descifra esto → muestra la flag
```

---

## ¿Qué formato tiene?

Está en **Base64** — es una forma de representar bytes binarios como texto legible:
```
Bytes binarios:  \x5c\xf0\xdf\x21\x10...   ← difícil de escribir/guardar
Base64:          XPDfIRDa09PzMVDkb3Y0...   ← fácil de guardar en código
```

Base64 solo usa estos caracteres:
```
A-Z  a-z  0-9  +  /  =
```

---

## El problema: los primeros caracteres `3z,`
```
3z,XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U=
↑↑↑
basura que añadió el descompilador
````

`3z,` **no son Base64 válido** — los añadió el descompilador de `.pyc` al extraer el string. Por eso había que saltárselos:

python

````python
# Incorrecto ❌
base64.b64decode("3z,XPDfIRDa09P...")   # error, chars inválidos

# Correcto ✅
base64.b64decode("XPDfIRDa09P...")      # saltar los 2 primeros chars
```

---

## El viaje completo del ciphertext
```
flag{pyth0n_decompile_OK}          ← texto original (la flag)
         │
         │  AES-ECB encrypt
         │  con clave uB3GbV45fdz6YMWf
         ▼
\x5c\xf0\xdf\x21\x10\xda\xd3...   ← bytes cifrados (ilegibles)
         │
         │  Base64 encode
         ▼
XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U=   ← guardado en el script
         │
         │  descompilador añade basura
         ▼
3z,XPDfIRDa09PzMVDkb3Y068rmkVC1gZWh0yGE59RPj3U=  ← lo que vimos nosotros
```

---

## Para descifrar, el proceso inverso
```
3z,XPDfIRDa09P...
         │
         │  saltar "3z," (basura)
         ▼
XPDfIRDa09P...
         │
         │  Base64 decode
         ▼
\x5c\xf0\xdf\x21...  (32 bytes)
         │
         │  AES-ECB decrypt
         │  con clave uB3GbV45fdz6YMWf
         ▼
flag{pyth0n_decompile_OK}  🎉
````

> 💡 **Resumen**: el ciphertext es la flag cifrada y codificada en Base64 para poder guardarla como texto dentro del script. El descompilador le añadió `3z,` de basura al principio, que era lo único que impedía decodificarlo directamente.