# install-resolve

Instalador de DaVinci Resolve y Resolve Studio para Linux: un solo script en Python
(`install-resolve`, sin dependencias externas) que encuentra el `.zip`/`.run` oficial en
`~/Downloads`, instala dependencias con `apt`/`dnf`/`pacman`, corre el instalador oficial,
limpia incompatibilidades y revisa GPU/OpenCL. Repo público; uso en `README.md`.
`CLAUDE.md` es un hard link de este archivo.

## Reglas

- Python 3.8+ y solo biblioteca estándar: no agregues dependencias.
- lizeth es un contenedor sin escritorio ni GPU para Resolve: no corras el instalador ahí.
  Verifica con `python3 -m py_compile install-resolve` y `./install-resolve --help`.
- El script pide root temporalmente para el display de X11/XWayland y lo revoca al terminar;
  cualquier cambio en esa parte debe conservar la revocación aunque algo falle.
- Mensajes y salida del script en inglés (el repo es público y en inglés).
- Commits en inglés, minúsculas, cortos, como el historial (`fix wayland installer display`).
  Sin trailers de atribución.
