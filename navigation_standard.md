# Estándar de Navegación — Aplicaciones Internas Luquin

**Versión:** 1.0.0  
**Fecha:** 2026-06-27  
**Autor:** Gabriel Luna — Productos Web  
**Alcance:** AuditCS · VTEX Control Center · Project Control Center · Commerce Hub · Futuras apps internas

> **Complementa:** `application_shell.md` (layout, sidebar, panel de usuario) · `style_guide.md §5.X` (componentes chip y dropdown)

---

## 1. Estructura canónica del sidebar

Todas las aplicaciones Luquin siguen esta jerarquía de elementos en el sidebar, en este orden:

```
SIDEBAR (224px fijo)
│
├─ [MARCA]
│   ├─ Ícono (34×34, gradiente --primary)
│   ├─ Nombre de la app (13px/700)
│   └─ Badge de versión → popover de changelog
│
├─ [STORE SELECTOR]  ← solo apps multi-tienda
│
├─ [OVERVIEW]  ← sección sin etiqueta, siempre el primer ítem de nav
│   └─ Home / Dashboard / Inicio
│
├─ [SECCIÓN A]  ← lectura y visualización
│   etiqueta: "Reportes" / "Análisis" / "Vistas"
│   └─ Módulos de consulta, reportes y dashboards secundarios
│
├─ [SECCIÓN B]  ← escritura y operaciones
│   etiqueta: "Gestión" / "Operaciones"
│   └─ Formularios, cargas, gestión de registros
│
├─ [SISTEMA]  ← solo admin, oculto para otros roles
│   └─ Configuración / Settings
│
├─ ─── spacer (flex: 1) ───
│
├─ [ESTADO DEL SISTEMA]  ← opcional, según app
│   └─ Chip de conexión (Sheets / API status)
│
└─ [FOOTER DE USUARIO]
    └─ Chip compacto → dropdown (ver §3)
```

---

## 2. Reglas de secciones

### 2.1 Sección Overview (sin etiqueta)

- El Home/Dashboard va **siempre primero**, sin etiqueta de sección (`div.nav-section`).
- Nunca agrupa con "Reportes" ni ninguna otra sección.
- Es el punto de entrada de la app — separarlo visualmente refuerza esta jerarquía.
- Si la app tiene un solo ítem (solo Inicio), puede omitirse la sección y poner el ítem directamente.

### 2.2 Sección de Lectura — "Reportes"

- Contiene vistas de **solo consulta**: dashboards secundarios, rankings, analytics, registros históricos de solo lectura.
- No contiene formularios de creación ni acciones de escritura.
- Nomenclatura recomendada: **Reportes** (apps de auditoría/calidad), **Análisis** (apps de datos), **Vistas** (apps operativas con múltiples vistas fijas).

### 2.3 Sección de Escritura — "Gestión"

- Contiene módulos que **crean, editan o cargan** datos: formularios, cargas de archivos, tablas con acciones CRUD.
- No contiene dashboards de solo lectura.
- Nomenclatura recomendada: **Gestión** (apps de auditoría/calidad), **Operaciones** (apps operativas).

### 2.4 Sección de Administración — "Sistema"

- Contiene la pantalla de **Configuración** del sistema (usuarios, roles, parámetros, integraciones).
- Visible **solo** para el rol Administrador (`isAdmin()` → `id_rol === 1`).
- Ocultar vía `applyRoleRestrictions()` en JS, no vía CSS hardcodeado.
- Etiqueta canónica: **Sistema**.

### 2.5 Casos borde

| Situación | Solución |
|-----------|----------|
| Un módulo sirve tanto para consulta como para escritura | Clasificar según la **acción principal**. Si el módulo es mayoritariamente lectura con edición secundaria → Reportes. Si el flujo principal es crear/editar → Gestión. |
| La app tiene pocos módulos (3 o menos) | Las secciones son opcionales; se pueden listar los ítems directamente sin etiquetas de sección. |
| Un módulo es visible solo para algunos roles, no solo para admin | Usar `applyRoleRestrictions()` basado en `canView(mod)`. No crear una sección exclusiva; ocultarlo individualmente. |

---

## 3. Área de usuario — chip + dropdown

### 3.1 Estructura

El footer del sidebar contiene **un único chip compacto** que abre un dropdown con todas las acciones del usuario:

```
[ GL ]  Gabriel Luna  ·  Administrador  [▾]
```

Al hacer clic, abre un dropdown encima del chip con:

```
┌─────────────────────────────┐
│ Gabriel Luna                │
│ gabriel@empresa.com         │
│ [Administrador]             │
├─────────────────────────────┤
│ ☾  Modo oscuro       [○●]   │
├─────────────────────────────┤
│ Cambiar contraseña          │
├─────────────────────────────┤
│ Cerrar sesión               │
└─────────────────────────────┘
```

