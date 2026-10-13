# Ruta de Prospección — Registro de versiones

Este archivo documenta las versiones publicadas de la aplicación.

## Regla de versiones

- La versión visible se define en `<meta name="app-version" content="N">` dentro de `index.html`.
- Cada modificación publicada debe incrementar el número entero de la versión (1, 2, 3...) y actualizar este registro en el mismo commit.
- La etiqueta se muestra antes de iniciar sesión y en la barra superior después de iniciar sesión. Sirve para comprobar qué versión del HTML está cargando el navegador.
- La versión visible confirma qué código HTML se está sirviendo; no sustituye una prueba funcional de cada cambio ni garantiza por sí sola que todos los servicios externos funcionen.

## Versión 2 — 2026-10-13

- Se añade al inicio de los archivos HTML un encabezado de comentario con la versión actual, la fecha, el historial y la regla para incrementar versiones.
- Se actualiza el número de versión de la aplicación a 2; la pantalla de acceso mostrará «Versión 2» y la barra superior «v2».
- Se conserva la versión 1 como línea base del historial.

## Versión 1 — 2026-10-13

- Se establece el control inicial de versiones para Ruta de Prospección.
- Se añade la etiqueta visible de versión en el acceso y en la barra superior.
- Se adopta este estado publicado como línea base para futuras modificaciones.
