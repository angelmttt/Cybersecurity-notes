Ejercicio 1 bandit:
Lectura de un archivo de nombre -

```
cat /home/bandit1/-
cat ./-
cat $(pwd)/-
grep -r "\w" 2>/dev/null | tail -n 1 | awk '{print $1}'
grep -r "\w" 2>/dev/null | tail -n 1 | tr ':' ' ' | awk 'NF{print $NF}'

```

Ejercicio 2 bandit:
Lectura de un archivo con espacio

```
cat ./*
find . -type f -name '--*' -print0 | xargs -0 cat
cat /home/bandit2/*
cat -- "--space in this filename--"
cat "./--spaces in this filename--"
```