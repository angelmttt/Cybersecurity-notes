<h1 align="center">Cybersecurity Notes</h1>

<p align="center">
  Notas personales de ciberseguridad ofensiva: fundamentos, reconocimiento, enumeración, hacking web, metodología de auditoría y resolución de máquinas.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Enfoque-Seguridad%20Ofensiva-0b5394">
  <img src="https://img.shields.io/badge/Entorno-Parrot%20OS-5a2d82">
  <img src="https://img.shields.io/badge/Formato-Obsidian-7c3aed">
  <img src="https://img.shields.io/badge/Idioma-Espa%C3%B1ol-informational">
</p>

---

## Sobre este repositorio

Documento aquí mi aprendizaje en ciberseguridad ofensiva (pentesting, red team y CTFs), el campo en el que quiero desarrollar mi carrera. Estudio Ingeniería de Sistemas de Información (GISI) en la Universidad CEU San Pablo.

El objetivo no es acumular comandos, sino entender el porqué de cada técnica: qué se busca en cada fase, con qué herramienta y por qué.

- Ganador del I Desafío de Ciberseguridad (CTF) INCIBE & Sherpa Tribe.
- Prácticas en el área de Red Team de Atalanta Ciberseguridad.
- Herramienta propia: [CheckPermisos](https://github.com/angelmttt/CheckPermisos), auditoría de permisos en Linux.

## Estructura

| Directorio | Contenido |
|---|---|
| `Pentesting/Cursos Practicas/` | Metodología por fases: Fundamentos, Linux para Pentester, OSINT, Reconocimiento, Enumeración, Nmap y Hacking Web y API, más mi propia metodología de auditoría web |
| `Temas HTB/` | Teoría de los conceptos vistos en Hack The Box y writeups de las máquinas de Starting Point |
| `Temas Intr Linux/` | Fundamentos de Linux: permisos (octal, SUID/SGID, sticky bit, capabilities), descriptores, redirecciones, estructura de directorios, scripting y ejercicios de OverTheWire |
| `Temas Conf Linux/` | Configuración de un entorno completo sobre Parrot OS: bspwm, Polybar, Picom, Rofi, ZSH + Powerlevel10k y Neovim |
| `Raiz/` | Notas índice que enlazan las secciones entre sí |

## Temario

- **Reconocimiento** — OSINT, subdominios, whois, DNS, fingerprinting, filtraciones.
- **Enumeración** — SMB, FTP, SSH, bases de datos, SNMP, LDAP, APIs, servicios web.
- **Nmap** — escaneo TCP/UDP, detección de servicios, scripts NSE y Lua.
- **Hacking web** — SQLi, XSS, IDOR, CSRF, XXE, SSRF, LFI/RFI, subida de archivos y JWT.
- **Metodología de auditoría** — flujo de trabajo propio de caja negra y plantilla de informe de hallazgos.

## Uso ético

Todo el contenido tiene fines formativos. Las técnicas se han practicado únicamente en laboratorios propios, plataformas de entrenamiento (Hack The Box, OverTheWire, DVWA, WebGoat) o con autorización expresa. Los dominios, correos y datos de los ejemplos son ficticios (`paginaejemplo.com`, `empresax.com`). Los writeups incluidos son de máquinas de Starting Point, conforme a la [política de write-ups de Hack The Box](https://help.hackthebox.com/en/articles/5188925-write-up-guidelines).

## Cómo leer las notas

Las notas están en Markdown con enlaces internos de [Obsidian](https://obsidian.md) (`[[nota]]`). Para navegarlas con los enlaces y las imágenes funcionando, clona el repositorio y ábrelo como bóveda:

```bash
git clone https://github.com/angelmttt/Cybersecurity-notes.git
```

En Obsidian: *Abrir carpeta como bóveda* y seleccionar la carpeta clonada.

## Contacto

[LinkedIn](https://www.linkedin.com/in/angel-martin-trejo/) · [GitHub](https://github.com/angelmttt)
