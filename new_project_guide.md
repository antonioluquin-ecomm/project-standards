# Guía de Proyecto Nuevo — Paso a Paso

**Versión:** 1.0  
**Alcance:** Proyectos del ecosistema: HTML/CSS/Vanilla JS + GAS + Google Sheets + GitHub Pages

---

## Antes de empezar — decisiones previas

Responder estas preguntas antes de tocar el teclado:

| Decisión | Opciones |
|----------|---------|
| ¿Tiene backend? | GAS + Sheets / Sin backend (estático puro) |
| ¿Tiene autenticación? | Sí (ver `login_standard.md`) / No (acceso por URL) |
| ¿Es multi-store? | Sí (prefijo por tienda en Script Properties) / No |
| ¿Tipo de frontend? | A: multi-página con sidebar / B: SPA / C: módulos standalone |
| ¿Nombre del repositorio? | kebab-case, sin espacios (`mi-proyecto`, no `Mi Proyecto`) |

Si la idea no está definida: trabajarla en ChatGPT primero para convertirla en alcance claro antes de crear archivos.

---

## Paso 1 — Crear la carpeta y el repo local

```powershell
# Desde C:\Users\gluna\Documents\Repos
mkdir nombre-del-proyecto
cd nombre-del-proyecto
git init
git branch -M main
```

---

## Paso 2 — Archivos base desde el día 1

Crear estos archivos antes de escribir la primera línea de lógica:

### `.gitignore`

```
# Credenciales y configuración local
.env
*.env.local
config.local.js

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
*.suo
*.user

# Node (si aplica)
node_modules/
dist/

# Archivos de Office (van en OneDrive, no en el repo)
*.docx
*.xlsx
*.pdf
```

### `README.md`

```markdown
# Nombre del Proyecto

Descripción breve (1-2 líneas).

## Qué es

Para qué sirve, quiénes lo usan.

## Stack

- Frontend: HTML/CSS/Vanilla JS · GitHub Pages
- Backend: Google Apps Script (si aplica)
- Base de datos: Google Sheets (si aplica)

## Cómo usar

Instrucciones mínimas.

## Cómo validar

Qué verificar para confirmar que funciona.
```

### `CHANGELOG.md`

```markdown
# Changelog

## [1.0.0] — YYYY-MM-DD

- Versión inicial
```

### `CLAUDE.md`

Copiar la estructura de un `CLAUDE.md` existente (ver cualquier proyecto del ecosistema) y adaptar:

- Sección de reglas específicas del proyecto
- Stack específico
- Freeze zones
- Referencia a `../project-standards/`
- Sección estándar "Documentación compartida"

Ver `../project-standards/project_workflow_template.md` para la estructura recomendada.

### `AGENTS.md`

```markdown
# AGENTS.md — [Nombre del Proyecto]

Alias de [`CLAUDE.md`](CLAUDE.md) para Codex y otros asistentes.

Las instrucciones del proyecto viven en `CLAUDE.md`. Leer ese archivo y los
documentos maestros de `../project-standards/` antes de proponer cambios.

## Documentación estándar compartida

La documentación estándar compartida se encuentra en `../project-standards/`:

- [`../project-standards/ai_rules.md`](../project-standards/ai_rules.md) — reglas de colaboración con IA
- [`../project-standards/style_guide.md`](../project-standards/style_guide.md) — colores, tipografía, componentes CSS, Git
- [`../project-standards/apps_script_standards.md`](../project-standards/apps_script_standards.md) — convenciones GAS
- [`../project-standards/google_sheets_standards.md`](../project-standards/google_sheets_standards.md) — estructura de Sheets
- [`../project-standards/login_standard.md`](../project-standards/login_standard.md) — patrón de autenticación
- [`../project-standards/application_shell.md`](../project-standards/application_shell.md) — shell de aplicación

### Entorno de trabajo

- El desarrollo se realiza desde `C:\Users\gluna\Documents\Repos`
- No usar OneDrive/SharePoint como carpeta de desarrollo
- GitHub es la fuente principal para versionado y colaboración
- OneDrive/SharePoint queda reservado para documentación funcional: archivos compartidos, PDFs, presentaciones, actas e imágenes
```

### `docs/project_workflow.md`

Copiar `../project-standards/project_workflow_template.md` y personalizar §7.1 (Freeze zones) y §12.5 (Convenciones).

### `docs/roadmap.md`

```markdown
# Roadmap — [Nombre del Proyecto]

## Próximos pasos

- [ ] [tarea 1]
- [ ] [tarea 2]

## Backlog

- [idea futura 1]
```

