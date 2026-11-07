https://hack4u.io/wp-content/uploads/2022/05/bash-redirections-cheat-sheet.pdf

; sirve para separar comandos
&& el segundo comando se ejecuta si y solo si el primero se ejecuta
|| (OR)

$? ver el código de estado 0 EXITOSO =! 0 ERRONEO


STDOUT & STDERR

stdout **(Standard Output):** Es el canal de **salida normal**. Por aquí el programa envía los datos o resultados finales cuando todo funciona correctamente. (Descriptor de archivo 1).

stderr **(Standard Error):** Es el canal de **diagnóstico o errores**. Por aquí el programa envía las quejas, advertencias o fallos sin mezclarlos con el resultado principal. (Descriptor de archivo 2).

```
whoam 2>/dev/null
```
Quiero que me mandes el error a esa ruta así no vemos el error
(/dev/null) Agujero negro lo mandamos y desaparece 

```
cat /etc/host > /dev/null 2>&1
//
cat /etc/host &>/dev/null
//
wireshark &>/dev/null & ( & funciona sin bloquear                               la terminal)
wireshark &>/dev/null & disown (disown puedo cerrar                                 la terminal y que                                   siga funcionando)
```
Ejecuta un comando en modo silencioso absoluto, descartando tanto la información de salida exitosa como los posibles mensajes de error (stder y stdour)

Es recomendable cuando ejecutas un programa corriendo y no quieres que te aparezca los output solamente que se ejecute

PID número indentificativo que representa y diferencia a los procesos
 



