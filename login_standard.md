# Login — Estándar de Autenticación

**Versión:** 1.0.0  
**Fecha:** 2026-06-24  
**Autor:** Gabriel Luna — Productos Web  
**Alcance:** VTEX Control Center · Commerce Hub · Project Control Center · Customer Service Control Center · Herramientas internas  
**Implementación de referencia:** `commerce-hub/login.html` + `src/js/auth.js`

> **Complementa:** `apps_script_standards.md §7` (Auth.gs, validación de sesiones server-side) · `style_guide.md §7` (formularios) · `style_guide.md §16` (accesibilidad de formularios de login) · `google_sheets_standards.md` (tabla SESIONES)

---

## Índice

1. [Propósito](#1-propósito)
2. [Estructura de la pantalla](#2-estructura-de-la-pantalla)
3. [Identidad visual por proyecto](#3-identidad-visual-por-proyecto)
4. [CSS canónico](#4-css-canónico)
5. [Campos y formulario](#5-campos-y-formulario)
6. [Validaciones y mensajes de error](#6-validaciones-y-mensajes-de-error)
7. [Estados de carga](#7-estados-de-carga)
8. [Manejo de sesión](#8-manejo-de-sesión)
9. [Cambio de contraseña](#9-cambio-de-contraseña)
10. [Recordar usuario](#10-recordar-usuario)
11. [Tema y anti-flash](#11-tema-y-anti-flash)
12. [Responsive](#12-responsive)
13. [Versión de la aplicación](#13-versión-de-la-aplicación)
14. [Pie institucional](#14-pie-institucional)
15. [Plantilla HTML canónica](#15-plantilla-html-canónica)
16. [Checklist de conformidad](#16-checklist-de-conformidad)

---

## 1. Propósito

Este documento define el **estándar de login** para todas las aplicaciones internas del ecosistema. El objetivo es garantizar una experiencia de acceso idéntica en todos los proyectos — mismo layout, misma lógica, mismos mensajes — de forma que cualquier proyecto nuevo pueda replicarla sin decisiones de diseño adicionales.

Reglas no negociables:

- Login como **página dedicada** (`login.html`) — nunca overlay ni modal encima de otra pantalla.
- **Sin Google Fonts** — las fuentes se cargan desde el CSS del proyecto (`main.css`).
- **Sin frameworks** — HTML + CSS + Vanilla JS.
- La **contraseña se hashea SHA-256 en el frontend** antes de enviarla al backend.
- **Nunca** se envía la contraseña en texto plano.

---

## 2. Estructura de la pantalla

```
┌─────────────────────────────────┐
│         (fondo: --bg)           │
│                                 │
│         ┌──────────┐            │
│         │  logo 48 │            │  ← .login-logo-icon  (icono cuadrado redondeado)
│         └──────────┘            │
│         Nombre de App           │  ← .login-app-name
│         Subtítulo · Empresa     │  ← .login-app-sub
│                                 │
│    ┌────────────────────────┐   │
│    │  Correo electrónico    │   │  ← .field > label + input[type=email]
│    │  [___________________] │   │
│    │                        │   │
│    │  Contraseña            │   │  ← .field > label + input[type=password]
│    │  [___________________] │   │
│    │                        │   │
│    │  [    Ingresar    ]    │   │  ← .btn-login (full-width)
│    │                        │   │
│    │  ┌──────────────────┐  │   │  ← .alert-err (oculto hasta error)
│    │  │  Mensaje error   │  │   │
│    │  └──────────────────┘  │   │
│    └────────────────────────┘   │
│                                 │
│    Herramienta interna…         │  ← .login-footer
│                                 │
└─────────────────────────────────┘
```

**Centrado vertical y horizontal** (`display:flex; align-items:center; justify-content:center; min-height:100vh`).  
**Ancho máximo del contenedor:** `380px`.  
**Header** (logo + nombre) encima de la card, no dentro.

---

## 3. Identidad visual por proyecto

Cada proyecto personaliza el icono, nombre y subtítulo. El resto del layout es idéntico.

| Proyecto | Sigla icono | Nombre en pantalla | Subtítulo |
|----------|-------------|-------------------|-----------|
| Commerce Hub | `CH` | Commerce Hub | eCommerce Ops · Luquin |
| Project Control Center | `PC` | Project Control Center | Gestión de Proyectos · Luquin |
| VTEX Control Center | `VC` | VTEX Control Center | Operaciones VTEX · Luquin |
| Customer Service | `CS` | Customer Service | Atención al Cliente · Luquin |

**Regla**: la sigla del icono es **siempre 2 caracteres**, en mayúsculas, `font-weight:800`. Nunca 3 o más.

---

## 4. CSS canónico

Todo el CSS del login es **inline en `<style>`** dentro de `login.html`. No se importan hojas adicionales salvo `main.css` del proyecto (que provee los tokens CSS).

```css
/* ── Contenedor centrado ─────────────────────────────────────── */
body {
  display: flex; align-items: center; justify-content: center;
  min-height: 100vh; padding: 24px;
}
.login-wrap { width: 100%; max-width: 380px; }

/* ── Header (logo + nombre) ──────────────────────────────────── */
.login-header  { text-align: center; margin-bottom: 24px; }
.login-logo-icon {
  width: 48px; height: 48px;
  background: linear-gradient(135deg, var(--primary), var(--primary-hover));
  border-radius: var(--radius-lg);                /* ~14-20px */
  display: inline-flex; align-items: center; justify-content: center;
  font-size: 18px; font-weight: 800; color: #fff; letter-spacing: -.5px;
  margin-bottom: 12px;
  box-shadow: 0 2px 8px color-mix(in srgb, var(--primary) 30%, transparent);
}
.login-app-name { font-size: 18px; font-weight: 700; color: var(--text); letter-spacing: -.3px; }
.login-app-sub  { font-size: 13px; color: var(--text2, var(--muted)); margin-top: 3px; }

/* ── Card (formulario) ───────────────────────────────────────── */
.login-card {
  background: var(--card, var(--s1, #fff));
  border: 1px solid var(--border, var(--line));
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow);
  padding: 28px;
}

/* ── Campos ──────────────────────────────────────────────────── */
.field { margin-bottom: 16px; }
.field label {
  display: block; font-size: 13px; font-weight: 500;
  color: var(--text); margin-bottom: 6px;
}
.field input {
  width: 100%; padding: 9px 12px; font-size: 14px; font-family: inherit;
  border: 1px solid var(--border, var(--line));
  border-radius: var(--radius, var(--radius-sm));
  outline: none; transition: border-color .15s, box-shadow .15s;
  color: var(--text); background: var(--bg);
  box-sizing: border-box;
}
.field input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 15%, transparent);
}

/* ── Botón Ingresar ──────────────────────────────────────────── */
.btn-login {
  width: 100%; padding: 10px; font-family: inherit;
  font-size: 14px; font-weight: 600; border: none;
  background: var(--primary); color: #fff;
  border-radius: var(--radius, var(--radius-sm));
  cursor: pointer; transition: background .15s; margin-top: 4px;
}
.btn-login:hover:not(:disabled) { background: var(--primary-hover); }
.btn-login:disabled { opacity: .6; cursor: default; }

/* ── Mensaje de error ────────────────────────────────────────── */
.alert-err {
  margin-top: 14px; padding: 10px 14px;
  background: var(--danger-bg, #fef2f2);
  border: 1px solid color-mix(in srgb, var(--danger) 30%, transparent);
  border-radius: var(--radius, var(--radius-sm));
  color: var(--danger, #991b1b); font-size: 13px;
}

/* ── Pie ─────────────────────────────────────────────────────── */
.login-footer {
  text-align: center; margin-top: 20px;
  font-size: 12px; color: var(--text2, var(--muted));
}
```

> **Nota de tokens**: los proyectos usan nombres distintos para algunos tokens (`--border` vs `--line`, `--s1` vs `--card`). Usar `var(--primary, fallback)` donde corresponda para que el CSS del login sea portable entre proyectos.

---

## 5. Campos y formulario

**Campos obligatorios:**

| Campo | Tipo HTML | `autocomplete` | Placeholder |
|-------|-----------|----------------|-------------|
| Correo electrónico | `email` | `email` | `usuario@empresa.com` |
| Contraseña | `password` | `current-password` | `••••••••` |

**Reglas del formulario:**
- `autofocus` en el campo de email.
- `autocomplete="on"` en el `<form>` para permitir que el browser guarde credenciales.
- El submit solo es posible con ambos campos completos — validar antes del fetch.
- IDs de los inputs: `login-email` y `login-pwd` (canonical).

---

## 6. Validaciones y mensajes de error

### Validaciones client-side (antes del fetch)

| Condición | Mensaje |
|-----------|---------|
| Email o contraseña vacíos | `Completá los dos campos.` |

### Errores de red / backend

| Condición | Mensaje |
|-----------|---------|
| `res.ok === false` (HTTP error) | `Error de conexión (STATUS).` |
| `json.ok === false` | `json.error` del backend, o `Credenciales incorrectas.` |
| Sin URL de API configurada | `No hay URL de API configurada.` |
| Catch genérico | `Error al iniciar sesión. Verificá tu conexión.` |

### Presentación del error

```js
// CORRECTO — usar hidden attribute (no style.display)
errEl.hidden = true;   // ocultar al inicio del submit
errEl.textContent = mensaje;
errEl.hidden = false;  // mostrar si hay error
```

```html
<!-- El div de error empieza oculto con el atributo hidden -->
<div class="alert-err" id="login-err" hidden></div>
```

**Anti-patrón a evitar:**
```js
// INCORRECTO — style.display='' vuelve al CSS display:none
errEl.style.display = '';
```

---

## 7. Estados de carga

```js
// Al iniciar el submit
btn.disabled    = true;
btn.textContent = 'Verificando…';

// Al finalizar (éxito o error)
btn.disabled    = false;
btn.textContent = 'Ingresar';
```

El botón se deshabilita durante el fetch para evitar doble envío. El texto cambia a `Verificando…` como feedback visual. El bloque `finally` siempre restaura el botón (salvo que haya redirección por login exitoso).

---

## 8. Manejo de sesión

### Estructura de sesión (localStorage)

```js
// Clave: '<prefix>_session'  (ej: 'ch_session', 'pcc_session')
{
  session_token: "uuid-v4",
  usuario: {
    id:        1,
    nombre:    "Gabriel Luna",
    email:     "gabriel.luna@empresa.com",
    id_rol:    1,
    nombre_rol: "Administrador"
  },
  permisos: {
    calendario: { ver: true, editar: true },
    lista:      { ver: true, editar: true }
    // ... un objeto por módulo
  },
  expira_en: "2026-06-24T20:00:00.000Z"  // TTL 8 horas
}
```

### Flujo completo de autenticación

```
login.html carga
    │
    ├─ ¿Ya hay sesión válida en localStorage?
    │       Sí → redirigir a index.html
    │       No → mostrar formulario
    │
    └─ Usuario envía el formulario
            │
            ├─ Validar campos (client-side)
            ├─ hash = SHA-256(password)
            ├─ POST { action:'login', email, password_hash: hash }
            │
            ├─ GAS responde { ok:true, session_token, usuario, permisos, expira_en }
            │       → guardar en localStorage['<prefix>_session']
            │       → redirigir a index.html
            │
            └─ GAS responde { ok:false, error }  o  error de red
                    → mostrar .alert-err con el mensaje
```

### Verificación de sesión en cada módulo (`initAuth`)

```js
// En window.onload de cada página protegida:
initAuth();  // verifica sesión, refresca permisos, o redirige a login
```

`initAuth()` hace:
1. `checkSetup` → si el sistema no tiene usuarios configurados, redirige a login.
2. Si hay sesión válida → `validateSession` para refrescar permisos del rol (pueden haber cambiado).
3. Si no hay sesión → redirige a login.

### Logout

```js
function logoutUser() {
  const token = SESSION.token;
  SESSION.clear();
  // Notificar al backend (fire and forget, no bloquea)
  if (token && !CFG.isMock()) {
    fetch(CFG.apiUrl, {
      method: 'POST',
      body: JSON.stringify({ action: 'logout', session_token: token })
    }).catch(() => {});
  }
  window.location.href = _loginPath();
}
```

### Hashing de contraseña

```js
async function sha256(str) {
  const buf = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(str));
  return Array.from(new Uint8Array(buf))
    .map(b => b.toString(16).padStart(2, '0')).join('');
}
// Uso: const password_hash = await sha256(password);
// NUNCA enviar la contraseña en texto plano.
```

### Redirección dinámica a login

```js
function _loginPath() {
  const tag = document.querySelector('script[src*="config.js"]');
  if (tag) return tag.getAttribute('src').replace('config.js', '') + 'login.html';
  return 'login.html';
}
```

Nunca hardcodear `'../login.html'` ni `'../../login.html'`. `_loginPath()` detecta la profundidad automáticamente.

---

## 9. Cambio de contraseña

No se implementa en `login.html`. Se accede desde el **sidebar footer** de la app (una vez logueado), mediante un modal renderizado por `auth.js`.

La función canónica es `_showChangePasswordModal()` (CH) / `showChangePasswordModal()` (PCC). Ambas:
- Abren un overlay modal con 3 campos: contraseña actual, nueva, confirmar nueva.
- Mínimo 6 caracteres para la nueva contraseña.
- Hashean ambas contraseñas con `sha256()` antes de enviar.
- Acción backend: `{ action: 'changePassword', password_actual_hash, password_nueva_hash }`.
- Soportan Enter para confirmar y Escape para cancelar.

**No existe** flujo de "olvidé mi contraseña" — el acceso es administrado por el administrador del sistema, que puede resetear contraseñas desde la sección Configuración.

---

## 10. Recordar usuario

**No implementado.** No se ofrece checkbox "Recordar sesión".

El browser gestiona el autocompletado de email y contraseña mediante:
```html
<form autocomplete="on">
  <input type="email" autocomplete="email">
  <input type="password" autocomplete="current-password">
</form>
```

Esto es suficiente para que el browser ofrezca guardar las credenciales. La sesión tiene un TTL fijo de 8 horas en el backend — no es ampliable desde el frontend.

---

## 11. Tema y anti-flash

El login respeta el tema (claro/oscuro) del sistema. El script anti-flash va **antes** de cualquier `<link rel="stylesheet">`, en línea en el `<head>`:

```html
<script>
  (function() {
    var saved = localStorage.getItem('<prefix>_theme');
    var t = saved ? saved : (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', t);
  })();
</script>
```

**Regla**: si no hay tema guardado, usar la preferencia del sistema operativo (`prefers-color-scheme`). Nunca forzar `'light'` como default.

El login **no tiene selector de tema** — la pantalla hereda el tema del sistema o el guardado por el usuario en la app principal.

---

## 12. Responsive

El contenedor `.login-wrap` tiene `max-width: 380px` y `width: 100%`. En móvil:

- El body tiene `padding: 24px` para que la card no toque los bordes.
- Los inputs son `box-sizing: border-box; width: 100%` — nunca desbordan.
- El botón ocupa el ancho completo de la card.

No se requieren media queries adicionales. El layout centrado con flexbox funciona en cualquier ancho.

---

## 13. Versión de la aplicación

El login **no muestra** la versión de la app. La versión se muestra en el sidebar de la app principal (`.brand-meta` con popover de changelog), no en la pantalla de acceso.

---

## 14. Pie institucional

```html
<div class="login-footer">Herramienta interna — acceso restringido</div>
```

Texto canónico: **`Herramienta interna — acceso restringido`**. Igual en todos los proyectos.  
Posición: debajo de la card, centrado.  
Estilo: `font-size: 12px; color: var(--text2, var(--muted)); text-align: center; margin-top: 20px`.

Opcionalmente se puede agregar un enlace para configurar la URL de API (como en PCC):
```html
<a onclick="_configApi()" style="cursor:pointer">Configurar URL de API</a>
```

---

## 15. Plantilla HTML canónica

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ingresar — NOMBRE APP</title>
  <script>
    (function() {
      var saved = localStorage.getItem('PREFIX_theme');
      var t = saved ? saved : (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
      document.documentElement.setAttribute('data-theme', t);
    })();
  </script>
  <link rel="icon" href="data:,">
  <link rel="stylesheet" href="src/css/main.css">
  <style>
    body { display: flex; align-items: center; justify-content: center; min-height: 100vh; padding: 24px; }
    .login-wrap    { width: 100%; max-width: 380px; }
    .login-header  { text-align: center; margin-bottom: 24px; }
    .login-logo-icon {
      width: 48px; height: 48px;
      background: linear-gradient(135deg, var(--primary), var(--primary-hover));
      border-radius: var(--radius-lg);
      display: inline-flex; align-items: center; justify-content: center;
      font-size: 18px; font-weight: 800; color: #fff; letter-spacing: -.5px;
      margin-bottom: 12px;
      box-shadow: 0 2px 8px color-mix(in srgb, var(--primary) 30%, transparent);
    }
    .login-app-name { font-size: 18px; font-weight: 700; color: var(--text); letter-spacing: -.3px; }
    .login-app-sub  { font-size: 13px; color: var(--text2, var(--muted)); margin-top: 3px; }
    .login-card {
      background: var(--card, var(--s1, #fff)); border: 1px solid var(--border, var(--line));
      border-radius: var(--radius-lg); box-shadow: var(--shadow); padding: 28px;
    }
    .field { margin-bottom: 16px; }
    .field label { display: block; font-size: 13px; font-weight: 500; color: var(--text); margin-bottom: 6px; }
    .field input {
      width: 100%; padding: 9px 12px; font-size: 14px; font-family: inherit;
      border: 1px solid var(--border, var(--line)); border-radius: var(--radius, var(--radius-sm));
      outline: none; transition: border-color .15s, box-shadow .15s;
      color: var(--text); background: var(--bg); box-sizing: border-box;
    }
    .field input:focus {
      border-color: var(--primary);
      box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 15%, transparent);
    }
    .btn-login {
      width: 100%; padding: 10px; font-family: inherit; font-size: 14px; font-weight: 600;
      border: none; background: var(--primary); color: #fff;
      border-radius: var(--radius, var(--radius-sm)); cursor: pointer; transition: background .15s; margin-top: 4px;
    }
    .btn-login:hover:not(:disabled) { background: var(--primary-hover); }
    .btn-login:disabled { opacity: .6; cursor: default; }
    .alert-err {
      margin-top: 14px; padding: 10px 14px;
      background: var(--danger-bg, #fef2f2);
      border: 1px solid color-mix(in srgb, var(--danger) 30%, transparent);
      border-radius: var(--radius, var(--radius-sm)); color: var(--danger, #991b1b); font-size: 13px;
    }
    .login-footer { text-align: center; margin-top: 20px; font-size: 12px; color: var(--text2, var(--muted)); }
  </style>
</head>
<body>
  <div class="login-wrap">
    <div class="login-header">
      <div class="login-logo-icon">XX</div>        <!-- sigla 2-3 chars -->
      <div class="login-app-name">Nombre App</div>
      <div class="login-app-sub">Descripción · Luquin</div>
    </div>
    <div class="login-card">
      <form id="loginForm" autocomplete="on">
        <div class="field">
          <label for="login-email">Correo electrónico</label>
          <input type="email" id="login-email" name="email"
                 placeholder="usuario@empresa.com" autocomplete="email" autofocus />
        </div>
        <div class="field">
          <label for="login-pwd">Contraseña</label>
          <input type="password" id="login-pwd" name="password"
                 placeholder="••••••••" autocomplete="current-password" />
        </div>
        <button type="submit" class="btn-login" id="btn-login">Ingresar</button>
      </form>
      <div class="alert-err" id="login-err" hidden></div>
    </div>
    <div class="login-footer">Herramienta interna — acceso restringido</div>
  </div>

  <script src="src/js/config.js"></script>
  <script>
    async function sha256(str) {
      const buf = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(str));
      return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('');
    }

    // Redirigir si ya hay sesión válida
    (function() {
      try {
        const d = JSON.parse(localStorage.getItem('PREFIX_session') || 'null');
        if (d && d.expira_en && new Date(d.expira_en) > new Date()) {
          window.location.href = 'index.html';
        }
      } catch (_) {}
    })();

    const form  = document.getElementById('loginForm');
    const errEl = document.getElementById('login-err');
    const btn   = document.getElementById('btn-login');

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      const email    = document.getElementById('login-email').value.trim().toLowerCase();
      const password = document.getElementById('login-pwd').value;
      errEl.hidden = true;

      if (!email || !password) {
        errEl.textContent = 'Completá los dos campos.';
        errEl.hidden = false;
        return;
      }

      btn.disabled    = true;
      btn.textContent = 'Verificando…';

      try {
        if (!CFG.apiUrl) throw new Error('No hay URL de API configurada.');
        const password_hash = await sha256(password);
        const res = await fetch(CFG.apiUrl, {
          method: 'POST',
          body:   JSON.stringify({ action: 'login', email, password_hash }),
        });
        if (!res.ok) throw new Error('Error de conexión (' + res.status + ').');
        const json = await res.json();
        if (!json.ok) throw new Error(json.error || 'Credenciales incorrectas.');

        localStorage.setItem('PREFIX_session', JSON.stringify({
          session_token: json.session_token,
          usuario:       json.usuario,
          permisos:      json.permisos,
          expira_en:     json.expira_en,
        }));
        window.location.href = 'index.html';
      } catch (err) {
        errEl.textContent = err.message || 'Error al iniciar sesión. Verificá tu conexión.';
        errEl.hidden = false;
      } finally {
        btn.disabled    = false;
        btn.textContent = 'Ingresar';
      }
    });
  </script>
</body>
</html>
```

> Reemplazar `PREFIX` con el prefijo del proyecto (`ch`, `pcc`, `vcc`, `cs`, etc.) y `XX` con la sigla del icono.

---

## 16. Checklist de conformidad

Antes de considerar un login como conforme al estándar, verificar:

**Estructura**
- [ ] Página dedicada `login.html` (no overlay)
- [ ] Contenedor centrado, `max-width: 380px`
- [ ] Header (logo + nombre + subtítulo) separado de la card
- [ ] No se usan Google Fonts ni CDN externo

**Identidad visual**
- [ ] `.login-logo-icon`: 48×48px, `border-radius: var(--radius-lg)`, gradient `--primary → --primary-hover`
- [ ] `.login-app-name`: 18px, `font-weight: 700`
- [ ] `.login-app-sub`: 13px, color muted
- [ ] Sigla del icono: 2-3 chars, mayúsculas, `font-weight: 800`

**Formulario**
- [ ] Dos campos: `type="email"` con `autocomplete="email"` y `type="password"` con `autocomplete="current-password"`
- [ ] `autofocus` en el campo email
- [ ] `<form autocomplete="on">`
- [ ] Botón full-width con texto `Ingresar`
- [ ] IDs canónicos: `login-email`, `login-pwd`, `btn-login`, `login-err`

**Errores**
- [ ] `<div class="alert-err" id="login-err" hidden>` (atributo `hidden`, no `style="display:none"`)
- [ ] Mostrar/ocultar con `errEl.hidden = false/true` (nunca `style.display = ''`)
- [ ] Mensajes según tabla §6
- [ ] Botón deshabilitado con texto `Verificando…` durante el fetch

**Seguridad**
- [ ] Contraseña hasheada con `sha256()` antes de enviar
- [ ] Nunca se envía la contraseña en texto plano
- [ ] Verificación de sesión existente al cargar (`if (isLoggedIn) → index.html`)

**Tema**
- [ ] Script anti-flash inline **antes** del `<link rel="stylesheet">`
- [ ] Fallback a `prefers-color-scheme` si no hay tema guardado
- [ ] `<link rel="icon" href="data:,">` para suprimir 404 de favicon

**Footer**
- [ ] Texto `Herramienta interna — acceso restringido`

**No implementado (no agregar)**
- [ ] No hay checkbox "Recordar sesión"
- [ ] No hay flujo "Olvidé mi contraseña" — el admin resetea desde Configuración
- [ ] No hay selector de tema en el login
- [ ] No se muestra la versión de la app