### Primer commit

```powershell
git add .gitignore README.md CHANGELOG.md CLAUDE.md AGENTS.md docs/project_workflow.md docs/roadmap.md
git status --short
git commit -m "chore: estructura inicial del proyecto"
```

---

## Paso 3 — Crear el repositorio en GitHub y primer push

1. Ir a [github.com/antonioluquin-ecomm](https://github.com/antonioluquin-ecomm)
2. **New repository** → nombre igual a la carpeta local (kebab-case)
3. Sin README ni .gitignore (ya los tenemos localmente)
4. Copiar la URL HTTPS del repo

```powershell
git remote add origin https://github.com/antonioluquin-ecomm/nombre-del-proyecto.git
git push -u origin main
```

---

## Paso 4 — Configurar GitHub Pages

1. En el repo de GitHub: **Settings > Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / `/ (root)`
4. **Save**

La URL queda disponible en:
```
https://antonioluquin-ecomm.github.io/nombre-del-proyecto/
```

Propagación inicial: 2–5 minutos. Verificar que `index.html` o la página principal carga correctamente.

> Para proyectos con `index.html` en la raíz: carga directo. Para proyectos con subdirectorios, asegurarse de que la ruta raíz tenga un `index.html`.

---

## Paso 5 — Configurar el preview local (`.claude/launch.json`)

Crear `.claude/launch.json` para poder usar el preview de Claude Code durante el desarrollo:

### Opción A — Python (más simple, sin dependencias)

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "nombre-del-proyecto",
      "runtimeExecutable": "python",
      "runtimeArgs": ["-m", "http.server", "PUERTO"],
      "port": PUERTO
    }
  ]
}
```

Reemplazar `PUERTO` con un número libre entre 3000 y 9000. Usar un puerto distinto por proyecto para poder correr varios en simultáneo.

| Proyecto | Puerto |
|----------|--------|
| vtex-control-center | 3500 |
| project-control-center | 3577 |
| correos-transaccionales | 8723 |
| [nuevo proyecto] | [elegir uno libre] |

### Opción B — Node.js con servidor estático

Si el proyecto ya usa Node.js (ej: vtex-bookmarklets):

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "nombre-del-proyecto",
      "runtimeExecutable": "node",
      "runtimeArgs": [".tmp-static-server.js"],
      "port": PUERTO,
      "autoPort": true
    }
  ]
}
```

Agregar `.claude/` al `.gitignore` si el `launch.json` tiene configuración local que no se quiere versionar. En este ecosistema se versiona porque es parte del setup del proyecto.

---

## Paso 6 — Setup del GAS (si aplica)

Si el proyecto tiene backend en Google Apps Script, seguir `../project-standards/gas_setup_template.md`.

Documentar el setup específico del proyecto en `docs/gas-setup.md`:
- Lista de archivos `.gs` a copiar
- Script Properties requeridas (nombres exactos)
- Función de bootstrap si existe
- URL del Web App (no en el repo — solo en la documentación operativa o `localhost`)

---

## Paso 7 — Actualizar `project-standards/ai_rules.md`

Agregar el nuevo proyecto a la lista de la línea `**Proyectos:**` en `ai_rules.md`.

```
**Proyectos:** ... · [Nombre del Proyecto] · Herramientas internas
```

---

## Checklist de lanzamiento

```
[ ] Carpeta creada en C:\Users\gluna\Documents\Repos\
[ ] git init + branch main
[ ] .gitignore con credenciales, OS, editor
[ ] README.md con descripción, stack, cómo usar
[ ] CHANGELOG.md con v1.0.0 inicial
[ ] CLAUDE.md con reglas específicas + sección estándar
[ ] AGENTS.md con sección estándar
[ ] docs/project_workflow.md (desde template, §7.1 y §12.5 personalizados)
[ ] docs/roadmap.md con próximos pasos
[ ] Primer commit: "chore: estructura inicial del proyecto"
[ ] Repo creado en github.com/antonioluquin-ecomm
[ ] git remote add origin + git push -u origin main
[ ] GitHub Pages configurado (Settings > Pages > main / root)
[ ] GitHub Pages carga correctamente (esperar 2-5 min)
[ ] .claude/launch.json con puerto libre
[ ] Preview local funciona desde Claude Code
[ ] GAS setup completado (si aplica) → docs/gas-setup.md
[ ] ai_rules.md actualizado con el nuevo proyecto
```
