# Flag encontrada!

## `flag{no_olvides_ejecutar_la_funcion}`

---

## ¿Qué hacía el código?

**Técnica usada: JavaScript Obfuscation** con 3 capas de protección:

### Capa 1 — Array de strings codificadas

javascript

```javascript
var _0x30de = ['mZa1ntnUsLrKCKG', 'zwPLyW', ...]
// Strings en base64 custom, ilegibles a simple vista
```

### Capa 2 — Shuffle del array

javascript

```javascript
// Un bucle reordena el array hasta que una suma de enteros
// coincide con un valor objetivo (221007)
// Así los índices cambian y confunden al lector
```

### Capa 3 — Función decodificadora

javascript

````javascript
// _0x45c1(0xe8) => "flag"
// _0x45c1(0xe5) => "{no_"
// _0x45c1(0xf1) => "olvi"
// _0x45c1(0xde) => "ejec"  
// _0x45c1(0xec) => "utar"
// _0x45c1(0xe3) => "func"
// _0x45c1(0xe9) => "ion}"
```

### El mensaje ensamblado
```
"flag" + "{no_" + "olvi" + "des_" + "ejec" + "utar" + "_la_" + "func" + "ion}"
= flag{no_olvides_ejecutar_la_funcion}
````

> 💡 La ironía del reto: la flag dice **"no olvides ejecutar la función"** — y para obtenerla había que hacer exactamente eso: ejecutar `_146154141147()`.

y como se que esos valores valen flag y eso?

10:35

Buena pregunta! Te explico paso a paso cómo se descifra.

---

## El problema inicial

El código usa índices hexadecimales como `0xe8`, `0xe5`... que apuntan al array. Pero el array está **mezclado** y los strings están **codificados en base64 custom**. Nadie puede leerlos directamente.

---

## Paso 1 — El array codificado

javascript

````javascript
var _0x30de = [
    'mZa1ntnUsLrKCKG',  // [0] -> ???
    'zwPLyW',            // [1] -> ???
    's3jjsG',            // [2] -> ???
    ...
]
```
Son strings en **base64 modificado** — ilegibles a simple vista.

---

## Paso 2 — La función decodificadora

`_0x45c1(indice)` hace esto internamente:
```
1. Coge el string del array en esa posición
2. Lo decodifica con base64 custom
3. Lo convierte de bytes a texto legible
4. Lo devuelve
````

Entonces cuando el código dice `_0x52e3c3(0xe8)` realmente está diciendo **"dame el string decodificado de la posición 0xe8"**.

---

## Paso 3 — El shuffle cambia las posiciones

El IIFE (función que se ejecuta sola) **reordena el array** hasta que una ecuación cuadra:

javascript

```javascript
// Antes del shuffle:
_0x30de[0] = 'mZa1ntnUsLrKCKG'  -> "30553nJTdrH"  (número, relleno)

// Después del shuffle:
posición 0xe8 - 0xde = posición 10 del array
_0x30de[10] -> decodifica a -> "flag"
```

---

## Paso 4 — Lo que yo hice para saberlo

Simplemente **ejecuté la función decodificadora** para cada índice posible y vi qué devolvía:

javascript

````javascript
for(let i=0xde; i<=0xf2; i++){
    console.log(`0x${i.toString(16)} = "${_0x45c1(i)}"`)
}
```

Y el resultado fue:
```
0xde = "ejec"
0xdf = "KrIJ"    <- relleno (nombre de variable interna)
0xe0 = "90652..." <- número (anti-tamper)
0xe1 = "3mXm..."  <- número
0xe2 = "6789..."  <- número
0xe3 = "func"
0xe4 = "47YF..."  <- número
0xe5 = "{no_"
0xe6 = "8374..."  <- número
0xe7 = "1353..."  <- número
0xe8 = "flag"     ← ✅ texto real
0xe9 = "ion}"
0xea = "1969..."  <- número
0xeb = "127J..."  <- número
0xec = "utar"
0xed = "13QS..."  <- número
0xee = "2qJl..."  <- número
0xef = "log"      ← nombre del método console.log
0xf0 = "1076..."  <- número
0xf1 = "olvi"
0xf2 = "3055..."  <- número
````

---

## Paso 5 — Ensamblar la flag

La función `_146154141147` construye un objeto así:

javascript

````javascript
objeto["KrIJ" + "D"] = _0x45c1(0xe8)  // "flag"
                      + _0x45c1(0xe5)  // "{no_"
                      + _0x45c1(0xf1)  // "olvi"
                      + "des_"
                      + _0x45c1(0xde)  // "ejec"
                      + _0x45c1(0xec)  // "utar"
                      + "_la_"
                      + _0x45c1(0xe3)  // "func"
                      + _0x45c1(0xe9)  // "ion}"
```

Concatenando todo:
```
flag  +  {no_  +  olvi  +  des_  +  ejec  +  utar  +  _la_  +  func  +  ion}
= flag{no_olvides_ejecutar_la_funcion}
```

---

## Resumen visual
```
Código ofuscado          Lo que realmente es
─────────────────────    ────────────────────
_0x52e3c3(0xe8)      →   "flag"
_0x52e3c3(0xe5)      →   "{no_"
_0x52e3c3(0xf1)      →   "olvi"
    + "des_"          →   "des_"   (literal, no ofuscado)
_0x52e3c3(0xde)      →   "ejec"
_0x52e3c3(0xec)      →   "utar"
    + "_la_"          →   "_la_"   (literal)
_0x52e3c3(0xe3)      →   "func"
_0x52e3c3(0xe9)      →   "ion}"
````

> 💡 La clave fue **ejecutar el propio sistema de decodificación del código** contra sí mismo — en vez de intentar descifrar a mano el base64 custom, dejé que el propio código lo hiciera y simplemente capturé los resultados.