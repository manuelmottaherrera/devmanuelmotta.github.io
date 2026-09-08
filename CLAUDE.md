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
   - **Excepción deliberada:** el dropdown de descarga del CV no usa i18n. Sus dos grupos (`English` / `Español`, clase `.btn-dropdown-group`) y sus cuatro enlaces están rotulados cada uno en el idioma del PDF que sirven, fijos, para que se vea en qué idioma está cada documento sin importar cómo esté el toggle del sitio. No agregarles `data-i18n`.

3. **Tema, menús y nav** (IIFE + código suelto): selector de tres estados `light | dark | system`, guardado en `localStorage.theme` (default `'system'`), con listener de `prefers-color-scheme` que solo actúa en modo `system`. El menú de tema y el dropdown de CV se cierran mutuamente y con click en `document`; cada handler propio hace `e.stopPropagation()`. Si se agrega otro dropdown, seguir ese mismo patrón o los tres empezarán a pelearse.

Secciones del sitio, en orden: nav, hero, about, experience, skills, services, projects, education, contact, footer. Todas marcadas con comentarios `<!-- ===== NOMBRE ===== -->`.

## CSS

Un solo archivo, organizado por bloques `/* ===== NOMBRE ===== */` que espejan las secciones del HTML. Colores, sombras, radio y transición son variables CSS en `:root`; el modo oscuro **solo redefine variables** en `[data-theme="dark"]` (más un par de overrides puntuales para `.nav` y `.footer`). Regla práctica: no escribir colores literales en reglas nuevas — usar `var(--…)`, o el modo oscuro quedará roto.

Breakpoints responsive: 768px y 480px, al final del archivo.

## CV (`resume/`)

Cuatro variantes — **idioma × formato** — con fuente LaTeX propia, compiladas a PDF y **commiteadas** al repo (GitHub Pages las sirve directo):

| Fuente | PDF | Idioma | Formato |
|---|---|---|---|
| `resume.tex` | `resume.pdf` | Inglés | Visual: color + iconos (`fontawesome5`) |
| `resume-ats.tex` | `resume-ats.pdf` | Inglés | ATS: sin color, sin iconos |
| `resume-es.tex` | `resume-es.pdf` | Español | Visual |
| `resume-ats-es.tex` | `resume-ats-es.pdf` | Español | ATS |

Las versiones ATS son planas **a propósito** (sin color, sin iconos, sin formato exótico), para que las lean los Applicant Tracking Systems. Mantenerlas así.

`resume.txt` es un volcado en texto plano del CV en inglés; no está enlazado desde el sitio.

**Las traducciones son pares que hay que mantener sincronizados**: al editar un `.tex` en un idioma, aplicar el mismo cambio en su gemelo. Única divergencia deliberada de maquetación: `resume-es.tex` usa `labelwidth=9em` en la lista de habilidades (contra `7em` del inglés) porque etiquetas como “Bases de Datos” no caben en la columna original.

Los dos PDF visuales existen además **duplicados con nombre presentable**, que son los que enlaza el sitio (el nombre se ve al guardar el archivo):

- `Manuel_Motta_Resume.pdf` — copia byte a byte de `resume.pdf`
- `Manuel_Motta_Hoja_de_Vida.pdf` — copia byte a byte de `resume-es.pdf`

Al regenerar un CV visual hay que refrescar también su copia, o el sitio seguirá sirviendo el PDF viejo.

### Compilar

`pdflatex` no está instalado en esta máquina; se compila con la imagen Docker de TeX Live (ya descargada localmente). El `-u` evita que los PDF queden como root:

```bash
docker run --rm -u "$(id -u):$(id -g)" -v "$PWD/resume":/work -w /work texlive/texlive:latest \
  bash -c 'for f in resume resume-ats resume-es resume-ats-es; do pdflatex -interaction=nonstopmode "$f.tex"; done'
```

Dos pasadas si se agregan referencias cruzadas. El número de páginas sale del log: `grep "Output written" resume/*.log` — hoy las cuatro variantes son de 2 páginas. Para revisar el resultado sin abrir un visor: `pdftoppm -png -r 80 resume/resume-es.pdf /tmp/out` (poppler sí está instalado).

Los artefactos de LaTeX (`.aux`, `.log`, `.out`, …) están en `.gitignore`; borrarlos después de compilar.

Detalle heredado: hay un archivo trackeado llamado `resume/ ` (un solo espacio como nombre), duplicado de `resume.pdf`, subido por accidente. No sirve para nada; borrarlo solo si el usuario lo pide.

## Convenciones

Commits convencionales en inglés: `type(scope): description` (`feat`, `fix`, `docs`, `style`, `refactor`, `chore`). El repo hoy usa mensajes en inglés, a diferencia de los proyectos de TYSE.

El contenido del sitio es bilingüe; el texto fuente del HTML va en inglés y su traducción en `translations.es`. Las páginas legales están íntegramente en español.