### 3.2 Qué NO va en el footer del sidebar

| Elemento | Dónde va en cambio |
|----------|--------------------|
| Config operativa de la app (responsable, epic, parámetros) | Dashboard widget o tab Parámetros en Configuración |
| Toggle de tema como botón a ancho completo independiente | Dentro del dropdown del usuario |
| Botones apilados (tema + contraseña + logout) | Todos dentro del dropdown del usuario |
| Email visible permanentemente | Solo visible dentro del dropdown, no en el chip |

### 3.3 Implementación

Ver `application_shell.md §4.7` para el código JS completo de `renderSidebarUser()` y los helpers `_openUserDropdown()`, `_closeUserDropdown()`, `_toggleUserDropdown()`.

Ver `style_guide.md §5.X` para el CSS canónico de `.user-chip`, `.user-avatar`, `.user-dropdown`, `.user-dropdown-item`.

---

## 4. Módulo de Configuración — tabs estándar

Toda app con RBAC tiene una pantalla **Configuración** (Sistema) con este orden de tabs:

| # | Tab | Contenido |
|---|-----|-----------|
| 1 | **Parámetros** | Configuración específica de la app (pesos, umbrales, listas, etc.) |
| 2 | **Usuarios** | Gestión CRUD de usuarios (crear, editar, activar/desactivar) |
| 3 | **Roles y permisos** | Matriz RBAC: una fila por módulo, selector 3 estados (Oculto / Ver / Ver+editar) |
| 4 | **Integraciones** | Conexiones con servicios externos (Sheets, APIs, webhooks) + estado de salud |

**Reglas:**
- El tab "Integraciones" reemplaza a "Conexión" en todos los proyectos. "Conexión" implica un único punto de conexión; "Integraciones" escala a múltiples servicios.
- Los identificadores internos del tab (valor del atributo `data-cfg-tab`, claves JS) pueden mantener el string `conexion` por compatibilidad. Solo el texto visible al usuario cambia.
- El Administrador (`id_rol === 1`) es el único rol que puede ver la pantalla de Configuración completa.

---

## 5. Mapas de navegación por aplicación

### AuditCS (Customer Service Control Center)

```
[sin etiqueta]     Inicio (dashboard)
[REPORTES]         Agentes · Observaciones
[GESTIÓN]          Nueva auditoría · Productividad semanal · Registros
[SISTEMA]          Configuración
  └─ tabs:         Parámetros · Usuarios · Roles y permisos · Integraciones
```

### VTEX Control Center *(por definir)*

```
[sin etiqueta]     Inicio / Dashboard
[REPORTES]         (vistas analíticas por definir)
[GESTIÓN]          (operaciones por definir)
[SISTEMA]          Configuración
  └─ tabs:         Parámetros · Usuarios · Roles y permisos · Integraciones
```

### Project Control Center *(por definir)*

```
[sin etiqueta]     Inicio / Dashboard
[PROYECTOS]        (vistas de proyectos por definir)
[GESTIÓN]          (operaciones por definir)
[SISTEMA]          Configuración
  └─ tabs:         Parámetros · Usuarios · Roles y permisos · Integraciones
```

### Commerce Hub *(por definir)*

```
[sin etiqueta]     Inicio / Dashboard
[REPORTES]         (vistas analíticas por definir)
[GESTIÓN]          (operaciones por definir)
[SISTEMA]          Configuración
  └─ tabs:         Parámetros · Usuarios · Roles y permisos · Integraciones
```

> Los mapas marcados como *"por definir"* se completan cuando el proyecto inicia. El estándar de estructura de secciones aplica desde el primer módulo.

---

## 6. Checklist de conformidad

Al crear o auditar la navegación de una app, verificar:

- [ ] Home/Dashboard está en su propia sección sin etiqueta, siempre primero
- [ ] Secciones nombradas según el estándar (Reportes / Gestión / Sistema)
- [ ] Sección Sistema visible solo para `isAdmin()`
- [ ] Footer de usuario implementado como chip + dropdown (no botones apilados)
- [ ] Dropdown cerrado al hacer clic fuera y al presionar Escape
- [ ] Toggle de tema vive en el dropdown del usuario, no como botón independiente
- [ ] Tab de conexiones externa llamado "Integraciones" (no "Conexión")
- [ ] Orden de tabs de Configuración: Parámetros → Usuarios → Roles → Integraciones
- [ ] Texto estático de config operativa (responsable, parámetros) fuera del sidebar footer
