# Style Guide Maestro — Productos Web Internos

**Versión:** 1.1.0  
**Fecha:** 2026-07-03  
**Alcance:** Marketplace Portal · Commerce Hub · VTEX Control Center · VTEX Bookmarklets · Project Control Center · Customer Service Control Center · Correos Transaccionales · Herramientas internas

---

## Índice

1. [Principios generales](#1-principios-generales)
2. [Sistema visual](#2-sistema-visual)
3. [Colores](#3-colores)
4. [Tipografía](#4-tipografía)
5. [Cards y widgets](#5-cards-y-widgets)
6. [Tablas](#6-tablas)
7. [Formularios](#7-formularios)
8. [Botones](#8-botones)
9. [Dashboards](#9-dashboards)
10. [JavaScript](#10-javascript)
11. [CSS](#11-css)
12. [Documentación](#12-documentación)
13. [Git](#13-git)
14. [Estructura de proyecto](#14-estructura-de-proyecto)
15. [Integración con IA](#15-integración-con-ia)
16. [Accesibilidad](#16-accesibilidad)

---

## 1. Principios generales

### 1.1 Filosofía de diseño

| Principio | Descripción |
|-----------|-------------|
| **Claridad ante todo** | La interfaz debe ser auto-explicativa. El usuario interno no tiene tiempo de leer manuales. Cada pantalla debe responder "¿qué puedo hacer aquí?" sin fricción. |
| **Densidad útil** | Las herramientas internas manejan datos. Se prefiere densidad informativa sobre espacios vacíos, sin sacrificar legibilidad. |
| **Acción trazable** | Toda acción destructiva o de escritura debe tener confirmación, log y posibilidad de reversión cuando sea técnicamente posible. |
| **Estado siempre visible** | El usuario debe saber en todo momento: qué tienda/cuenta está activa, qué rol tiene, qué acción está ejecutando, cuál fue el resultado. |
| **Consistencia por encima de creatividad** | Nuevas pantallas deben sentirse como parte del mismo sistema. No inventar componentes cuando ya existe uno que cumple la función. |

### 1.2 Principios de desarrollo

- **Vanilla first.** No añadir dependencias externas si el problema se puede resolver con HTML/CSS/JS puro.
- **Sin over-engineering.** Una función que se usa una sola vez no necesita ser genérica. Abstraer cuando hay 3+ usos reales.
- **Zero comentarios obvios.** Solo comentar el *por qué* cuando no es deducible del código. Nunca el *qué*.
- **Seguridad desde el diseño.** Las credenciales nunca tocan el frontend. Las acciones de escritura siempre validan permisos en el backend.
- **Bajo costo de mantenimiento.** Si un desarrollador nuevo no entiende el código en 5 minutos leyendo el archivo, el código es demasiado complejo.

---

## 2. Sistema visual

### 2.1 Espaciado

Sistema basado en múltiplos de 4px.

| Token | Valor | Uso típico |
|-------|-------|-----------|
| `--space-1` | 4px | Gap mínimo entre elementos inline |
| `--space-2` | 8px | Gap entre chips, badges, botones |
| `--space-3` | 12px | Padding interno de inputs y badges |
| `--space-4` | 16px | Gap entre cards en grid |
| `--space-5` | 20px | Padding interno de panels y cards |
| `--space-6` | 24px | Margin entre secciones menores |
| `--space-7` | 32px | Padding de containers |
| `--space-8` | 48px | Padding de topbar/sidebar |
| `--space-9` | 64px | Padding inferior de páginas |

### 2.2 Border radius

```css
--radius-xs:  4px;   /* Tags, chips inline */
--radius-sm:  8px;   /* Botones, inputs, badges */
--radius:    14px;   /* Cards, panels, modales */
--radius-lg: 20px;   /* Modales grandes */
--radius-pill: 999px; /* Badges de estado, store-btns */
```

### 2.3 Sombras

```css
--shadow-sm: 0 1px 3px rgba(0,0,0,.07), 0 1px 2px rgba(0,0,0,.05);
--shadow:    0 4px 16px rgba(17,24,39,.08), 0 1px 4px rgba(17,24,39,.04);
--shadow-lg: 0 12px 36px rgba(17,24,39,.10);
```

Reglas de uso:
- `shadow-sm` → cards en reposo, panels, topbar
- `shadow` → modales, dropdowns, popovers
- `shadow-lg` → modales grandes, cards en hover

### 2.4 Layout

**Layout con sidebar** (dashboards y herramientas con múltiples secciones):

```
┌──────────────────┬─────────────────────────────────────────┐
│ SIDEBAR (224px)  │ CONTENT AREA                            │
│ fijo, 100vh      │  ├─ Topbar sticky (search + métricas)   │
│                  │  └─ Main container (scrolleable)        │
└──────────────────┴─────────────────────────────────────────┘

> Ancho canónico del sidebar: **224px** (`--sidebar-w`). Ver `application_shell.md §4.2`.
```

**Layout hub-and-spoke** (módulos standalone):

```
TOPBAR (gradiente, 100%)
└── MAIN CONTAINER (padding 32px 48px)
    ├── Store row (selector de cuenta)
    ├── Panel de búsqueda/filtros
    ├── KPI grid
    └── Panel de tabla
```

### 2.5 Grid

```css
/* Grid de cards en home */
.home-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 14px;
}

/* Grid de KPIs */
.kpi-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(138px, 1fr));
  gap: 10px;
}
```

### 2.6 Responsive

| Breakpoint | Ancho | Cambios principales |
|------------|-------|---------------------|
| Desktop | > 900px | Layout completo, sidebar visible |
| Tablet | 600–900px | Sidebar colapsado (hamburger), grid 2 cols |
| Mobile | < 600px | Una columna, topbar compacto, sin sidebar |

```css
@media (max-width: 900px) { /* Sidebar colapsado */ }
@media (max-width: 768px) { /* Ajustes de padding y grid */ }
@media (max-width: 600px) { /* Layout mobile */ }
```

---

## 3. Colores

### 3.1 Paleta base

```css
:root {
  /* Fondo */
  --bg:   #f0f2f7;
  --card: #ffffff;

  /* Texto */
  --text:  #111827;
  --muted: #6b7280;

  /* Líneas */
  --line: #e5e7eb;

  /* Primary — azul institucional */
  --primary:       #1a3f6b;
  --primary-hover: #14325a;
  --primary-soft:  #e8f0fb;
  --primary-mid:   #bfd4f5;

  /* Success */
  --success:    #0a7040;
  --success-bg: #ecfdf5;

  /* Warning */
  --warning:    #92400e;
  --warning-bg: #fffbeb;

  /* Danger */
  --danger:       #991b1b;
  --danger-bg:    #fef2f2;
  --danger-hover: #7f1d1d;

  /* Info (alias de primary-soft) */
  --info:    var(--primary-soft);
  --info-mid: var(--primary-mid);
}
```

### 3.2 Tokens semánticos

| Token | Color | Uso |
|-------|-------|-----|
| `--primary` | `#1a3f6b` | Botones principales, links, headers, sidebar |
| `--success` | `#0a7040` | Estado disponible, guardado OK, importación OK |
| `--warning` | `#92400e` | Pendiente, alerta no bloqueante, riesgo medio |
| `--danger` | `#991b1b` | Error, eliminación, acción destructiva, riesgo alto |
| `--muted` | `#6b7280` | Texto secundario, labels, placeholders |
| `--line` | `#e5e7eb` | Bordes, separadores, fondos de tabla |

### 3.3 Colores del sidebar

El sidebar usa fondo slate claro, uniforme en todos los proyectos:

```css
--sidebar-bg:   #f1f5f9;   /* slate-100 */
--sidebar-line: #e2e8f0;   /* slate-200 — bordes y separadores del sidebar */
```

```css
.sidebar {
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-line);
}
```

Textos sobre sidebar:
- Principal: `var(--text)`
- Secundario / labels: `var(--muted)`
- Item activo: `color: var(--primary); background: var(--primary-soft)`
- Item hover: `color: var(--text); background: color-mix(in srgb, var(--text) 4%, transparent)`

**Dark mode:** el sidebar hereda `--sidebar-bg: var(--s1)` (fondo oscuro del proyecto), manteniendo coherencia sin gradiente.

### 3.4 Topbar (módulos standalone)

```css
background: linear-gradient(135deg, #0e2d4f 0%, #1a3f6b 55%, #1e4d82 100%);
```

Con overlay decorativo:
```css
::after {
  background: radial-gradient(ellipse 70% 120% at 80% -20%,
    rgba(100,160,255,.12) 0%, transparent 60%);
}
```

### 3.5 Reglas de uso de color

- **No usar colores fuera de la paleta** sin agregarlos como variable CSS primero.
- **El rojo (`--danger`) solo para acciones destructivas irreversibles.** No para warnings informativos.
- **Los badges de estado** siguen siempre el esquema: fondo suave + texto oscuro del mismo tono.
- **Nunca usar opacidad en colores de texto** salvo en overlays o elementos sobre fondo oscuro.

---

### 3.6 Modo oscuro (dark/light toggle)

Los proyectos pueden ofrecer modo oscuro. El mecanismo estándar usa el atributo `data-theme` en `<html>`.

**Estructura base:**

```css
/* Modo claro — valores por defecto (en :root, igual que hoy) */
:root {
  --bg:   #f0f2f7;
  --card: #ffffff;
  /* ... resto de tokens */
}

/* Modo oscuro — sobreescribe solo lo que cambia */
[data-theme="dark"] {
  --bg:   #0d1117;
  --card: #161b22;
  --card2: #1c2128;   /* superficie secundaria */

  --text:  #e6edf3;
  --muted: #848d97;

  --line: #30363d;

  /* Primary — azul institucional legible sobre fondo oscuro */
  --primary:       #58a6ff;
  --primary-hover: #79b8ff;
  --primary-soft:  rgba(88,166,255,.12);
  --primary-mid:   rgba(88,166,255,.25);

  /* Semánticos adaptados para fondo oscuro */
  --success:    #3fb950;
  --success-bg: rgba(63,185,80,.12);

  --warning:    #d29922;
  --warning-bg: rgba(210,153,34,.12);

  --danger:       #f85149;
  --danger-bg:    rgba(248,81,73,.12);
  --danger-hover: #ff7b72;

  --info:     var(--primary-soft);
  --info-mid: var(--primary-mid);
}
```

**Toggle JS — patrón estándar:**

```js
// Al cargar la página — respetar preferencia guardada o del sistema
const saved = localStorage.getItem('theme')
  || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
document.documentElement.dataset.theme = saved;

// Toggle manual
function toggleTheme() {
  const next = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
  document.documentElement.dataset.theme = next;
  localStorage.setItem('theme', next);
}
```

**Botón de toggle (topbar):**

```html
<button class="btn-icon" onclick="toggleTheme()" aria-label="Cambiar tema" title="Cambiar tema">
  ☀️<!-- o ícono SVG de sol/luna -->
</button>
```

**Sidebar y topbar en modo oscuro:**

El sidebar slate en dark mode usa `--sidebar-bg: var(--s1)` del proyecto (fondo oscuro nativo), manteniendo coherencia sin gradiente.  
El topbar standalone sobre fondo oscuro usa el mismo gradiente institucional, sin cambios.

**Reglas:**

- El modo por defecto de **todos** los proyectos es **claro** (`data-theme` ausente = light).
- **Todos los colores hardcodeados en CSS rompen el dark mode** — siempre usar variables CSS.
- Los colores semánticos en dark son **versiones más claras/saturadas** de los de light (necesitan contraste sobre fondo oscuro).
- Commerce-hub es la implementación de referencia para dark/light mode completo.

---

## 4. Tipografía

### 4.1 Fuentes

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?
  family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,700;1,9..40,400
  &family=DM+Mono:wght@400;500
  &display=swap" />
```

```css
--font: 'DM Sans', system-ui, sans-serif;
--mono: 'DM Mono', 'Fira Mono', monospace;
```

| Variable | Fuente | Uso |
|----------|--------|-----|
| `--font` | DM Sans | Todo el UI: labels, párrafos, botones, headings |
| `--mono` | DM Mono | Valores numéricos de KPIs, datos de tabla, IDs, códigos |

### 4.2 Jerarquía tipográfica

```css
h1 { font-size: 30px; font-weight: 700; letter-spacing: -.02em; }
h2 { font-size: 17px; font-weight: 600; letter-spacing: -.01em; }
h3 { font-size: 15px; font-weight: 600; }

/* Eyebrow — etiqueta de sección */
.eyebrow {
  font-size: 11px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: .14em;
  opacity: .6;
  margin-bottom: 6px;
}

/* Body */
body     { font-size: 15px; line-height: 1.6; }
p        { font-size: 14px; color: var(--muted); line-height: 1.5; }
small    { font-size: 12px; font-weight: 600; }
```

### 4.3 Tamaños de referencia rápida

| Uso | Tamaño | Peso |
|-----|--------|------|
| Título de página (h1) | 30px | 700 |
| Título de sección (h2) | 17–22px | 600 |
| Título de card (h3) | 15px | 600 |
| Eyebrow / label uppercase | 11px | 500–700 |
| Cuerpo principal | 15px | 400 |
| Texto de card | 14px | 400 |
| Labels de tabla (th) | 11.5px | 600 |
| Datos de tabla (td) | 12–12.5px | 400 |
| KPI valor | 26–32px | 700 |
| KPI label | 12px | 700 |
| Botones | 13.5px | 600 |

### 4.4 Reglas de tipografía

- **`--mono` obligatorio** en: valores KPI, datos numéricos de tablas, IDs, códigos de producto, timestamps.
- **Nunca centrar texto** en tablas ni en párrafos de UI (excepto modales de confirmación).
- **Eyebrow siempre en mayúsculas** con `letter-spacing` amplio. Nunca reemplazarlo con h4 o h5.
- **Subtítulos de módulo** usan clase `.subtitle` con color `rgba(255,255,255,.72)` sobre fondo oscuro.

---

## 5. Cards y widgets

### 5.1 Card base

```css
.card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  padding: 22px 24px;
}
```

### 5.2 Module card (home)

Estructura HTML estándar:

```html
<a class="module-card active" href="ruta/al/modulo.html"
   data-module
   data-section="catalogo"
   data-status="available"
   data-keywords="palabras clave para búsqueda">
  <span class="module-tag">Área</span>
  <h3>Nombre del módulo</h3>
  <p>Descripción breve de qué hace y para qué sirve.</p>
  <small>Disponible</small>
</a>
```

Estados de module card:

| Clase | Visual | Interacción |
|-------|--------|-------------|
| `.active` | Normal, cursor pointer | Clickeable, hover eleva la card |
| `.pending` | Border left warning, opacidad .8 | No clickeable, muestra razón |
| `.disabled` | Opacidad .5 | No clickeable |

```css
.module-card.active:hover {
  transform: translateY(-3px);
  box-shadow: var(--shadow-lg);
  border-color: var(--primary-mid);
}
```

### 5.3 KPI card

```html
<div class="kpi-card">
  <div class="kpi-label">Disponibles</div>
  <div class="kpi-value">14</div>
</div>
```

```css
.kpi-card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: var(--radius-sm);
  padding: 12px 14px;
}
.kpi-value {
  font-size: 26px;
  font-weight: 700;
  font-family: var(--mono);
  color: var(--primary);
}
.kpi-label {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: .08em;
  color: var(--muted);
  margin-bottom: 4px;
}
```

### 5.4 Hero card / panel informativo

```html
<div class="hero-card">
  <span class="module-tag">Contexto</span>
  <h2>Título descriptivo</h2>
  <p>Descripción del propósito de esta sección.</p>
</div>
```

### 5.5 Module tag (badge de área)

```css
.module-tag {
  width: fit-content;
  background: var(--primary-soft);
  color: var(--primary);
  border-radius: 999px;
  font-size: 11px;
  font-weight: 700;
  padding: 4px 10px;
  letter-spacing: .06em;
  text-transform: uppercase;
}
```

### 5.X Chip de usuario y dropdown

El área de usuario del sidebar se implementa como un **chip compacto** que abre un dropdown. No usar botones a ancho completo apilados (patrón obsoleto).

**Estructura visual (dos líneas, sin avatar):**
```
Gabriel Luna ▾
Administrador
```

**HTML del chip (generado por JS — ver `application_shell.md §4.7`):**
```html
<div id="sidebar-user-chip" class="user-chip" role="button"
     aria-haspopup="true" aria-expanded="false"
     title="Gabriel Luna">
  <div class="user-chip-info">
    <span class="user-chip-name">
      Gabriel Luna <span class="user-chip-chevron" aria-hidden="true">▾</span>
    </span>
    <span class="user-chip-role">Administrador</span>
  </div>
</div>
```

**HTML del dropdown (generado por JS, adjunto a `<body>`):**
```html
<div id="user-dropdown" class="user-dropdown" role="menu" style="display:none">
  <div class="user-dropdown-header">
    <div class="sidebar-user-name">Gabriel Luna</div>
    <div class="sidebar-user-email">gabriel@empresa.com</div>
    <div class="sidebar-user-meta">
      <span class="auth-chip-role">Administrador</span>
    </div>
  </div>
  <div class="user-dropdown-sep"></div>
  <button class="user-dropdown-item theme-toggle" type="button" onclick="toggleTheme()">
    <span class="th-icon">☾</span>
    <span class="th-label">Modo oscuro</span>
  </button>
  <div class="user-dropdown-sep"></div>
  <button class="user-dropdown-item" type="button" onclick="openChangePasswordModal()">Cambiar contraseña</button>
  <div class="user-dropdown-sep"></div>
  <button class="user-dropdown-item danger" type="button" onclick="authLogout()">Cerrar sesión</button>
</div>
```

**CSS canónico:**
```css
/* Chip */
.user-chip { display:flex; align-items:center; padding:8px 10px; border-radius:var(--radius-sm); cursor:pointer; border:1px solid transparent; transition:all .15s; user-select:none; width:100%; }
.user-chip:hover { background:color-mix(in srgb,var(--text) 5%,transparent); border-color:var(--line); }
.user-chip.open { background:var(--primary-soft); border-color:color-mix(in srgb,var(--primary) 20%,transparent); }

.user-chip-info { display:flex; flex-direction:column; gap:1px; min-width:0; flex:1; }
.user-chip-name { font-size:13px; font-weight:600; color:var(--text); overflow:hidden; text-overflow:ellipsis; white-space:nowrap; }
.user-chip-role { font-size:11px; color:var(--muted); font-weight:400; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.user-chip-chevron { font-size:9px; opacity:.5; margin-left:3px; }
.user-chip.open .user-chip-name { color:var(--primary); }

/* Dropdown */
.user-dropdown { position:fixed; background:var(--card); border:1px solid var(--line); border-radius:8px; box-shadow:0 8px 24px rgba(0,0,0,.12); z-index:200; overflow:hidden; min-width:200px; }
.user-dropdown-header { padding:12px 14px; }
.user-dropdown-sep { height:1px; background:var(--line); }
.user-dropdown-item { display:flex; align-items:center; gap:8px; width:100%; padding:9px 14px; font-size:13px; font-weight:500; color:var(--text); background:none; border:none; cursor:pointer; text-align:left; font-family:inherit; transition:background .12s; }
.user-dropdown-item:hover { background:color-mix(in srgb,var(--text) 5%,transparent); }
.user-dropdown-item.danger { color:var(--danger); }
.user-dropdown-item.danger:hover { background:var(--danger-bg); }
```

**Reglas:**
- El dropdown se adjunta al `<body>` para evitar clipping del `overflow-y` del sidebar.
- Se posiciona `fixed` calculando `getBoundingClientRect()` del chip al abrir.
- El ítem `.theme-toggle` en el dropdown usa las mismas clases que el patrón estándar — `setTheme()` lo actualiza automáticamente junto con cualquier otro `.theme-toggle` de la página.
- Cerrar: clic fuera del dropdown (listener en `document`) o tecla Escape.
- El dropdown **no** se duplica en topbar ni como panel separado.

---

## 6. Tablas

### 6.1 Estructura HTML canónica

```html
<div class="panel">
  <!-- Toolbar: búsqueda + acciones -->
  <div class="table-toolbar">
    <input type="search" id="tableSearch" placeholder="Buscar..." />
    <span id="rowCount" class="row-count"></span>
    <button id="btnExport" class="secondary">Exportar CSV</button>
  </div>

  <!-- Barra de progreso (visible durante carga API, oculta en finally) -->
  <div id="progressWrap" style="display:none;">
    <div class="progress-header">
      <span class="progress-label">Cargando…</span>
    </div>
    <div class="progress-track">
      <div id="progressBar" class="progress-bar"></div>
    </div>
    <div class="progress-footer">
      <span class="progress-detail">Aguardá un momento.</span>
    </div>
  </div>

  <!-- Status bar -->
  <div id="statusBar" class="status-bar"></div>

  <!-- Paginación -->
  <div class="pagination" id="pagination" style="display:none">
    <span class="pagination-info" id="paginationInfo"></span>
    <select id="pageSizeSel" aria-label="Filas por página">
      <option value="25" selected>25 por página</option>
      <option value="50">50 por página</option>
      <option value="100">100 por página</option>
    </select>
    <button id="btnPrev" class="secondary btn-sm">← Anterior</button>
    <button id="btnNext" class="secondary btn-sm">Siguiente →</button>
  </div>

  <!-- Tabla con scroll -->
  <div class="table-wrap">
    <table class="data-table">
      <thead>
        <tr>
          <th data-sortable data-key="name">Nombre</th>
          <th data-sortable data-key="status">Estado</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody id="tableBody"></tbody>
    </table>
  </div>
</div>
```

### 6.2 CSS de tabla

El CSS de tabla y los patrones de sorting/paginación están centralizados en `styles.css`. No agregar overrides locales — solo excepciones puntuales justificadas (ej. scroll height fijo).

```css
/* En styles.css — no copiar en módulos */
.table-wrap { overflow: auto; border: 1px solid var(--line); border-radius: var(--radius); background: var(--card); }
table { width: 100%; border-collapse: collapse; }
th { background: var(--bg); font-weight: 600; position: sticky; top: 0; z-index: 1;
     white-space: nowrap; font-size: 11.5px; text-transform: uppercase; letter-spacing: .05em; padding: 10px 12px; }
td { padding: 10px 12px; font-size: 12.5px; border-bottom: 1px solid var(--line); vertical-align: middle; }
tbody tr:hover { background: color-mix(in srgb, var(--primary) 4%, var(--card)); }
tbody tr:last-child td { border-bottom: 0; }
.data-table td { white-space: nowrap; font-family: var(--mono); font-size: 12px; }

/* Sorting */
th[data-sortable] { cursor: pointer; user-select: none; }
th[data-sortable]:hover { background: var(--primary-soft); }
th[data-sortable]::after { content: ' ↕'; font-size: 10px; opacity: .35; }
th[data-sortable].sort-asc::after  { content: ' ↑'; opacity: 1; color: var(--primary); }
th[data-sortable].sort-desc::after { content: ' ↓'; opacity: 1; color: var(--primary); }

/* Paginación */
.pagination { display: flex; align-items: center; gap: 10px; margin-top: 14px; flex-wrap: wrap; }
.pagination-info { font-size: 12.5px; color: var(--muted); flex: 1; }
```

`.data-table` en `styles.css` asume, además, que la columna de **nombre** cae en la 4ta posición y una de **descripción** en la 5ta, con `min-width: 1400px` — pensado para tablas grandes tipo listados de categorías/marcas:

```css
/* En styles.css */
.data-table { min-width: 1400px; }
.data-table td:nth-child(4) { font-family: var(--font); font-weight: 500; white-space: normal; min-width: 140px; }
.data-table td:nth-child(5) { color: var(--muted); max-width: 160px; overflow: hidden; text-overflow: ellipsis; }
```

#### 6.2.1 Override local cuando el layout de columnas no coincide

Una tabla más chica o con otro orden de columnas (la de nombre en la 2da posición, sin columna de descripción, 5-6 columnas en vez de 10+) no encaja con esas dos reglas posicionales. **No** crear una clase local nueva tipo `.mi-modulo-table` que reimplemente `.data-table` desde cero — eso es exactamente el tipo de duplicación que este documento busca evitar (visto repetido en VTEX Control Center como `.offers-table`, `.col-table`, cada uno casi idéntico a `.data-table` con números ligeramente distintos). En su lugar, seguir usando `class="data-table"` y neutralizar/reapuntar las dos reglas posicionales con overrides **scoped por ID de sección**, marcando la columna de texto libre con una clase explícita (`.col-name`) en vez de depender de su posición:

```css
/* Local al módulo — el layout de esta tabla no coincide con la asunción de .data-table */
.data-table { min-width: 780px; }                    /* ancho acorde a esta tabla, no 1400px */
#resultsPanel td:nth-child(4):not(.col-name),
#resultsPanel td:nth-child(5):not(.col-name) {
  font-family: var(--mono); font-weight: 400; white-space: nowrap;
  color: inherit; max-width: none; overflow: visible; text-overflow: clip;
}
#resultsPanel td.col-name {
  font-family: var(--font); font-weight: 500; white-space: normal; min-width: 220px; max-width: 380px;
}
```

```html
<td class="col-name">${producto.nombre}</td>  <!-- en cualquier posición -->
```

El selector `#resultsPanel td:nth-child(4)` (ID) le gana en especificidad a `.data-table td:nth-child(4)` (clase) del CSS global sin importar el orden de carga, así que el reset es determinístico. La condición `:not(.col-name)` evita pisar la celda que sí es la de nombre si por coincidencia cae en la posición 4 o 5. Este es el **único** tipo de override local aceptable sobre `.data-table` — ancho mínimo + neutralizar las 2 reglas posicionales cuando no aplican al layout real, nunca una clase paralela completa.

### 6.3 Comportamiento requerido

| Feature | Regla |
|---------|-------|
| **Sticky header** | `position: sticky; top: 0; z-index: 1` en `th` — ya en `styles.css` |
| **Búsqueda** | `input` event → filtro en memoria → resetea `currentPage = 1` |
| **Sorting** | `th[data-sortable][data-key]` + clases `sort-asc`/`sort-desc`; resetea `currentPage = 1` al cambiar columna |
| **Paginación** | Selector 25/50/100 + Anterior/Siguiente + info "X–Y de Z"; **25 por defecto** |
| **Export CSV** | Usa `_pageRows` (vista filtrada completa), **nunca** `currentRows` (solo la página visible) |
| **Empty state** | `<td colspan="N" class="text-muted">Sin resultados.</td>` dentro del `tbody` — nunca un `<p>` externo |
| **Loading** | `progressWrap` visible al iniciar la llamada API; se oculta en `finally` |

### 6.4 Sorting — patrón JS canónico

```javascript
let sortCol = null;
let sortDir = 'asc';

function sortData(rows) {
  if (!sortCol) return rows;
  return [...rows].sort((a, b) => {
    const va = String(a[sortCol] ?? '').toLowerCase();
    const vb = String(b[sortCol] ?? '').toLowerCase();
    return sortDir === 'asc' ? va.localeCompare(vb) : vb.localeCompare(va);
  });
}

function updateSortHeaders() {
  document.querySelectorAll('th[data-sortable]').forEach(th => {
    th.classList.remove('sort-asc', 'sort-desc');
    if (th.dataset.key === sortCol) th.classList.add('sort-' + sortDir);
  });
}

// Wiring (en DOMContentLoaded o tras renderizar el thead dinámico)
document.querySelectorAll('th[data-sortable]').forEach(th => {
  th.addEventListener('click', () => {
    currentPage = 1;
    sortDir = sortCol === th.dataset.key && sortDir === 'asc' ? 'desc' : 'asc';
    sortCol = th.dataset.key;
    updateSortHeaders();
    renderTable(sortData(applyFilters(allRows)));
  });
});
```

### 6.5 Paginación — patrón JS canónico

```javascript
let currentPage = 1;
let pageSize    = 25;
let _pageRows   = []; // vista completa filtrada+ordenada (para export y navegación)

function renderTable(rows) {
  _pageRows = rows;
  const total     = rows.length;
  const pageCount = Math.max(1, Math.ceil(total / pageSize));
  if (currentPage > pageCount) currentPage = pageCount;
  const start = (currentPage - 1) * pageSize;
  const slice = rows.slice(start, start + pageSize);

  tbody.innerHTML = slice.map(rowHtml).join('');

  const pag = document.getElementById('pagination');
  pag.style.display = total > 0 ? 'flex' : 'none';
  document.getElementById('paginationInfo').textContent =
    total <= pageSize ? `${total} filas` : `${start + 1}–${Math.min(start + pageSize, total)} de ${total}`;
  document.getElementById('btnPrev').disabled = currentPage === 1;
  document.getElementById('btnNext').disabled = currentPage >= pageCount;
}

document.getElementById('pageSizeSel').addEventListener('change', e => {
  pageSize = Number(e.target.value); currentPage = 1; renderTable(_pageRows);
});
document.getElementById('btnPrev').addEventListener('click', () => {
  if (currentPage > 1) { currentPage--; renderTable(_pageRows); }
});
document.getElementById('btnNext').addEventListener('click', () => {
  const pc = Math.ceil(_pageRows.length / pageSize);
  if (currentPage < pc) { currentPage++; renderTable(_pageRows); }
});

// Búsqueda y filtros siempre resetean la página
tableSearch.addEventListener('input', () => { currentPage = 1; renderTable(sortData(applyFilters(allRows))); });
```

**Invariante crítica**: el botón Exportar siempre usa `_pageRows`, nunca el tbody ni `currentRows`.

### 6.6 Loading state — patrón JS canónico

```javascript
const progressWrap = document.getElementById('progressWrap');
const progressBar  = document.getElementById('progressBar');

async function loadData() {
  setLoading(btnLoad, true);
  progressWrap.style.display = 'block';
  progressBar.style.width    = '60%';
  try {
    const res = await callApi('myAction', { sessionToken: getSessionToken() });
    renderTable(sortData(res.data || []));
  } catch (err) {
    setStatus(statusBar, err.message, 'error');
  } finally {
    setLoading(btnLoad, false);
    progressWrap.style.display = 'none';
    progressBar.style.width    = '0%';
  }
}
```

### 6.7 Normalización para búsqueda

```javascript
function normalizeText(value) {
  return (value || '').toString()
    .normalize('NFD')
    .replace(/[̀-ͯ]/g, '')
    .toLowerCase();
}
```

Aplicar siempre antes de comparar texto del usuario con datos de la tabla.

### 6.8 Exportación CSV

```javascript
function csvEscape(value) {
  const str = String(value ?? '').replace(/"/g, '""');
  return /[,;\n"]/.test(str) ? `"${str}"` : str;
}

function downloadCSV(filename, rows) {
  const content = rows.map(r => r.map(csvEscape).join(',')).join('\n');
  const blob = new Blob(['﻿' + content], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const a = Object.assign(document.createElement('a'), { href: url, download: filename });
  a.click();
  URL.revokeObjectURL(url);
}
```

El BOM `﻿` es obligatorio para compatibilidad con Excel en Windows.

---

## 7. Formularios

### 7.1 Estructura de input

```html
<div class="field">
  <label for="inputId">Label del campo</label>
  <input id="inputId" type="text" placeholder="Ej: valor esperado" />
  <span class="field-error" id="inputId-err" style="display:none;">Mensaje de error</span>
</div>
```

```css
.field {
  display: grid;
  gap: 5px;
}

.field label {
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
}

input[type="text"],
input[type="email"],
input[type="password"],
input[type="search"],
select,
textarea {
  border: 1px solid var(--line);
  border-radius: var(--radius-sm);
  padding: 9px 14px;
  background: var(--card);
  color: var(--text);
  font-family: var(--font);
  font-size: 13.5px;
  outline: none;
  transition: border-color .15s, box-shadow .15s;
  width: 100%;
}

input:focus, select:focus, textarea:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 15%, transparent);
}

input.error, select.error, textarea.error {
  border-color: var(--danger);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--danger) 12%, transparent);
}

.field-error {
  font-size: 12px;
  color: var(--danger);
  font-weight: 500;
}
```

### 7.2 Validación

Reglas:
- Validar **al intentar guardar**, no en tiempo real (salvo formatos complejos como email).
- **Mensajes de error inline** debajo del campo, nunca solo en toast o status bar.
- Marcar el campo con clase `.error` y hacer scroll al primer campo inválido.
- Deshabilitar el botón de guardar mientras hay un request en curso.

```javascript
function validateForm(fields) {
  let valid = true;
  fields.forEach(({ id, required, maxLength, label }) => {
    const el = document.getElementById(id);
    const errEl = document.getElementById(`${id}-err`);
    const value = el.value.trim();
    let msg = '';

    if (required && !value) msg = `${label} es obligatorio.`;
    else if (maxLength && value.length > maxLength)
      msg = `${label} no puede superar ${maxLength} caracteres.`;

    el.classList.toggle('error', !!msg);
    if (errEl) { errEl.textContent = msg; errEl.style.display = msg ? '' : 'none'; }
    if (msg) valid = false;
  });
  return valid;
}
```

### 7.3 Estados del botón de guardar

```javascript
function setButtonLoading(btn, loading) {
  btn.disabled = loading;
  btn.classList.toggle('loading', loading);
  if (!loading) btn.textContent = btn.dataset.originalText || 'Guardar';
}
```

### 7.4 Modales de formulario

```html
<div class="modal-overlay" id="editModal" style="display:none;">
  <div class="modal" role="dialog" aria-modal="true" aria-labelledby="editModalTitle">
    <h3 id="editModalTitle">Editar elemento</h3>
    <div id="editModalBody">
      <!-- campos del formulario -->
    </div>
    <div id="editModalStatus" class="status-bar" style="display:none;"></div>
    <div class="modal-actions">
      <button class="secondary" onclick="closeModal('editModal')">Cancelar</button>
      <button id="editModalSave" onclick="saveEdit()">Guardar</button>
    </div>
  </div>
</div>
```

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  backdrop-filter: blur(2px);
}

.modal {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow-lg);
  padding: 28px 28px 24px;
  width: 100%;
  max-width: 480px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}
```

---

## 8. Botones

### 8.1 Estilos base

```css
button, .button {
  border: 0;
  border-radius: var(--radius-sm);
  padding: 9px 16px;
  font-family: var(--font);
  font-size: 13.5px;
  font-weight: 600;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  white-space: nowrap;
  transition: background .15s, opacity .15s, transform .1s;
}

button:hover:not(:disabled) { transform: translateY(-1px); }
button:focus-visible { outline: 3px solid var(--primary-mid); outline-offset: 2px; }
button:disabled { opacity: .42; cursor: not-allowed; transform: none !important; }
```

### 8.2 Variantes

```css
/* Primario — acción principal */
button            { background: var(--primary); color: white; }
button:hover:not(:disabled) { background: var(--primary-hover); }

/* Secundario — acción alternativa o cancel */
button.secondary  { background: var(--bg); color: var(--text); border: 1px solid var(--line); }
button.secondary:hover:not(:disabled) { background: color-mix(in srgb, var(--text) 6%, var(--bg)); }

/* Danger — eliminación o acción destructiva */
button.danger     { background: var(--danger); color: white; }
button.danger:hover:not(:disabled) { background: var(--danger-hover); }

/* Ghost — acción terciaria o en sidebar */
button.ghost      { background: transparent; color: var(--primary); border: 1px solid var(--primary-mid); }
button.ghost:hover:not(:disabled) { background: var(--primary-soft); }

/* Loading — durante request */
button.loading    { pointer-events: none; opacity: .75; }
button.loading::after {
  content: '';
  width: 12px; height: 12px;
  border: 2px solid rgba(255,255,255,.35);
  border-top-color: white;
  border-radius: 50%;
  animation: spin .7s linear infinite;
  display: inline-block;
}
```

### 8.3 Reglas de uso

| Situación | Botón |
|-----------|-------|
| Acción principal de la pantalla | Primario |
| Cancelar / Volver | Secundario |
| Eliminar / Archivar masivo | Danger, con confirmación previa |
| Exportar / Ver detalle | Secundario o Ghost |
| Acciones en sidebar | Ghost o `sidebar-action-btn` |
| Admin-only | Agregar clase `.admin-only` para deshabilitar para Agentes |

### 8.4 Restricción por rol

```javascript
// En auth.js — deshabilita .admin-only si el usuario es Agente
window.restrictWriteIfAgent = function () {
  if (isAdmin()) return;
  document.querySelectorAll('.admin-only').forEach(el => {
    el.disabled = true;
    el.title = 'Solo administradores pueden ejecutar esta acción';
    el.classList.add('agent-disabled');
  });
};
```

---

## 9. Dashboards

### 9.1 Estructura recomendada

```
1. TOPBAR / SIDEBAR
   └── Identidad, navegación, usuario, tienda activa

2. STORE ROW (en módulos standalone)
   └── Cuenta activa + selector + info del módulo

3. STATUS BAR
   └── Feedback del último evento (idle / cargando / ok / error)

4. KPI SECTION
   └── 3–6 métricas clave, arriba del fold

5. FILTROS / BÚSQUEDA
   └── Panel sticky con search + selects + botón aplicar

6. TABLA O GRID PRINCIPAL
   └── Datos paginados, ordenables, exportables

7. BULK ACTION BAR (opcional)
   └── Aparece al seleccionar filas: contador + acciones masivas
```

### 9.2 Status bar

Siempre presente en módulos de datos. Indica el estado de la última operación.

```html
<div id="statusBar" class="status-bar">Listo.</div>
```

```javascript
function setStatus(msg, type = '') {
  const el = document.getElementById('statusBar');
  el.className = 'status-bar' + (type ? ' ' + type : '');
  el.textContent = msg;
}
// type: '' | 'info' | 'success' | 'error' | 'warning'
```

```css
.status-bar        { background: var(--bg); border: 1px solid var(--line); color: var(--muted); }
.status-bar.info   { background: var(--primary-soft); border-color: var(--primary-mid); color: var(--primary); }
.status-bar.success{ background: var(--success-bg); border-color: color-mix(in srgb, var(--success) 45%, transparent); color: var(--success); }
.status-bar.error  { background: var(--danger-bg); border-color: color-mix(in srgb, var(--danger) 45%, transparent); color: var(--danger); }
.status-bar.warning{ background: var(--warning-bg); border-color: color-mix(in srgb, var(--warning) 55%, transparent); color: var(--warning); }
```

### 9.3 Bulk action bar

Aparece solo cuando hay filas seleccionadas. Debe permanecer sticky en el bottom del viewport.

```html
<div id="bulkBar" class="bulk-bar" style="display:none;">
  <span id="bulkCount">0 seleccionados</span>
  <button class="secondary" onclick="clearSelection()">Limpiar selección</button>
  <button class="danger admin-only" onclick="bulkDelete()">Eliminar seleccionados</button>
</div>
```

### 9.4 Progreso de operaciones bulk

Para operaciones que procesan ítems en lote (importación, actualización masiva):

```html
<div class="progress-header">
  <span class="progress-label">Procesando...</span>
  <span class="progress-pct" id="pct">0%</span>
</div>
<div class="progress-track">
  <div class="progress-bar" id="progressBar" style="width:0%"></div>
</div>
<div class="progress-footer">
  <span class="progress-detail" id="progressDetail">0 / 0</span>
  <span class="progress-stats" id="progressStats"></span>
</div>
```

---

## 10. JavaScript

### 10.1 Organización de un módulo

Cada archivo HTML de módulo debe organizar su script en este orden:

```javascript
// ── 1. Constantes y estado ─────────────────────────────────
const PAGE_SIZE = 50;
let _allRows = [];
let _currentPage = 1;
let _sortCol = '';
let _sortAsc = true;

// ── 2. API calls ───────────────────────────────────────────
async function loadData() { ... }

// ── 3. Render ──────────────────────────────────────────────
function renderTable(rows) { ... }
function renderPagination(total) { ... }

// ── 4. Filtros y búsqueda ──────────────────────────────────
function applyFilters() { ... }
function normalizeText(v) { ... }

// ── 5. Acciones (CRUD) ─────────────────────────────────────
async function saveEdit() { ... }
async function deleteItem(id) { ... }
async function bulkAction() { ... }

// ── 6. UI helpers ──────────────────────────────────────────
function openModal(id) { ... }
function closeModal(id) { ... }
function setStatus(msg, type) { ... }
function setButtonLoading(btn, loading) { ... }

// ── 7. Init ────────────────────────────────────────────────
document.addEventListener('DOMContentLoaded', () => {
  requireAuth();
  renderUserChip();      // puebla #storeRow con selector de tienda + chip de usuario
  restrictWriteIfAgent();
  loadData();
});
```

### 10.2 Naming conventions

```javascript
// Variables: camelCase descriptivo
let activeFilter = 'all';
let selectedRows = [];

// Funciones: verbo + sustantivo
function loadData() {}
function renderTable(rows) {}
function applyFilters() {}
function openEditModal(id) {}
function exportToCSV() {}

// Constantes: UPPER_SNAKE_CASE
const PAGE_SIZE = 50;
const MAX_BULK = 500;

// IDs del DOM: kebab-case
// <div id="edit-modal">    ← para modales
// <button id="save-btn">   ← para botones con referencia
// <div id="statusBar">     ← excepción: convención establecida en el proyecto
```

### 10.3 Funciones de API

El backend es Google Apps Script. Todas las llamadas pasan por helpers definidos en `app.js`:

```javascript
// Llamada estándar (inyecta storeId automáticamente)
const data = await callApi('getProducts', { page: 1 });

// Llamada sin storeId (rutas de admin o auth)
const result = await callApiRaw('updateUser', { email, passwordHash });

// Llamada bulk (no falla en errores parciales)
const result = await callApiBulk('deleteItems', { ids });
```

Regla: **nunca hacer `fetch()` directo en un módulo**. Siempre usar los helpers de `app.js`.

### 10.4 Manejo de errores

```javascript
async function loadData() {
  setStatus('Cargando...', 'info');
  try {
    const data = await callApi('getItems');
    _allRows = data.items;
    renderTable(_allRows);
    setStatus(`${_allRows.length} registros cargados.`, 'success');
  } catch (err) {
    setStatus(`Error al cargar: ${err.message}`, 'error');
  }
}
```

- Nunca mostrar `err.stack` al usuario.
- Mensajes de error: breves, en primera persona del sistema, sin jerga técnica.
- Loguear en consola solo en desarrollo.

### 10.5 Seguridad

```javascript
// Escapado HTML — obligatorio antes de insertar datos externos en el DOM
function escapeHtml(value) {
  return String(value ?? '')
    .replace(/&/g, '&amp;').replace(/</g, '&lt;')
    .replace(/>/g, '&gt;').replace(/"/g, '&quot;');
}

// NUNCA hacer esto:
el.innerHTML = userData;           // ← XSS vulnerability

// SIEMPRE hacer esto:
el.innerHTML = escapeHtml(userData);   // ← seguro
// o mejor:
el.textContent = userData;            // ← más seguro aún cuando no necesitás HTML
```

### 10.6 Eventos

- Usar `addEventListener`, nunca atributos HTML `onclick=` salvo en handlers de modales inline donde el contexto está claro.
- Limpiar listeners en modales que se re-usan: clonar el elemento o usar el patrón de flag.
- Para store changes globales, usar el evento custom: `window.dispatchEvent(new Event('vtex:storeChanged'))`.

---

## 11. CSS

### 11.1 Variables CSS obligatorias

Todo proyecto debe definir en `:root` al menos:

```css
:root {
  /* Colores semánticos */
  --bg, --card, --text, --muted, --line
  --primary, --primary-hover, --primary-soft, --primary-mid
  --success, --success-bg
  --warning, --warning-bg
  --danger, --danger-bg, --danger-hover

  /* Espaciado */
  --radius, --radius-sm, --radius-pill
  --shadow-sm, --shadow, --shadow-lg

  /* Tipografía */
  --font, --mono
}
```

### 11.2 Naming de clases

```css
/* Componente */
.module-card {}
.kpi-grid {}
.status-bar {}

/* Modificador con guión */
.module-card.active {}
.status-bar.error {}
.button.secondary {}

/* Estado con clase separada (no &) */
.loading {}
.disabled {}
.pending {}

/* Elemento hijo con guión */
.module-card-title {}   ← evitar; preferir h3 semántico dentro de .module-card
```

Regla: **las clases describen qué es el elemento, no cómo se ve**. Preferir `.status-bar.error` sobre `.red-bar`.

### 11.3 Organización del archivo CSS

```css
/* 1. Import de fuentes */
@import url('...');

/* 2. Variables (:root) */
:root { ... }

/* 3. Reset y base */
*, body, a, ...

/* 4. Layout global */
.topbar, .sidebar, .app-layout, .content-area, .container, ...

/* 5. Tipografía */
h1, h2, h3, .eyebrow, .subtitle, ...

/* 6. Componentes (orden: más general → más específico) */
.panel, .hero-card, .module-card, .kpi-card, ...
button, .button, input, select, textarea, ...
table, th, td, .table-wrap, ...
.modal-overlay, .modal, ...
.status-bar, .notice, .bulk-bar, ...

/* 7. Utilidades */
.mono, .text-muted, .risk-alto, ...

/* 8. Responsive */
@media (max-width: 900px) { ... }
@media (max-width: 768px) { ... }
@media (max-width: 600px) { ... }

/* 9. Animaciones */
@keyframes spin { ... }
@keyframes fadeIn { ... }
```

### 11.4 Responsive

- **Mobile-last:** escribir el CSS base para desktop, luego usar `max-width` para adaptar.
- En móvil, los grids colapsan a 1 columna y el padding se reduce a 16–20px.
- Las tablas en mobile usan scroll horizontal (nunca `overflow: hidden` en `.table-wrap`).
- El sidebar se colapsa con transform (no display none) para mantener el layout shift mínimo.

---

## 12. Documentación

### 12.1 Archivos mínimos requeridos

Todo proyecto interno debe contener:

| Archivo | Propósito |
|---------|-----------|
| `README.md` | Descripción del proyecto, stack, cómo correr localmente, estructura de carpetas |
| `CHANGELOG.md` | Historial de versiones (o integrado en `config.js` como array) |
| `CLAUDE.md` | Instrucciones para Claude Code: stack, reglas de versionado, patrones del proyecto |
| `style_guide.md` | Referencia a este documento maestro (o copia local con overrides del proyecto) |

### 12.2 README.md — estructura mínima

```markdown
# Nombre del Proyecto

Descripción breve de una línea.

## Stack
- HTML / CSS / Vanilla JS
- Google Apps Script (backend)
- Google Sheets (base de datos)
- GitHub Pages (hosting)

## Cómo correr localmente
python -m http.server 3000

## Estructura
/
├── index.html          # Home / dashboard
├── login.html          # Autenticación
├── config.js           # Configuración global y versión
├── app.js              # Utilidades compartidas y helpers de API
├── auth.js             # Sesión y autenticación
├── styles.css          # Estilos globales
└── modules/
    ├── catalogo/
    ├── operaciones/
    ├── logistica/
    └── admin/

## Roles
- **Admin**: acceso total, puede escribir y eliminar
- **Agente**: lectura y exportación, sin escritura

## Versioning
Ver `config.js` → `CONFIG.VERSION` y `CONFIG.CHANGELOG`.
```

### 12.3 CLAUDE.md — instrucciones para el asistente

Incluir siempre:
- Stack y dependencias
- Regla de versionado (qué bump aplica a cada tipo de cambio)
- Cómo se actualiza la versión (dónde en config.js)
- Patrones de autenticación y seguridad del proyecto
- Convenciones de nombres específicas del proyecto

### 12.4 Versioning en config.js

```javascript
VERSION: {
  number: '1.5.0',          // semver
  date:   '2026-06-16',     // YYYY-MM-DD
  notes:  'Descripción igual que en CHANGELOG'
},

CHANGELOG: [
  { v: '1.5.0', date: '2026-06-16', desc: 'Descripción del cambio' },
  { v: '1.4.2', date: '2026-06-10', desc: '...' },
]
```

| Tipo de cambio | Bump |
|----------------|------|
| Nuevo módulo o feature visible | Minor `1.4.0 → 1.5.0` |
| Bug fix, mejora UX, estilo | Patch `1.4.0 → 1.4.1` |
| Cambio de arquitectura, breaking | Major `1.4.0 → 2.0.0` |
| Docs, comentarios, README | Sin bump |

---

## 13. Git

### 13.1 Branches

| Branch | Uso |
|--------|-----|
| `main` | Producción. Siempre deployable. |
| `feature/nombre-corto` | Desarrollo de una feature nueva |
| `fix/descripcion-bug` | Corrección de bug |
| `hotfix/descripcion` | Fix urgente directo a main |

Regla: **nunca commitear directamente a main en proyectos de equipo**. En proyectos personales, main directo es aceptable.

### 13.2 Commits

Formato: `tipo: descripción breve en infinitivo`

```
feat: agregar módulo de promociones
fix: corregir exportación CSV con caracteres especiales
style: ajustar padding del sidebar en mobile
refactor: extraer función normalizeText a app.js
docs: actualizar README con instrucciones de deploy
chore: bump versión a 1.5.0
```

Reglas:
- **Una línea descriptiva**, presente/infinitivo, sin punto final.
- Si el cambio es complejo, agregar cuerpo separado por línea en blanco.
- **Incluir `Co-Authored-By`** cuando el commit fue generado con asistencia de IA.
- El mensaje describe el **propósito**, no el mecanismo (`agregar sidebar` > `modificar index.html`).

### 13.3 Releases

- Cada versión Minor o Major merece un commit de bump con `chore: bump versión a X.Y.0`.
- El número de versión vive en `config.js`, no en `package.json` (proyectos sin Node).
- En proyectos con GitHub Pages: el push a `main` es el deploy. No existe paso adicional.

---

## 14. Estructura de proyecto

### 14.1 Estructura estándar

```
/
├── index.html                    # Home: dashboard o hub de módulos
├── login.html                    # Autenticación
│
├── config.js                     # Configuración global, versión, stores, changelog
├── app.js                        # Utilidades: CSV, API helpers, store mgmt
├── auth.js                       # Sesión, guards, renderUserChip
├── styles.css                    # Estilos globales
│
├── modules/                      # Módulos por área
│   ├── catalogo/
│   │   ├── categorias.html
│   │   ├── colecciones.html
│   │   └── ...
│   ├── operaciones/
│   │   ├── pedidos.html
│   │   └── promociones.html
│   ├── logistica/
│   │   ├── stock.html
│   │   └── estrategias-envio.html
│   └── admin/
│       ├── usuarios.html
│       └── roles.html
│
├── assets/                       # Recursos estáticos (solo si existen)
│   ├── img/
│   └── icons/
│
├── docs/                         # Documentación del proyecto
│   ├── style_guide.md            # Estándares visuales y técnicos
│   ├── ai_rules.md               # Reglas de colaboración con IA
│   ├── project_workflow.md       # Flujo operativo, freeze zones, aprendizajes
│   ├── gas-setup.md              # Setup del GAS paso a paso
│   └── decisions/                # ADRs (decisiones de arquitectura)
│
├── .claude/
│   ├── launch.json               # Configuración del servidor de preview
│   └── settings.json             # Permisos y hooks para Claude Code
│
├── README.md
├── CLAUDE.md                     # Instrucciones para Claude Code y Codex
└── AGENTS.md                     # Alias de CLAUDE.md para Codex
```

### 14.2 Convenciones de nombres de archivo

| Tipo | Convención | Ejemplo |
|------|-----------|---------|
| HTML de módulo | kebab-case | `score-productos.html` |
| JS compartido | camelCase corto | `app.js`, `auth.js` |
| CSS global | kebab-case | `styles.css` |
| Documentación | snake_case o UPPER | `style_guide.md`, `ai_rules.md`, `project_workflow.md` |
| Assets | kebab-case | `logo-vtex.svg` |

### 14.3 Carpetas de áreas

Las áreas de módulos siguen el mismo nombre que los `data-section` del HTML:

| Carpeta | data-section | Contenido |
|---------|-------------|-----------|
| `modules/catalogo/` | `"catalogo"` | Productos, SKUs, especificaciones |
| `modules/operaciones/` | `"operaciones"` | Pedidos, promociones |
| `modules/logistica/` | `"logistica"` | Stock, envíos |
| `modules/admin/` | `"admin"` | Usuarios, roles |

> **Nota:** El layout visual del shell (sidebar, topbar, panel de usuario, temas, versión) está documentado en [`application_shell.md`](application_shell.md). Esta sección §14 cubre solo la estructura de archivos y carpetas.

> **Documentación maestra compartida:** los archivos de estándares (`ai_rules.md`, `style_guide.md`, `apps_script_standards.md`, etc.) viven en `../project-standards/` relativo a cada proyecto. No se copian a los proyectos — cada proyecto los referencia desde su `CLAUDE.md` y `AGENTS.md`.

---

## 15. Integración con IA

Esta sección establece reglas para que ChatGPT, Claude Code, Codex y futuros asistentes respeten el estándar sin generar desvíos.

### 15.1 Reglas generales para cualquier asistente

1. **Leer este documento antes de proponer cambios de UI o arquitectura.**
2. **No introducir dependencias externas** (npm packages, CDNs de componentes, frameworks) sin consultar.
3. **No cambiar el sistema de colores** — siempre usar las variables CSS del proyecto (`--primary`, `--danger`, etc.).
4. **No cambiar la fuente** — DM Sans y DM Mono son la identidad tipográfica.
5. **No crear nuevos archivos de estilo** — todo CSS va en `styles.css` o en `<style>` inline solo si es estrictamente específico de ese módulo y no reutilizable.
6. **Respetar el versionado** — todo cambio funcional debe actualizar `config.js` según la tabla de bumps.
7. **No agregar comentarios obvios** en el código — solo comentar el *por qué* cuando no es deducible.
8. **No agregar manejo de errores para escenarios imposibles** — confiar en las garantías del framework y del backend.

### 15.2 Para Claude Code (CLAUDE.md)

El archivo `CLAUDE.md` en la raíz de cada proyecto es la fuente de verdad para el asistente. Debe incluir:

```markdown
## Versionado obligatorio
[reglas de bump del proyecto]

## Stack
[lista de tecnologías]

## Patrones de seguridad
[cómo se manejan credenciales y sesiones]

## Convenciones locales
[cualquier desviación del style guide maestro específica del proyecto]
```

Claude Code debe:
- Leer `CLAUDE.md` al inicio de cada sesión de trabajo.
- Actualizar `config.js` **antes del commit** en cada cambio funcional.
- No hacer push sin confirmación explícita del usuario.
- Verificar visualmente los cambios con el preview server antes de reportar la tarea como completa.

### 15.3 Para ChatGPT / Codex

Al iniciar una conversación sobre un proyecto, proveer siempre:

```
Este proyecto usa:
- HTML/CSS/Vanilla JS, sin frameworks
- Variables CSS: --primary (#1a3f6b), --success (#0a7040), --warning (#92400e), --danger (#991b1b)
- Font: DM Sans (UI) + DM Mono (datos numéricos)
- Border radius: 8px (componentes), 14px (cards)
- Cualquier nuevo componente debe seguir el patrón de styles.css existente
- No introducir dependencias externas
- El backend es Google Apps Script; las llamadas API van por callApi() en app.js
```

### 15.4 Checklist antes de aplicar código generado por IA

Antes de integrar código sugerido por cualquier asistente, verificar:

- [ ] ¿Usa las variables CSS del proyecto o hardcodea colores?
- [ ] ¿Introduce alguna dependencia externa?
- [ ] ¿Respeta la jerarquía tipográfica (DM Sans / DM Mono)?
- [ ] ¿Los nuevos componentes siguen el naming de clases existente?
- [ ] ¿Hay llamadas directas a `fetch()` en vez de usar `callApi()`?
- [ ] ¿Se incluye `escapeHtml()` donde se insertan datos del usuario o de la API?
- [ ] ¿Se actualiza `config.js` con el nuevo número de versión?
- [ ] ¿El mensaje de commit describe el propósito del cambio, no el mecanismo?

### 15.5 Lo que la IA NO debe hacer sin consultar

- Cambiar el layout principal de una pantalla (de topbar a sidebar, de hub a SPA, etc.)
- Reemplazar `styles.css` completo
- Cambiar el sistema de autenticación o el manejo de sesiones
- Agregar una librería de componentes (Bootstrap, Tailwind, Ant Design, etc.)
- Cambiar el backend de GAS a otro servicio
- Subir o publicar información del proyecto a servicios externos (pastebins, diagramas online, etc.)

---

## 16. Accesibilidad

> **Nivel objetivo:** WCAG 2.1 AA para productos internos. El foco es no-bloquear a usuarios de teclado ni lectores de pantalla — no se requiere certificación formal.

### 16.1 HTML semántico primero

Usar el elemento correcto antes de recurrir a ARIA. ARIA solo complementa; no reemplaza semántica nativa.

| Caso | Correcto | Incorrecto |
|------|---------|-----------|
| Acción que navega | `<a href="...">` | `<div onclick="...">` |
| Acción que ejecuta | `<button>` | `<div onclick="...">` |
| Listado de ítems | `<ul>/<ol>` | `<div>` anidados |
| Encabezados de sección | `<h2>`, `<h3>` (jerarquía coherente) | `<p class="titulo">` |
| Datos tabulares | `<table>` con `<th scope="col">` | `<div>` con CSS de grid |

### 16.2 Atributos ARIA esenciales

```html
<!-- Icono sin texto visible -->
<button aria-label="Cerrar panel">✕</button>

<!-- Elemento que describe a otro -->
<input id="email" aria-describedby="email-hint" />
<p id="email-hint">Usá tu correo corporativo.</p>

<!-- Landmark regions -->
<nav aria-label="Navegación principal">...</nav>
<main>...</main>
<aside aria-label="Panel de usuario">...</aside>

<!-- Estado de carga -->
<div role="status" aria-live="polite" id="statusBar">Cargando...</div>
```

**Reglas:**
- `aria-label` cuando no hay texto visible y no hay elemento de referencia.
- `aria-labelledby` cuando el label ya existe en el DOM — apuntar a su `id`.
- `aria-describedby` para hints, errores y tooltips secundarios (no el label principal).
- No poner `aria-label` en elementos que ya tienen texto visible — es redundante y confunde.

### 16.3 Modales y diálogos

```html
<div role="dialog" aria-modal="true" aria-labelledby="modal-title" id="miModal">
  <h2 id="modal-title">Confirmar acción</h2>
  <p>¿Deseas eliminar este registro?</p>
  <button id="modal-confirm">Confirmar</button>
  <button id="modal-cancel">Cancelar</button>
</div>
```

**Checklist de modal:**
- `role="dialog"` + `aria-modal="true"` en el contenedor.
- `aria-labelledby` apuntando al `<h2>` o título del modal.
- Al abrir: mover el foco al primer elemento interactivo del modal (o al contenedor con `tabindex="-1"`).
- Al cerrar: devolver el foco al elemento que abrió el modal (`document.activeElement` guardado antes de abrir).
- Bloquear el Tab dentro del modal (focus trap) mientras está abierto.
- `Escape` cierra el modal.

### 16.4 Gestión de foco

```js
// Guardar referencia antes de abrir
const _focusOrigin = document.activeElement;

function openModal(modalId) {
  const modal = document.getElementById(modalId);
  modal.removeAttribute('hidden');
  // Mover foco al primer elemento interactivo
  const first = modal.querySelector('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
  if (first) first.focus();
}

function closeModal(modalId) {
  document.getElementById(modalId).setAttribute('hidden', '');
  // Devolver foco al origen
  if (_focusOrigin) _focusOrigin.focus();
}
```

**Reglas de foco:**
- Nunca quitar `outline` sin reemplazarlo: `outline: none` sin alternativa rompe la navegación por teclado.
- Los estilos de focus de este proyecto usan `var(--primary-mid)` como anillo de foco — no sobreescribir sin reemplazar.
- `tabindex="-1"` permite recibir foco por JS sin entrar en el Tab flow natural.
- `tabindex="0"` añade un elemento custom al Tab flow (usar solo si el elemento no tiene rol interactivo nativo).
- Valores de `tabindex` positivos (`tabindex="1"`, `tabindex="2"`) están prohibidos — rompen el orden natural.

### 16.5 Regiones dinámicas (aria-live)

```html
<!-- Mensajes de estado no urgentes (carga completada, éxito) -->
<div role="status" aria-live="polite" id="statusBar" class="status-bar"></div>

<!-- Errores urgentes -->
<div role="alert" aria-live="assertive" id="errorAlert"></div>
```

| Tipo | Atributo | Cuándo |
|------|---------|--------|
| `role="status"` | `aria-live="polite"` | Mensajes informativos: "Cargando…", "Guardado." |
| `role="alert"` | `aria-live="assertive"` | Errores críticos que requieren atención inmediata |

**Nota:** el `statusBar` del template de Apéndice A ya usa `role="status"` — mantenerlo.  
No usar `aria-live="assertive"` para mensajes informativos de rutina — interrumpe al lector de pantalla.

### 16.6 Navegación por teclado

Teclas estándar que deben funcionar sin JS adicional en elementos nativos:

| Tecla | Elemento | Comportamiento esperado |
|-------|---------|------------------------|
| `Tab` / `Shift+Tab` | Todos | Navegar entre elementos interactivos en orden del DOM |
| `Enter` | `<a>`, `<button>` | Activar |
| `Space` | `<button>`, `<input type="checkbox">` | Activar / marcar |
| `Escape` | Modal, dropdown abierto | Cerrar y devolver foco |
| `Arrow keys` | `<select>`, grupos de radio | Cambiar opción |

**Casos que requieren JS:**
- Modales: bloquear Tab fuera del modal mientras está abierto.
- Dropdowns custom: Arrow Up/Down para navegar ítems; Enter para seleccionar; Escape para cerrar.
- Tabs de navegación: Arrow Left/Right para cambiar tab activo (patrón ARIA Tabs).

### 16.7 Imágenes y contraste

**Imágenes:**
```html
<!-- Imagen informativa -->
<img src="logo.png" alt="Logo de Sporting" />

<!-- Imagen decorativa (no aporta información) -->
<img src="separador.png" alt="" role="presentation" />
```

**Contraste (referencia a tokens de este sistema):**

| Uso | Token | Contraste mínimo |
|-----|-------|-----------------|
| Texto principal | `--text` (#111827) sobre `--bg` (#f0f2f7) | ✅ > 7:1 |
| Texto secundario | `--muted` (#6b7280) sobre `--card` (#fff) | ✅ > 4.5:1 |
| Texto en botón primary | blanco sobre `--primary` (#1a3f6b) | ✅ > 7:1 |
| Texto en tag `--primary-soft` | `--primary` sobre `--primary-soft` | ✅ > 4.5:1 |

> No crear variantes de color custom sin verificar contraste. Usar [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) o el panel de accesibilidad de DevTools.

### 16.8 Formularios accesibles

```html
<!-- Siempre label explícito o aria-label -->
<label for="usuario">Usuario</label>
<input id="usuario" type="text" aria-required="true" autocomplete="username" />

<!-- Error de validación -->
<input id="email" type="email" aria-invalid="true" aria-describedby="email-error" />
<span id="email-error" role="alert">El email no tiene un formato válido.</span>

<!-- Fieldset para grupos de opciones -->
<fieldset>
  <legend>Tipo de acceso</legend>
  <label><input type="radio" name="rol" value="admin" /> Admin</label>
  <label><input type="radio" name="rol" value="agente" /> Agente</label>
</fieldset>
```

**Reglas:**
- Todo `<input>` y `<select>` debe tener `<label>` asociado por `for`/`id`, o `aria-label`.
- `placeholder` no reemplaza al `<label>` — desaparece al escribir y no es legible por todos los lectores.
- `aria-required="true"` en campos obligatorios (complementa el atributo `required`).
- `aria-invalid="true"` + `aria-describedby` al error cuando la validación falla.

### 16.9 Checklist para la IA

Antes de entregar código con UI, verificar:

- [ ] ¿Todos los `<input>` y `<select>` tienen `<label>` o `aria-label`?
- [ ] ¿Los modales tienen `role="dialog"`, `aria-modal="true"` y `aria-labelledby`?
- [ ] ¿El foco se mueve al abrir un modal y regresa al cerrarlo?
- [ ] ¿Los botones de icono tienen `aria-label`?
- [ ] ¿El `statusBar` usa `role="status"`? ¿Los errores críticos usan `role="alert"`?
- [ ] ¿No hay `tabindex` positivos?
- [ ] ¿El texto en custom colors tiene ratio > 4.5:1?
- [ ] ¿Las imágenes informativas tienen `alt` descriptivo? ¿Las decorativas tienen `alt=""`?
- [ ] ¿`Escape` cierra modales y dropdowns?
- [ ] ¿No se usó `outline: none` sin alternativa visible?

---

## Apéndice A — Snippet de módulo nuevo

Template mínimo para un nuevo módulo standalone:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Nombre del Módulo — VTEX Control Center</title>
  <link rel="stylesheet" href="../../styles.css" />
</head>
<body>

  <header class="topbar">
    <div class="topbar-copy">
      <p class="eyebrow">Área</p>
      <h1>Nombre del Módulo</h1>
      <p class="subtitle">Descripción breve de qué hace este módulo.</p>
    </div>
    <a class="button secondary" href="../../index.html">← Volver al inicio</a>
  </header>

  <div id="storeRow" class="store-row"></div>

  <main class="container">

    <section class="panel">
      <div class="toolbar">
        <input type="search" id="moduleSearch" placeholder="Buscar..." />
        <button class="admin-only" onclick="openCreateModal()">+ Nuevo</button>
        <button class="secondary" onclick="exportCSV()">Exportar CSV</button>
      </div>
      <div id="statusBar" class="status-bar">Seleccioná una tienda para comenzar.</div>
    </section>

    <section class="panel kpi-section">
      <div class="kpi-grid" id="kpiGrid"></div>
    </section>

    <section class="panel">
      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Columna 1</th>
              <th>Columna 2</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody id="tableBody"></tbody>
        </table>
      </div>
      <div class="pagination" id="pagination"></div>
    </section>

  </main>

  <div id="modals"></div>

  <script src="../../config.js"></script>
  <script src="../../auth.js"></script>
  <script src="../../app.js"></script>
  <script>
    const PAGE_SIZE = 50;
    let _allRows = [], _currentPage = 1;

    async function loadData() {
      const store = getActiveStore();
      if (!store) return setStatus('Seleccioná una tienda.', 'warning');
      setStatus('Cargando...', 'info');
      try {
        const data = await callApi('getItems');
        _allRows = data.items || [];
        renderTable(_allRows);
        setStatus(`${_allRows.length} registros.`, 'success');
      } catch (err) {
        setStatus(`Error: ${err.message}`, 'error');
      }
    }

    function renderTable(rows) {
      const tbody = document.getElementById('tableBody');
      if (!rows.length) {
        tbody.innerHTML = '<tr><td colspan="3" style="text-align:center;color:var(--muted)">Sin resultados</td></tr>';
        return;
      }
      tbody.innerHTML = rows.map(r => `
        <tr>
          <td>${escapeHtml(r.col1)}</td>
          <td>${escapeHtml(r.col2)}</td>
          <td><button class="secondary" onclick="openEditModal('${escapeHtml(r.id)}')">Editar</button></td>
        </tr>
      `).join('');
    }

    function setStatus(msg, type = '') {
      const el = document.getElementById('statusBar');
      el.className = 'status-bar' + (type ? ' ' + type : '');
      el.textContent = msg;
    }

    document.addEventListener('DOMContentLoaded', () => {
      requireAuth();
      renderUserChip();
      restrictWriteIfAgent();
      loadData();
    });

    window.addEventListener('vtex:storeChanged', loadData);
  </script>
</body>
</html>
```

---

## Apéndice B — Paleta de referencia rápida

```
Primary dark:    #0b2641   (topbar top — ya no se usa en sidebar)
Primary mid:     #1a3f6b   (--primary, topbar)
Primary light:   #e8f0fb   (--primary-soft, tag bg)
Primary border:  #bfd4f5   (--primary-mid, focus ring)

Success dark:    #0a7040   (--success)
Success light:   #ecfdf5   (--success-bg)
Success border:  #a7f3d0

Warning dark:    #92400e   (--warning)
Warning light:   #fffbeb   (--warning-bg)
Warning border:  #fcd34d

Danger dark:     #991b1b   (--danger)
Danger hover:    #7f1d1d   (--danger-hover)
Danger light:    #fef2f2   (--danger-bg)
Danger border:   #fca5a5

Neutral text:    #111827   (--text)
Neutral muted:   #6b7280   (--muted)
Neutral line:    #e5e7eb   (--line)
Neutral bg:      #f0f2f7   (--bg)
Neutral card:    #ffffff   (--card)

Sidebar bg:      #f1f5f9   (--sidebar-bg, slate-100)
Sidebar line:    #e2e8f0   (--sidebar-line, slate-200)
```

---

*Documento mantenido por el equipo de Productos Web. Actualizar junto con cada cambio de estándar que afecte a 2+ proyectos.*
