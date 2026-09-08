# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este repo

Sitio personal/portafolio de Manuel Motta, publicado con **GitHub Pages** desde la rama `main` del repo `manuelmottaherrera/devmanuelmotta.github.io`. El `CNAME` apunta el sitio a `devmanuelmotta.com`.

No hay build system, ni framework, ni dependencias npm: HTML + CSS + JS vanilla servido tal cual. **Hacer push a `main` es desplegar.** Por eso `main` es rama protegida por convención del usuario — pedir permiso explícito antes de `git commit` y antes de `git push`.

## Desarrollo y verificación

No hay tests ni linter. Para ver cambios:

```bash
python3 -m http.server 8000   # luego abrir http://localhost:8000
```

Abrir `index.html` con `file://` también funciona para maquetación, pero rompe rutas absolutas y el widget externo.

Al tocar `index.html` o `css/style.css`, verificar siempre **las cuatro combinaciones**: tema claro/oscuro × idioma EN/ES. Es donde se rompen las cosas en este repo.

## Estructura del contenido

| Archivo | Rol |
|---|---|
| `index.html` | El sitio entero: markup + i18n + JS de UI, todo inline (~730 líneas) |
| `css/style.css` | Único stylesheet; tokens en `:root` y override en `[data-theme="dark"]` |
| `privacy-policy.html`, `terms-of-service.html`, `data-deletion.html` | Páginas legales **standalone** en español, con `<style>` propio. No usan `css/style.css` ni comparten nav/footer. Existen para la revisión de app de Meta (WhatsApp Business) |
| `widget-test.html` | Página suelta de prueba del widget de chat de tyse-helpdesk (`api.devmanuelmotta.com/widget/widget.js`) |
| `resume/` | Fuentes LaTeX + PDFs compilados del CV |

## index.html: los tres subsistemas de JS

Todo el JS vive inline en `index.html`. Al modificar cualquiera de estos, entender que interactúan entre sí:

1. **Anti-FOUC de tema** (`<head>`, línea ~26): script bloqueante que lee `localStorage.theme` y estampa `data-theme` en `<html>` antes del primer paint. Debe seguir siendo síncrono y estar en el `<head>`, o el tema oscuro parpadea en blanco al cargar.

2. **i18n** (IIFE al final): el idioma base es **inglés, escrito directamente en el HTML**. El español vive en el objeto `translations.es` dentro del script. El mecanismo:
   - `data-i18n="clave"` → reemplaza `textContent`.
   - `data-i18n-html="clave"` → reemplaza `innerHTML` (para texto con `<br>`, `<strong>`, o SVG embebido).
   - `collectOriginals()` guarda el texto inglés del DOM al cargar; volver a EN restaura desde ahí. **Por eso no existe un diccionario `en`**: agregar texto nuevo significa escribirlo en inglés en el HTML con su atributo `data-i18n*`, y añadir solo la clave española a `translations.es`. Una clave sin entrada en `es` simplemente se queda en inglés al conmutar, sin error visible.
   - Preferencia persistida en `localStorage.lang` (default `'en'`).

3. **Tema, menús y nav** (IIFE + código suelto): selector de tres estados `light | dark | system`, guardado en `localStorage.theme` (default `'system'`), con listener de `prefers-color-scheme` que solo actúa en modo `system`. El menú de tema y el dropdown de CV se cierran mutuamente y con click en `document`; cada handler propio hace `e.stopPropagation()`. Si se agrega otro dropdown, seguir ese mismo patrón o los tres empezarán a pelearse.

Secciones del sitio, en orden: nav, hero, about, experience, skills, services, projects, education, contact, footer. Todas marcadas con comentarios `<!-- ===== NOMBRE ===== -->`.

## CSS

Un solo archivo, organizado por bloques `/* ===== NOMBRE ===== */` que espejan las secciones del HTML. Colores, sombras, radio y transición son variables CSS en `:root`; el modo oscuro **solo redefine variables** en `[data-theme="dark"]` (más un par de overrides puntuales para `.nav` y `.footer`). Regla práctica: no escribir colores literales en reglas nuevas — usar `var(--…)`, o el modo oscuro quedará roto.

Breakpoints responsive: 768px y 480px, al final del archivo.

## CV (`resume/`)

Dos variantes con fuente LaTeX propia, compiladas a PDF y **commiteadas** al repo (GitHub Pages las sirve directo):

- `resume.tex` → `resume.pdf`: versión visual, con color e iconos (`fontawesome5`).
- `resume-ats.tex` → `resume-ats.pdf`: versión plana para sistemas ATS — sin color, sin iconos, sin formato exótico. Mantenerla así a propósito.
- `resume.txt`: volcado en texto plano.

`Manuel_Motta_Resume.pdf` es una **copia byte a byte de `resume.pdf`** con nombre presentable; es la que enlaza el dropdown del hero. Al regenerar el CV visual hay que actualizar las dos, o el sitio servirá el PDF viejo.

Compilar (`pdflatex` no está instalado en esta máquina — instalarlo o compilar en otro lado):

```bash
cd resume && pdflatex resume.tex && pdflatex resume-ats.tex
```

Los artefactos de LaTeX (`.aux`, `.log`, `.out`, …) están en `.gitignore`.

Detalle heredado: hay un archivo trackeado llamado `resume/ ` (un solo espacio como nombre), duplicado de `resume.pdf`, subido por accidente. No sirve para nada; borrarlo solo si el usuario lo pide.

## Convenciones

Commits convencionales en inglés: `type(scope): description` (`feat`, `fix`, `docs`, `style`, `refactor`, `chore`). El repo hoy usa mensajes en inglés, a diferencia de los proyectos de TYSE.

El contenido del sitio es bilingüe; el texto fuente del HTML va en inglés y su traducción en `translations.es`. Las páginas legales están íntegramente en español.
