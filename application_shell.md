# Application Shell — Estándar de Layout y Navegación

**Versión:** 1.5.0  
**Fecha:** 2026-06-24  
**Autor:** Gabriel Luna — Productos Web  
**Alcance:** VTEX Control Center · Commerce Hub · Project Control Center · Customer Service Control Center · Marketplace Portal · Herramientas internas

---

## Índice

1. [Propósito](#1-propósito)
2. [Anatomía del shell](#2-anatomía-del-shell)
3. [Anti-flash de tema](#3-anti-flash-de-tema)
4. [Sidebar](#4-sidebar)
5. [Topbar](#5-topbar)
6. [Panel de usuario](#6-panel-de-usuario)
7. [Sistema de temas](#7-sistema-de-temas)
8. [Version badge y popover de changelog](#8-version-badge-y-popover-de-changelog)
9. [Estados globales](#9-estados-globales)
10. [Comportamientos comunes](#10-comportamientos-comunes)
11. [Variantes por tipo de proyecto](#11-variantes-por-tipo-de-proyecto)
12. [Plantilla HTML base](#12-plantilla-html-base)
13. [Checklist de conformidad](#13-checklist-de-conformidad)

---

## 1. Propósito

Este documento define el **shell de aplicación**: el conjunto de elementos estructurales y comportamientos que rodean el contenido de cada módulo y que son comunes a todos los proyectos del ecosistema.

**No cubre** estilos de componentes aislados (tablas, botones, formularios) — eso corresponde a `style_guide.md`.  
**No cubre** reglas de colaboración con IA — eso corresponde a `ai_rules.md`.

Un proyecto nuevo que respete este documento se verá y se comportará como parte del mismo sistema desde el primer día.

---

## 2. Anatomía del shell

```
┌─────────────────────────────────────────────────────────┐
│  SIDEBAR                    CONTENT AREA                │
│  ┌──────────────┐           ┌───────────────────────┐   │
│  │ Brand        │           │ Topbar                │   │
│  │ (logo·ver·   │           │ (breadcrumb · user    │   │
│  │  nombre app) │           │  chip · acciones)     │   │
│  ├──────────────┤           ├───────────────────────┤   │
│  │ Store        │           │                       │   │
│  │ selector     │           │  <main>               │   │
│  │ (si aplica)  │           │                       │   │
│  ├──────────────┤           │                       │   │
│  │ Nav          │           │                       │   │
│  │ (grupos /    │           │                       │   │
│  │  ítems)      │           │                       │   │
│  ├──────────────┤           │                       │   │
│  │ ▲ spacer     │           │                       │   │
│  ├──────────────┤           └───────────────────────┘   │
│  │ Footer       │                                        │
│  │ (usuario·    │                                        │
│  │  rol·logout) │                                        │
│  └──────────────┘                                        │
└─────────────────────────────────────────────────────────┘
```

**Grilla CSS canónica:**

```css
.app-layout {
  display: grid;
  grid-template-columns: var(--sidebar-width, 220px) 1fr;
  min-height: 100vh;
}
```

En mobile (≤ 768 px) el sidebar colapsa como drawer con overlay.

---

## 3. Anti-flash de tema

**Obligatorio. Debe ser el primer `<script>` del `<head>`, antes de cualquier CSS.**

Carga el tema guardado en `localStorage` y lo aplica al `<html>` antes de que el navegador pinte.

```html
<script>(function(){
  var k = '<APP_PREFIX>_theme';
  var t = localStorage.getItem(k) || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  document.documentElement.setAttribute('data-theme', t);
})();</script>
```

### Convención de clave localStorage

| Proyecto | Clave |
|----------|-------|
| VTEX Control Center | `vtex_cc_theme` |
| Commerce Hub | `ch_theme` |
| Project Control Center | `pcc_theme` |
| Customer Service CC | `cs_theme` |
| Marketplace Portal | `mp_theme` |
| Proyectos nuevos | `<siglas>_theme` — sin espacios, en minúsculas |

**Nunca usar** `theme` sin prefijo (colisión entre proyectos en el mismo dominio).

---

## 4. Sidebar

### 4.1 Estructura completa

```html
<aside class="sidebar" id="sidebar">

  <!-- 1. Brand -->
  <div class="sidebar-brand">
    <div class="brand-icon">XX</div>          <!-- Siglas del proyecto, 2–3 chars -->
    <div>
      <div class="brand-name">Nombre de App</div>
      <div class="brand-meta" id="sidebarVersionBtn" title="Ver historial de cambios">
        <span id="sidebarVersion">v…</span>
        <span>· Equipo / Área</span>
      </div>
    </div>
    <div class="sidebar-version-popover" id="versionPopover" style="display:none;"></div>
  </div>

  <!-- 2. Store selector (solo proyectos multi-tienda) -->
  <div class="sidebar-store">
    <div class="sidebar-section-label">Tienda</div>
    <div class="store-selector" id="storeSelectorHome"></div>
    <div id="storeNotice" class="sidebar-store-notice" style="display:none;">
      ⚠ Seleccioná una tienda para comenzar.
    </div>
  </div>

  <!-- 3. Navegación -->
  <nav class="sidebar-nav" id="sidebarNav">
    <!-- grupos y/o ítems directos según la complejidad del proyecto -->
  </nav>

  <!-- 4. Spacer -->
  <div class="sidebar-spacer"></div>

  <!-- 5. Footer de usuario (renderizado por JS en DOMContentLoaded) -->
  <div class="sidebar-footer" id="sidebarFooter"></div>

</aside>
```

### 4.2 CSS canónico del sidebar

Los valores a continuación son el estándar. Usarlos en todos los proyectos sin variaciones estructurales. Lo único variable es `--primary` y `--primary-hover` (identidad del proyecto, ver §4.7).

```css
/* ── Contenedor ───────────────────────────────────────────── */
.sidebar {
  width: 224px;             /* --sidebar-w si se usa variable */
  flex-shrink: 0;
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-line);
  display: flex; flex-direction: column;
  position: sticky; top: 0; height: 100vh;  /* o fixed+overflow-y:auto según Tipo */
}

/* ── Brand ─────────────────────────────────────────────────── */
.sidebar-brand {
  display: flex; align-items: center; gap: 10px;
  padding: 18px 16px 14px;
  border-bottom: 1px solid var(--sidebar-line);
  flex-shrink: 0;
}
.brand-icon {
  width: 34px; height: 34px; border-radius: 9px;
  background: linear-gradient(135deg, var(--primary), var(--primary-hover));
  color: #fff; font-weight: 800; font-size: 13px; letter-spacing: -.5px;
  display: inline-flex; align-items: center; justify-content: center;
  flex-shrink: 0;
  box-shadow: 0 2px 8px color-mix(in srgb, var(--primary) 30%, transparent);
}
.brand-name { font-size: 13px; font-weight: 700; letter-spacing: -.2px; }
.brand-meta {
  font-size: 10px; color: var(--muted); font-family: var(--mono);
  display: flex; align-items: center; gap: 4px; margin-top: 2px;
  cursor: pointer; transition: color .15s; user-select: none;
}
.brand-meta:hover { color: var(--text); }

/* ── Version popover ───────────────────────────────────────── */
.sidebar-version-popover {
  position: fixed; top: 58px; left: 232px;
  background: var(--card); border: 1px solid var(--sidebar-line);
  border-radius: var(--radius-sm); box-shadow: var(--shadow-lg);
  padding: 14px 16px; min-width: 280px; max-width: 340px;
  max-height: 70vh; overflow-y: auto;
  z-index: 1000;            /* ≥1000 obligatorio para no quedar bajo contenido */
  color: var(--text); line-height: 1.5;
}

/* ── Nav ───────────────────────────────────────────────────── */
.nav { display: flex; flex-direction: column; gap: 1px; }
.nav-item {
  display: flex; align-items: center; gap: 9px; padding: 7px 10px;
  border-radius: var(--radius-sm); color: var(--text2); font-size: 13px;
  font-weight: 500; cursor: pointer; user-select: none;
  border: 1px solid transparent; transition: all .15s;
}
.nav-item:hover  { background: color-mix(in srgb, var(--text) 5%, transparent); color: var(--text); }
.nav-item.active {
  background: var(--primary-soft); color: var(--primary); font-weight: 600;
  border-color: color-mix(in srgb, var(--primary) 20%, transparent);
}
.nav-icon { font-size: 15px; width: 18px; text-align: center; opacity: .7; }
.nav-item.active .nav-icon { opacity: 1; }

/* ── Spacer + footer ───────────────────────────────────────── */
.sidebar-spacer { flex: 1; }
.sidebar-footer {
  padding: 10px 8px;
  border-top: 1px solid var(--sidebar-line);
  flex-shrink: 0;
}
```

> **Proyectos con íconos SVG** (como Commerce Hub): `.nav-icon` lleva `width: 15px; height: 15px` en lugar de `font-size`. Los ítems con íconos Unicode usan `font-size: 15px; width: 18px`.  
> **No variar** los valores de padding, gap, border-radius ni font-size entre proyectos — son lo que hace que el sidebar se vea idéntico.

### 4.3 Brand (referencia de contenido)

| Elemento | Descripción |
|----------|-------------|
| `.brand-icon` | Siglas en mayúsculas (2–3 caracteres). Fondo `--primary`, texto blanco. |
| `.brand-name` | Nombre completo de la aplicación. |
| `.brand-meta` | Versión (`vX.Y.Z`) + separador `·` + equipo/área. Clickeable → abre popover de changelog. |
| `#versionPopover` | Lista de entradas del `CHANGELOG`. Siempre `display:none` inicial. |

### 4.4 Store selector

Solo en proyectos multi-tienda. Renderizado por JS. Cada tienda es un `<button class="store-btn">`.  
El botón activo lleva clase `.active`. Si no hay tienda seleccionada, mostrar `#storeNotice`.

### 4.5 Nav groups (proyectos con muchos módulos)

```html
<div class="sidebar-nav-group">
  <button class="sidebar-group-header" data-filter="catalogo" type="button">
    Catálogo
    <span class="sidebar-group-count">10</span>
  </button>
  <a class="sidebar-nav-item" href="modules/catalogo/categorias.html">
    <span class="sidebar-dot available"></span>Categorías
  </a>
  <!-- … -->
</div>
```

- `data-filter` coincide con el `data-section` de los módulos en el contenido.
- `.sidebar-dot` indica el estado del módulo: `available` · `pending` · `disabled`.
- El grupo Admin se oculta por defecto (`display:none`) y se muestra solo para `role === 'admin'`.

### 4.6 Nav items directos (proyectos SPA con pocas secciones)

```html
<div class="nav-item" data-page="inicio" onclick="showPage('inicio')">
  <svg class="nav-icon">…</svg> Inicio
</div>
```

- `data-page` permite que `showPage()` marque el ítem activo con `.active`.
- Siempre `type="button"` en botones, `href` en links de navegación real.

### 4.7 Sidebar footer — área de usuario

Renderizado por JS tras cargar la sesión (no hardcodeado en HTML para evitar flash de datos vacíos):

```js
function renderSidebarUser() {
  const footer = document.getElementById('sidebarFooter');
  if (!footer || document.getElementById('sidebar-user-info')) return;
  const u = SESSION.usuario;   // SESSION de auth.js (modelo RBAC)
  if (!u) return;
  const rolAdmin = Number(u.id_rol) === 1;
  const roleLabel = u.nombre_rol || (rolAdmin ? 'Administrador' : 'Rol ' + u.id_rol);
  const roleCls   = rolAdmin ? 'auth-role-admin' : 'auth-role-agente';
  const info = document.createElement('div');
  info.id = 'sidebar-user-info';
  info.innerHTML =
    '<div class="sidebar-user-name">' + escapeHtml(u.nombre || u.email) + '</div>'
    + '<div class="sidebar-user-meta"><span class="auth-chip-role ' + roleCls + '">' + roleLabel + '</span></div>'
    + '<div class="sidebar-user-actions">'
    + '<button class="theme-toggle" onclick="toggleTheme()" type="button">☾ Modo oscuro</button>'
    + '<button class="sidebar-action-btn" onclick="showChangePasswordModal()" type="button">Cambiar contraseña</button>'
    + '<button class="sidebar-action-btn danger" onclick="authLogout()" type="button">Cerrar sesión</button>'
    + '</div>';
  footer.insertBefore(info, footer.firstChild);
  _updateThemeToggles(document.documentElement.getAttribute('data-theme') || 'light');
}
```

**Orden canónico del sidebar footer** (de arriba hacia abajo):

1. Nombre del usuario (`u.name` o `u.email` como fallback)
2. Badge de rol (nombre real, no hardcodeado — ver §6.2)
3. Theme toggle (`.theme-toggle`) — **aquí y solo aquí**
4. "Cambiar contraseña"
5. "Cerrar sesión"

> No implementar el theme toggle como chip separado en el sidebar ni en el topbar. Un botón "Tema" fuera del footer de usuario rompe la coherencia entre proyectos.

**El theme toggle vive en el sidebar footer, no en el topbar.** Ver §5 para la excepción SPA (sin sidebar-footer).

### 4.8 Identidad visual por proyecto

Cada proyecto tiene un color primario (`--primary`) que diferencia su brand visualmente. El `.brand-icon` usa ese color como fondo. **No compartir el mismo `--primary` entre dos proyectos.**

| Proyecto | Siglas | Nota de color |
|----------|--------|--------------|
| VTEX Control Center | VCC | Azul institucional VTEX |
| Commerce Hub | CH | Definido en `src/css/main.css` |
| Project Control Center | PCC | Definido en `src/css/main.css` |
| Customer Service CC | CS | Definido en `src/css/main.css` |
| Proyectos nuevos | 2–3 chars | Elegir `--primary` que no colisione con los anteriores |

El color de acento define el tono global de la aplicación: botón primario, estados activos, bordes de foco, badge Administrador. Todo lo demás hereda de las variables CSS compartidas (`style_guide.md`).

---

## 5. Topbar

### 5.1 CSS canónico del topbar

```css
.topbar {
  height: 50px;
  border-bottom: 1px solid var(--line);
  display: flex; align-items: center;
  padding: 0 22px; gap: 10px;
  background: var(--card);
  position: sticky; top: 0; z-index: 40;
  flex-shrink: 0;
}
.topbar-brand   { font-size: 13px; font-weight: 600; color: var(--muted); }
.topbar-sep     { color: var(--muted); font-size: 12px; }
.topbar-crumb   { font-size: 13px; color: var(--text); font-weight: 600; }
.topbar-actions { display: flex; gap: 6px; align-items: center; margin-left: auto; }
```

> **Altura fija 50px.** El topbar no crece con su contenido. Nunca poner H1 ni subtítulo dentro del topbar — eso rompe la altura y la consistencia entre páginas.

### 5.2 Topbar estándar (Tipo B y C — con sidebar)

Aplica a SPA y a multi-página con sidebar. El topbar muestra solo el breadcrumb y los botones de acción:

```html
<div class="topbar">
  <span class="topbar-brand">Nombre de App</span>
  <span class="topbar-sep">›</span>
  <span class="topbar-crumb">Sección actual</span>  <!-- o id="topbar-crumb" si se actualiza por JS -->
  <div class="topbar-actions">
    <!-- Botones de acción global (solo si los hay) -->
  </div>
</div>
```

En SPA, `topbar-crumb` se actualiza por JS al cambiar de sección:

```js
function showPage(page) {
  document.getElementById('topbar-crumb').textContent = PAGE_TITLES[page] || page;
}
```

En multi-página con sidebar (Tipo C), el crumb es texto estático en cada HTML.

### 5.3 Cabecera de página — `.page-header`

El título (H1) y subtítulo de cada módulo viven **dentro de `<main>`**, no en el topbar:

```html
<main class="container">
  <div class="page-header">
    <h1>Nombre del módulo</h1>
    <p class="subtitle">Descripción breve del propósito del módulo.</p>  <!-- opcional -->
  </div>
  <!-- contenido del módulo -->
</main>
```

```css
.page-header        { padding: 24px 0 20px; }
.page-header h1     { font-size: 22px; font-weight: 700; letter-spacing: -.02em; margin: 0; }
.page-header .subtitle { font-size: 13px; color: var(--muted); margin-top: 4px; }
```

### 5.4 Topbar de módulo sin sidebar (Tipo A — multi-página puro)

Solo aplica a proyectos donde los módulos **no** tienen sidebar (ej. VTEX Control Center). En ese caso el topbar sí puede contener el H1 porque no hay breadcrumb de sidebar que dé contexto:

```html
<header class="topbar">
  <div>
    <p class="eyebrow">Área / Sección</p>
    <h1>Nombre del módulo</h1>
    <p class="subtitle">Descripción breve.</p>
  </div>
  <a class="button secondary" href="../../index.html">← Volver al inicio</a>
</header>
<div id="storeRow" class="store-row"></div>
```

> Esto es la **excepción**, no la regla. Si el módulo tiene sidebar, usar §5.2 + §5.3.

### 5.5 Breadcrumb — formato canónico

```
Nombre de App  ›  Sección actual
```

- Solo dos niveles. No usar rutas de carpetas ni slugs técnicos.
- El separador es `›` (no `/`, no `>`).
- `.topbar-brand` en color `var(--muted)` (más suave); `.topbar-crumb` en `var(--text)` (énfasis).

---

## 6. Panel de usuario

### 6.1 Ubicación según tipo de proyecto

| Tipo | Dónde aparece |
|------|--------------|
| Multi-página (index) | Sidebar footer (renderizado por JS) |
| Multi-página (módulo) | `#storeRow` → chip de usuario |
| SPA con sidebar | Sidebar footer (renderizado por JS) |
| SPA sin sidebar | Topbar |

### 6.2 Información mostrada

| Elemento | Descripción |
|----------|-------------|
| Nombre | `user.name` o `user.email` como fallback |
| Rol | Badge con el **nombre real del rol** (`Administrador`, `Ecommerce`, o cualquier rol personalizado) — se resuelve por `nombre_rol` de la sesión, no se hardcodea. El Administrador (`id_rol===1`) usa estilo destacado; el resto, neutro. |
| Cambio de contraseña | Abre modal — siempre disponible |
| Cerrar sesión | Llama `authLogout()` — siempre disponible |
| Theme toggle | Botón `.theme-toggle` — en sidebar footer o topbar |

> Con el modelo de roles flexible (ver `apps_script_standards.md §7` y `../Proyectos/CLAUDE.md`), el badge **no** puede asumir un set fijo de roles. `isAdmin()` es siempre `id_rol===1`; todo lo demás son roles personalizados con nombre arbitrario.

### 6.3 Seguridad en la interpolación

Todo dato de sesión que se inserte en `innerHTML` debe escaparse:

```js
const esc = v => String(v || '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
```

### 6.4 Pantalla de Configuración (admin) — Usuarios / Roles / Conexión

Todo proyecto con RBAC tiene una pantalla **Configuración** visible solo para Administrador (`isAdmin()`). Su subtítulo canónico: *"Gestión de usuarios, permisos y configuración del sistema"*. Se arma con un **strip de 3 tabs** (segmented control: un contenedor con botones, el activo resaltado — cada proyecto usa sus propias clases/variables de tema):

| Tab | Contenido |
|-----|-----------|
| **Usuarios** | Tabla (Nombre · Email · Rol · Estado · Acciones) + form de alta/edición. El `<select>` de Rol se **puebla dinámicamente** desde `getRoles()` (solo activos); el badge de Rol en cada fila se resuelve por lookup contra esa misma lista. Sin hard-delete: "Desactivar". |
| **Roles y permisos** | Tabla de roles (Rol · Tipo `Sistema`/`Personalizado` · Estado · Acciones) + botón "Nuevo rol". Al elegir un rol se muestra su **matriz de permisos**: una fila por módulo con un selector de 3 estados (`Oculto` / `Solo ver` / `Ver + editar`). El Administrador aparece como solo-lectura (sin Editar/Desactivar/editar permisos). |
| **Conexión al Google Sheet** | Override opcional de la URL del Web App + estado de conexión + estructura de hojas requerida. Solo admin. |

**Reglas de implementación:**

- La pantalla la arma una única función (`renderUserManagementSection()`), idempotente (no re-renderiza si ya existe el contenedor). Las secciones de conexión que vivían sueltas en el HTML se **mueven** dentro del tab "Conexión" por DOM, no se duplican.
- **Orden de carga:** los usuarios dependen de los roles para mostrar el nombre del rol → cargar roles **antes** que usuarios (`_loadRoles().then(_loadUsuarios)`), no en paralelo, o la primera pintura muestra "Rol 1/2/3".
- Los 3 estados de permiso **no** son un campo nuevo: derivan de `puede_ver`+`puede_editar` (`Oculto`=NO/NO, `Solo ver`=SI/NO, `Ver+editar`=SI/SI). Al guardar se envían ambos booleanos por módulo.
- Ocultar botones es solo UX; el backend valida cada escritura por módulo (`apps_script_standards.md §7.2`). El enforcement real no vive en esta pantalla.

> Implementación de referencia: `commerce-hub/src/js/auth.js` (`renderUserManagementSection`, `_renderPermisosMatrix`, `_loadRoles`).

---

## 7. Sistema de temas

### 7.1 Convención

- Dos temas: `light` y `dark`.
- El atributo `data-theme` se pone en `<html>`.
- CSS usa `[data-theme="dark"] { … }` para las overrides.
- `localStorage` guarda la preferencia del usuario (ver §3 para las claves).
- Si no hay preferencia guardada, usar `prefers-color-scheme`.

### 7.2 Función `toggleTheme()`

```js
function toggleTheme() {
  const html = document.documentElement;
  const next = html.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
  html.setAttribute('data-theme', next);
  localStorage.setItem('<APP_PREFIX>_theme', next);
  _updateThemeToggles(next);
}

function _updateThemeToggles(theme) {
  document.querySelectorAll('.theme-toggle').forEach(btn => {
    btn.textContent = theme === 'dark' ? '☀ Modo claro' : '☾ Modo oscuro';
  });
}
```

### 7.3 Clase `.theme-toggle`

Todo botón de cambio de tema lleva clase `.theme-toggle`. Su texto cambia con `_updateThemeToggles()`. Puede haber más de uno en la misma página (topbar + sidebar) — ambos se sincronizan.

---

## 8. Version badge y popover de changelog

### 8.1 Ubicación del badge de versión

La versión se muestra **únicamente en el sidebar brand**, dentro de `.brand-meta` como primer `<span>`:

```html
<div class="brand-meta" id="sidebarVersionBtn" title="Ver historial de cambios">
  <span id="sidebarVersion">v…</span>
  <span>· Equipo / Área</span>
</div>
```

El badge no se duplica en topbar, footer de página ni ningún otro lugar. El texto inicial `v…` es un placeholder — se sobreescribe en `DOMContentLoaded` por `initVersionBadge()`.

### 8.2 Formato del número de versión

```
vX.Y.Z        → versión semántica, siempre con prefijo "v" minúscula
v1.2.3        ✓
1.2.3         ✗ (sin prefijo)
V1.2.3        ✗ (mayúscula)
v1.2          ✗ (sin parche)
```

El `CHANGELOG` usa la misma versión en el campo `v` pero **sin** prefijo:

```js
{ v: '1.2.3', date: '2026-06-24', desc: 'Descripción del cambio.' }
```

El prefijo `v` lo agrega `initVersionBadge()` al renderizar.

### 8.3 Formato de fecha

Las fechas se almacenan en formato **ISO 8601** (`YYYY-MM-DD`) en el array `CHANGELOG`. Se muestran tal cual sin transformación:

```js
{ v: '1.2.3', date: '2026-06-24', desc: '…' }   // correcto
{ v: '1.2.3', date: '24/06/2026', desc: '…' }   // incorrecto
{ v: '1.2.3', date: 'Jun 24',     desc: '…' }   // incorrecto
```

### 8.4 Estructura del `CHANGELOG`

Array de objetos definido en `config.js`, **ordenado del más reciente al más antiguo** (la entrada más nueva siempre en el índice 0). El popover renderiza las entradas en el mismo orden — la versión actual aparece primero.

```js
// En config.js — junto a VERSION
const VERSION = {
  number: '1.2.3',
  date:   '2026-06-24',
  notes:  'Descripción breve de la versión actual.',
};

const CHANGELOG = [
  { v: '1.2.3', date: '2026-06-24', desc: 'Descripción del cambio más reciente.' },
  { v: '1.2.2', date: '2026-06-23', desc: 'Descripción del cambio anterior.' },
  { v: '1.2.1', date: '2026-06-22', desc: 'Descripción de un cambio previo.' },
  // … entradas más antiguas al final
];
```

> `VERSION.number` siempre debe coincidir con el `v` de la primera entrada del `CHANGELOG`. Si se actualiza la versión, se agrega una nueva entrada al principio del array.

### 8.5 Cantidad de versiones visibles

No hay límite fijo de entradas. El popover tiene `max-height: 70vh` con `overflow-y: auto` — si el changelog crece, aparece scroll interno.

**Recomendación práctica:** mantener las últimas **10 versiones** en el `CHANGELOG`. Entradas más antiguas pueden archivarse en el historial de commits.

### 8.6 Comportamiento de apertura y cierre

| Acción | Resultado |
|--------|-----------|
| Click en `.brand-meta` (`#sidebarVersionBtn`) | Abre el popover |
| Click en `.brand-meta` cuando ya está abierto | Lo cierra (toggle) |
| Click en cualquier otro punto de la página | Cierra el popover |
| Click dentro del popover | No lo cierra (la propagación se detiene en el popover mismo) |
| Scroll en el popover | Hace scroll interno — no cierra el popover |
| Resize de ventana | El popover permanece abierto (posición `fixed`, no se recalcula) |

El popover **no** tiene botón de cierre explícito. El click fuera es suficiente.

### 8.7 Posición del popover

```css
.sidebar-version-popover {
  position: fixed;
  top: 58px;          /* debajo del área de brand del sidebar */
  left: 232px;        /* borde derecho del sidebar (224px) + 8px de separación */
}
```

`position: fixed` ancla el popover al viewport — no se mueve con el scroll del contenido principal. La combinación `top: 58px; left: 232px` lo posiciona justo a la derecha y debajo del sidebar brand.

> **No usar `position: absolute`** — queda pisado por el contenido de la página en proyectos con tablas o panels (`z-index` de contexto apilado).

### 8.8 Tamaño y scroll interno

```css
.sidebar-version-popover {
  min-width: 280px;
  max-width: 340px;
  max-height: 70vh;
  overflow-y: auto;
  padding: 14px 16px;
}
```

- `min-width: 280px` — evita que las líneas largas se corten.
- `max-width: 340px` — el popover no invade demasiado el contenido.
- `max-height: 70vh` — si el changelog tiene muchas entradas, el popover no supera el 70% de la ventana.
- `overflow-y: auto` — scroll interno automático cuando el contenido supera `max-height`.

### 8.9 Función canónica `initVersionBadge()`

Debe vivir en **`config.js`**, no en el HTML de la página. Se llama desde `DOMContentLoaded` de cada página que tenga sidebar (en Tipo C se llama en todos los módulos).

```js
function initVersionBadge() {
  var span    = document.getElementById('sidebarVersion');
  var btn     = document.getElementById('sidebarVersionBtn');
  var popover = document.getElementById('versionPopover');
  if (!span) return;
  span.textContent = 'v' + VERSION.number;
  if (!btn || !popover || !CHANGELOG.length) return;
  popover.innerHTML =
    '<div style="font-size:11px;font-weight:600;text-transform:uppercase;letter-spacing:.06em;'
    + 'color:var(--muted);padding-bottom:8px;margin-bottom:10px;'
    + 'border-bottom:1px solid var(--sidebar-line)">Historial de cambios</div>'
    + CHANGELOG.map(function(c) {
      return '<div style="margin-bottom:8px;">'
        + '<span style="font-weight:600;font-size:13px;">v' + c.v + '</span>'
        + '<span style="color:var(--muted);font-size:11px;margin-left:6px;">' + c.date + '</span>'
        + '<div style="font-size:12px;margin-top:2px;line-height:1.4;">' + c.desc + '</div>'
        + '</div>';
    }).join('');
  btn.style.cursor = 'pointer';
  btn.addEventListener('click', function(e) {
    e.stopPropagation();
    popover.style.display = popover.style.display === 'none' ? 'block' : 'none';
  });
  document.addEventListener('click', function() { popover.style.display = 'none'; });
}
```

> Proyectos que usen ES6 (arrow functions, template literals) en `config.js` pueden usar esa sintaxis — la lógica es idéntica. La implementación de referencia en Commerce Hub usa ES6; la de Project Control Center usa ES5.

### 8.10 Tipografía interna del popover

| Elemento | Estilo |
|----------|--------|
| Encabezado "Historial de cambios" | `font-size:11px`, `font-weight:600`, `text-transform:uppercase`, `letter-spacing:.06em`, `color:var(--muted)` |
| Separador bajo el encabezado | `border-bottom:1px solid var(--sidebar-line)`, `padding-bottom:8px`, `margin-bottom:10px` |
| Número de versión (`v1.2.3`) | `font-size:13px`, `font-weight:600` |
| Fecha | `font-size:11px`, `color:var(--muted)`, `margin-left:6px` |
| Descripción | `font-size:12px`, `line-height:1.4`, `margin-top:2px` |
| Separación entre entradas | `margin-bottom:8px` en el contenedor de cada entrada |

### 8.11 Responsive y mobile

En viewport ≤ 768 px el sidebar colapsa como drawer (ver §10.1). El popover **no se adapta automáticamente** porque usa `position: fixed` con coordenadas hardcodeadas:

- Si el sidebar está cerrado (drawer oculto), `#sidebarVersionBtn` no es visible → el popover no puede abrirse.
- Si el sidebar está abierto como drawer en mobile, el popover puede quedar fuera del viewport o solaparse con el overlay.

**Comportamiento esperado en mobile:** no mostrar el popover. Si el sidebar está abierto como drawer, el popover se puede abrir pero la experiencia es degradada (posición incorrecta).

**Workaround mínimo** (opcional): detectar mobile y deshabilitar el trigger:

```js
btn.addEventListener('click', function(e) {
  if (window.innerWidth <= 768) return;  // deshabilitar en mobile
  e.stopPropagation();
  popover.style.display = popover.style.display === 'none' ? 'block' : 'none';
});
```

> En la práctica estas herramientas son de uso interno en desktop. El workaround mobile es opcional y no es parte del estándar mínimo.

### 8.12 `<title>` del documento

```html
<!-- Index / Home -->
<title>Nombre de App</title>

<!-- Módulos -->
<title>Nombre de App | Nombre del Módulo</title>
```

---

## 9. Estados globales

### 9.1 Barra de progreso de carga API

Para módulos que realizan múltiples llamadas API al iniciar, usar una barra de progreso visual en el topbar. Evita mostrar spinners en cada sección individualmente.

```html
<div id="loadingBar" class="loading-bar" style="display:none;">
  <div class="loading-bar-fill" id="loadingBarFill"></div>
</div>
```

Actualizar con JS: `loadingBarFill.style.width = (step / total * 100) + '%'`.

### 9.2 Modo demo / QA

Cuando la aplicación corre en modo demo o apunta a un entorno de QA, mostrar un banner sticky debajo del topbar:

```html
<div class="env-banner demo-banner">
  ⚠ Modo demo activo. Los cambios no se guardan de forma permanente.
</div>
```

Clase `.demo-banner` para demo, `.qa-banner` para QA. Colores:
- Demo: `background: var(--warning-bg); color: var(--warning);`
- QA: `background: var(--info-bg); color: var(--info);`

### 9.3 Estado de conexión

Si el proyecto depende de una conexión activa al GAS backend, puede mostrar un indicador de estado. No es obligatorio pero sigue esta convención si se implementa:

```html
<span class="conn-status" data-status="ok">● Conectado</span>
```

Estados: `ok` (verde) · `error` (rojo) · `loading` (gris).

---

## 10. Comportamientos comunes

### 10.1 Sidebar mobile

En resoluciones ≤ 768 px, el sidebar actúa como drawer lateral:

- Un botón hamburguesa (`#sidebarToggle`) lo abre/cierra.
- Un overlay (`#sidebarOverlay`) cubre el contenido; click en él cierra el sidebar.
- Clases: `.sidebar.open` cuando está visible.
- Función `closeSidebarMobile()` para cerrar desde JS.

```html
<button class="sidebar-toggle" id="sidebarToggle" aria-label="Abrir menú" type="button">
  <!-- icono hamburguesa (SVG inline) -->
</button>
<div class="sidebar-overlay" id="sidebarOverlay"></div>
```

### 10.2 Restricción de escritura por módulo (RBAC)

Los elementos de escritura llevan clase `.admin-only`. `restrictWriteIfAgent()` los deshabilita si el usuario no puede editar **el módulo actual**. El módulo se deriva de la URL (`modules/<módulo>/`). El Administrador (`id_rol===1`) y el modo demo siempre pueden editar.

```js
// En auth.js — modelo RBAC flexible (no chequear role string)
function _currentModule() {
  const m = (location.pathname || '').match(/modules\/([^/]+)\//);
  return m ? m[1] : '';
}

function restrictWriteIfAgent() {
  const mod = _currentModule();
  const puedeEditar = isAdmin() || (mod && canEdit(mod));
  if (puedeEditar) { document.body.classList.remove('no-edit'); return; }
  document.body.classList.add('no-edit');
  document.querySelectorAll('.admin-only').forEach(function (el) {
    el.disabled = true;
    el.classList.add('agent-disabled');
    el.title = 'No tenés permisos de edición en este módulo';
  });
}
```

> **No usar** `role === 'agente'` ni `role !== 'admin'` — con el modelo RBAC flexible los roles son dinámicos. La fuente de verdad es `canEdit(mod)` (que lee `permisos` de la sesión) y `isAdmin()` (`id_rol===1`).

Llamar en `DOMContentLoaded` junto con `requireAuth()` y `renderUserChip()`.

### 10.3 Visibilidad de la sección Admin en el sidebar

El nav group de Admin se oculta por defecto y se muestra solo para el Administrador (`isAdmin()`):

```js
const adminGroup = document.getElementById('adminNavGroup');
if (adminGroup && isAdmin()) {
  adminGroup.style.display = '';
}
```

> Usar `isAdmin()` (definido en `auth.js` como `id_rol===1`), no comparar strings de rol.

### 10.4 Active state en navegación

**SPA:** Marcar el ítem activo con clase `.active` en `data-page`:

```js
document.querySelectorAll('.nav-item').forEach(el => {
  el.classList.toggle('active', el.dataset.page === page);
});
```

**Multi-página:** Marcar el ítem activo en el sidebar según la URL actual:

```js
const current = location.pathname.split('/').pop();
document.querySelectorAll('.sidebar-nav-item').forEach(a => {
  a.classList.toggle('active', a.getAttribute('href').endsWith(current));
});
```

### 10.5 Inicialización en DOMContentLoaded

Orden canónico de llamadas en cada módulo/página:

```js
document.addEventListener('DOMContentLoaded', () => {
  requireAuth();           // 1. Verificar sesión (redirige a login si no hay)
  renderUserChip();        // 2. Mostrar chip de usuario
  restrictWriteIfAgent();  // 3. Deshabilitar escritura para agentes
  // … inicialización específica del módulo
});
```

En el index (home), sumar:

```js
  renderSidebarUser();     // 4. Renderizar footer de usuario en sidebar
  initVersionBadge();      // 5. Mostrar versión y popover de changelog
  initSidebarNav();        // 6. Activar filtros y navegación del sidebar
```

---

## 11. Variantes por tipo de proyecto

### Tipo A — Multi-página (VTEX Control Center)

- `index.html` es el hub de módulos. Tiene sidebar completo.
- Cada módulo es un HTML independiente en `modules/<área>/`.
- Los módulos **no** tienen sidebar; tienen `<header class="topbar">` con eyebrow + H1 + subtítulo.
- El user chip se inserta en `#storeRow` dentro de cada módulo.
- La navegación entre módulos es navegación real del navegador (no JS).

```
index.html          → shell completo (sidebar + content-topbar)
modules/area/mod.html → topbar de módulo (sin sidebar)
```

### Tipo B — SPA (Commerce Hub, Project Control Center)

- `index.html` contiene todo. El sidebar está siempre presente.
- La navegación entre secciones es JS puro (`showPage()`).
- El topbar muestra el breadcrumb actualizado con la sección activa.
- El user chip puede estar en el sidebar footer o en el topbar.
- No hay botón "Volver al inicio".

```
index.html          → shell completo + todas las secciones (visibility toggle)
```

### Tipo C — Multi-página con sidebar por módulo (Project Control Center)

Variante intermedia: cada módulo tiene su propio HTML e **incluye el sidebar completo** (brand + nav + footer de usuario). No es un shell de index que apunta a módulos sin sidebar — es un shell replicado en cada archivo.

- El sidebar de cada módulo es idéntico al del index; comparten `config.js` y `auth.js`.
- El header de módulo va **dentro del `<main>`** (eyebrow + H1 + subtítulo), no en un `<header>` separado al estilo Tipo A.
- El sidebar footer con usuario, rol y theme toggle está presente en **todos** los módulos — no solo en el index.
- No hay botón "Volver al inicio" (el sidebar cubre la navegación).

```
index.html             → shell completo (sidebar + home)
modules/area/mod.html  → shell completo (sidebar + header de módulo en main)
```

> **Costo:** el sidebar se repite en cada HTML. Cambiar el nav implica editar todos los archivos. Preferir este tipo solo cuando la URL directa por módulo es un requisito real y el sidebar no se espera cambiar frecuentemente.

### Cuándo usar cada tipo

| Criterio | Tipo A (multi-página) | Tipo B (SPA) | Tipo C (multi-página + sidebar) |
|----------|-----------------------|--------------|---------------------------------|
| Cantidad de módulos | 5 o más | Menos de 6 | Cualquiera |
| URL directa por módulo | Sí | No | Sí |
| Sidebar persistente entre módulos | No | Sí | Sí (replicado) |
| Módulos con lógica compleja e independiente | Sí | No | Sí |
| Carga inicial rápida | No | Sí | No |
| Mantenimiento del sidebar | Bajo | Bajo | Alto (replicado) |

---

## 12. Plantilla HTML base

### 12.1 Index / Home (Tipo A y B)

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Nombre de App</title>
  <link rel="stylesheet" href="styles.css" />
  <!-- Anti-flash: PRIMER script, antes del CSS si el CSS usa data-theme -->
  <script>(function(){
    var t = localStorage.getItem('<APP_PREFIX>_theme') || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', t);
  })();</script>
</head>
<body>

  <!-- Mobile: hamburguesa + overlay -->
  <button class="sidebar-toggle" id="sidebarToggle" aria-label="Abrir menú" type="button">
    <!-- SVG hamburguesa -->
  </button>
  <div class="sidebar-overlay" id="sidebarOverlay"></div>

  <div class="app-layout">

    <!-- ═══ SIDEBAR ═══ -->
    <aside class="sidebar" id="sidebar">

      <div class="sidebar-brand">
        <div class="brand-icon">XX</div>
        <div>
          <div class="brand-name">Nombre de App</div>
          <div class="brand-meta" id="sidebarVersionBtn" title="Ver historial de cambios">
            <span id="sidebarVersion">v…</span>
            <span>· Equipo</span>
          </div>
        </div>
        <div class="sidebar-version-popover" id="versionPopover" style="display:none;"></div>
      </div>

      <!-- Store selector: omitir si el proyecto no es multi-tienda -->
      <div class="sidebar-store">
        <div class="sidebar-section-label">Tienda</div>
        <div class="store-selector" id="storeSelectorHome"></div>
        <div id="storeNotice" class="sidebar-store-notice" style="display:none;">
          ⚠ Seleccioná una tienda para comenzar.
        </div>
      </div>

      <nav class="sidebar-nav" id="sidebarNav">
        <!-- Nav groups o nav items según el tipo de proyecto -->
        <div id="adminNavGroup" style="display:none;">
          <!-- Admin nav items -->
        </div>
      </nav>

      <div class="sidebar-spacer"></div>
      <div class="sidebar-footer" id="sidebarFooter"></div>

    </aside>
    <!-- ═══ /SIDEBAR ═══ -->

    <!-- ═══ CONTENT ═══ -->
    <div class="content-area">

      <!-- Topbar (SPA: breadcrumb; multi-página home: search+metrics) -->
      <!-- Ver §5 para la estructura según variante -->

      <main class="container">
        <!-- Contenido -->
      </main>

    </div>
    <!-- ═══ /CONTENT ═══ -->

  </div><!-- .app-layout -->

  <script src="config.js"></script>
  <script src="auth.js"></script>
  <script src="app.js"></script>
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      requireAuth();
      renderSidebarUser();
      restrictWriteIfAgent();
      initVersionBadge();
      initSidebarNav();
      // … inicialización específica
    });
  </script>
</body>
</html>
```

### 12.2 Módulo (Tipo A — multi-página)

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Nombre de App | Nombre del Módulo</title>
  <link rel="stylesheet" href="../../styles.css" />
  <script>(function(){
    var t = localStorage.getItem('<APP_PREFIX>_theme') || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', t);
  })();</script>
</head>
<body>

  <header class="topbar">
    <div>
      <p class="eyebrow">Área</p>
      <h1>Nombre del Módulo</h1>
      <p class="subtitle">Descripción breve del propósito de este módulo.</p>
    </div>
    <a class="button secondary" href="../../index.html">← Volver al inicio</a>
  </header>

  <div id="storeRow" class="store-row"></div>

  <main class="container">
    <!-- Contenido del módulo -->
  </main>

  <script src="../../config.js"></script>
  <script src="../../auth.js"></script>
  <script src="../../app.js"></script>
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      requireAuth();
      renderUserChip();
      restrictWriteIfAgent();
      // … inicialización específica
    });
  </script>
</body>
</html>
```

---

## 13. Checklist de conformidad

Usar para auditar proyectos existentes o validar proyectos nuevos.

### Shell general

- [ ] Anti-flash script es el primer `<script>` del `<head>`
- [ ] Clave `localStorage` sigue la convención `<siglas>_theme`
- [ ] `<html lang="es">` presente
- [ ] `<title>` sigue el formato canónico
- [ ] Grilla `.app-layout` con `display:grid` y `--sidebar-width`

### Sidebar

- [ ] CSS de sidebar sigue los valores canónicos de §4.2 (padding, border-radius, font-size, gaps)
- [ ] `.sidebar-brand` tiene icon + nombre + versión + popover
- [ ] Versión inicializada con `initVersionBadge()` definida en `config.js`
- [ ] `CHANGELOG` en `config.js`, ordenado del más reciente al más antiguo (entrada [0] = versión actual)
- [ ] `VERSION.number` coincide con el `v` de la primera entrada del `CHANGELOG`
- [ ] Número de versión en formato `X.Y.Z` (sin prefijo `v` en el array, con prefijo en el badge)
- [ ] Fechas en formato `YYYY-MM-DD`
- [ ] Popover: `position:fixed; top:58px; left:232px; z-index:1000`
- [ ] Popover: `background:var(--card); border:1px solid var(--sidebar-line)`
- [ ] Popover: `min-width:280px; max-width:340px; max-height:70vh; overflow-y:auto`
- [ ] Popover muestra encabezado "Historial de cambios" con separador
- [ ] Tipografía interna del popover sigue §8.10
- [ ] Store selector solo si el proyecto es multi-tienda
- [ ] Grupo Admin oculto por defecto (`display:none`) y visible solo para admins
- [ ] `.sidebar-footer` renderizado por JS (no hardcodeado)

### Panel de usuario

- [ ] Nombre y rol del usuario escapados con `esc()`
- [ ] Badge de rol muestra el **nombre real** del rol (no `Admin`/`Agente` hardcodeado)
- [ ] Botón "Cambiar contraseña" llama `openChangePasswordModal()`
- [ ] Botón "Cerrar sesión" llama `authLogout()`
- [ ] Theme toggle llama `toggleTheme()` y tiene clase `.theme-toggle`

### Configuración (admin)

- [ ] Pantalla con 3 tabs: Usuarios · Roles y permisos · Conexión al Google Sheet
- [ ] `<select>` de rol y badges se pueblan dinámicamente desde `getRoles()`
- [ ] Roles cargan **antes** que usuarios (no en paralelo)
- [ ] Matriz de permisos con 3 estados (Oculto / Solo ver / Ver + editar) por módulo
- [ ] Administrador (`id_rol===1`) es solo-lectura: sin editar/desactivar/editar permisos

### Topbar

- [ ] CSS sigue §5.1: `height: 50px`, `padding: 0 22px`, `z-index: 40`
- [ ] Topbar contiene **solo** breadcrumb + `.topbar-actions` — nunca H1 ni subtítulo
- [ ] Breadcrumb: `.topbar-brand` (muted) + `.topbar-sep` (`›`) + `.topbar-crumb` (text)
- [ ] H1 y subtítulo están en `.page-header` **dentro de `<main>`** (§5.3)
- [ ] Excepción Tipo A (sin sidebar en módulos): §5.4 permite H1 en topbar

### Comportamientos

- [ ] `DOMContentLoaded` llama `requireAuth()` + `renderUserChip()` + `restrictWriteIfAgent()`
- [ ] `restrictWriteIfAgent()` usa `isAdmin()` + `canEdit(mod)` — **no** `role === 'agente'`
- [ ] `isAdmin()` se resuelve como `id_rol===1` — **no** `role === 'admin'`
- [ ] Grupo Admin en sidebar usa `isAdmin()` para mostrarse/ocultarse
- [ ] Botones de escritura tienen clase `.admin-only`
- [ ] Sidebar mobile funciona con hamburguesa + overlay
- [ ] Active state en nav se sincroniza al cambiar de sección/página

### Identidad visual

- [ ] `--primary` definido en `main.css` es único (no colisiona con otros proyectos del ecosistema)
- [ ] `.brand-icon` usa `--primary` como color de fondo
- [ ] Theme toggle **en el sidebar footer**, no como chip separado ni en el topbar
