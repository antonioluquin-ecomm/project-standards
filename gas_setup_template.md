# GAS Setup Template — Google Apps Script como Backend

**Versión:** 1.0  
**Alcance:** Proyectos con stack HTML/CSS/Vanilla JS + Google Apps Script + Google Sheets

---

## Contexto

En este ecosistema, Google Apps Script actúa como middleware entre el frontend estático (GitHub Pages) y los datos (Google Sheets u APIs externas). El frontend nunca tiene credenciales — solo conoce la URL del Web App.

```
Frontend (GitHub Pages) → GAS Web App → Google Sheets / API externa
```

Este documento cubre los pasos manuales en Google (crear el proyecto GAS, configurar credenciales, publicar el Web App). El código `.gs` ya vive en el repositorio bajo `apps-script/`.

---

## Paso 1 — Crear el proyecto GAS y copiar los archivos

### Opción A — Desde Google Sheets (proyectos con Sheets como base de datos)

1. Abrir el Google Sheet del proyecto
2. `Extensiones > Apps Script`
3. El proyecto GAS queda vinculado al Sheet automáticamente

### Opción B — Proyecto GAS independiente (proyectos con APIs externas)

1. Ir a [script.google.com](https://script.google.com)
2. Crear nuevo proyecto → nombrarlo con el nombre del proyecto
3. El GAS no está vinculado a ningún Sheet por defecto

### Copiar los archivos `.gs`

Por cada archivo `.gs` en `apps-script/` del repo:

1. En el editor GAS, click en `+` (archivo nuevo) → nombrar igual al archivo del repo (sin `.gs`)
2. Pegar el contenido del archivo del repo
3. Guardar (`Ctrl+S`)
4. Eliminar el `Code.gs` de ejemplo si quedó vacío o duplicado

> **Nota:** En GAS, todos los archivos `.gs` del mismo proyecto comparten el mismo scope global. Las funciones de `Utils.gs` están disponibles en `Categorias.gs` automáticamente, sin imports.

**Lista de archivos a copiar:** ver `CLAUDE.md` del proyecto o la sección §7 de `docs/project_workflow.md`.

---

## Paso 2 — Configurar Script Properties

En el editor GAS: **Configuración del proyecto (⚙) > Propiedades del script > Agregar propiedad**

Las credenciales **nunca** van en el código ni en el repositorio. Solo en Script Properties.

### Propiedades comunes (todos los proyectos con Sheets)

| Propiedad | Descripción | Ejemplo |
|-----------|-------------|---------|
| `SPREADSHEET_ID` | ID del Google Sheet principal | `1BxiMV...` (de la URL del Sheet) |

### Propiedades para proyectos multi-store (VTEX)

Cada tienda usa un prefijo en UPPER_SNAKE_CASE. El prefijo debe coincidir con el `id` de `config.js` en mayúsculas.

| Propiedad | Descripción |
|-----------|-------------|
| `{TIENDA}_VTEX_ACCOUNT` | Nombre de la cuenta VTEX |
| `{TIENDA}_VTEX_APP_KEY` | App Key de la API de VTEX |
| `{TIENDA}_VTEX_APP_TOKEN` | App Token de la API de VTEX |
| `{TIENDA}_VTEX_ENVIRONMENT` | Entorno VTEX (opcional) |
| `LOG_SPREADSHEET_ID` | ID del Sheet de logs (compartido por todas las tiendas) |

**Agregar una tienda nueva:** solo añadir las propiedades con el nuevo prefijo. No se modifica ningún archivo GAS.

### Propiedades para proyectos con auth de usuarios

| Propiedad | Descripción |
|-----------|-------------|
| `SPREADSHEET_ID` | ID del Sheet con las tablas de usuarios y sesiones |

> Ver `../project-standards/login_standard.md` para el patrón de autenticación completo.

### Cómo obtener el ID de un Google Sheet

De la URL:
```
https://docs.google.com/spreadsheets/d/  ESTE_ES_EL_ID  /edit
```

---

## Paso 3 — Google Sheet de datos o logs

### Si el Sheet es la base de datos principal

1. Crear un Google Sheet nuevo con un nombre descriptivo (ej: `Nombre del Proyecto — Base de Datos`)
2. Copiar su ID y cargarlo en `SPREADSHEET_ID`
3. Si el GAS tiene una función de bootstrap (ej: `setupAll`), ejecutarla para crear hojas y headers automáticamente

### Si el Sheet es solo para logs

1. Crear un Google Sheet vacío (ej: `Nombre del Proyecto — Logs`)
2. Copiar su ID y cargarlo en `LOG_SPREADSHEET_ID`
3. El GAS crea las hojas de log automáticamente al ejecutarse por primera vez

### Hojas creadas automáticamente (patrón estándar)

| Hoja | Contenido |
|------|-----------|
| `ejecuciones` | Resumen de cada corrida (total, éxitos, errores, duración) |
| `errores` | Errores técnicos con stack trace |
| `logs_{entidad}` | Cambios campo a campo por entidad |
| `backups_{entidad}` | Snapshot antes de cada modificación |

> Todas incluyen columna `tienda` en proyectos multi-store para diferenciar operaciones por cuenta.

---

## Paso 4 — Publicar como Web App

1. En el editor GAS: **Implementar > Nueva implementación**
2. Tipo: **Aplicación web**
3. **Ejecutar como:** Yo (la cuenta dueña del proyecto GAS)
4. **Quién tiene acceso:** Cualquier persona (necesario para que el frontend estático pueda llamarlo sin autenticación de browser)
5. Click **Implementar** → autorizar los permisos que pida
6. Copiar la **URL del Web App** (`https://script.google.com/macros/s/.../exec`)

> La URL no cambia al actualizar versiones. No es necesario modificar el frontend tras cada redeploy.

---

## Paso 5 — Configurar el frontend

La URL del Web App se carga en el frontend a través de `config.js` o `localStorage`, según el proyecto.

**Patrón con `config.js` (proyectos con URL hardcodeada):**
```js
const CONFIG = {
  APPS_SCRIPT_URL: 'https://script.google.com/macros/s/TU_ID/exec',
  // ...
};
```

**Patrón con `localStorage` (proyectos en modo demo / configuración por usuario):**
```
localStorage.setItem('nombre_proyecto_api_url', 'https://script.google.com/macros/s/TU_ID/exec');
```

Sin URL configurada → el frontend corre en **modo demo** con datos locales (si el proyecto lo soporta).

> Los `id` de tienda en `config.js` deben coincidir exactamente (en minúsculas) con el prefijo de las Script Properties (en mayúsculas). Ej: `id: 'sporting'` → propiedades `SPORTING_VTEX_APP_KEY`, etc.

---

## Paso 6 — Verificación

### Health check básico

Abrir en el navegador:
```
URL_DEL_WEBAPP?accion=health
```
Respuesta esperada:
```json
{ "ok": true, "status": "running" }
```

### Verificación desde el editor GAS

Muchos proyectos incluyen funciones de test que se ejecutan directamente en el editor:

```js
testConfig('sporting')   // verifica credenciales de una tienda
_testDoPost()            // prueba el router doPost
```

Seleccionar la función en el dropdown → **Ejecutar** → ver los logs en la consola inferior.

### Verificación desde el frontend

1. Abrir la herramienta en el browser
2. Ejecutar la primera acción que llame al GAS
3. Revisar la consola del browser y la pestaña Network para confirmar que la respuesta es `{"ok": true, ...}`

---

## Actualizar el GAS después de cambios en `.gs`

Cada vez que se modifica un archivo `.gs` hay que re-deployar para que los cambios tomen efecto en la URL productiva:

1. En el editor GAS: **Implementar > Administrar implementaciones**
2. Click en el lápiz (editar) de la implementación activa
3. Cambiar "Versión" a **Nueva versión**
4. **Guardar**

> La URL del Web App no cambia. No es necesario modificar `config.js` ni el frontend.

---

## Límites conocidos del GAS

| Límite | Valor | Mitigación |
|--------|-------|-----------|
| Tiempo máximo de ejecución | 6 minutos | Dividir operaciones masivas en batches |
| Requests `fetchAll` paralelas | Sin límite de GAS; API externa puede throttlear | Usar batches de 50–200 según la API |
| Tamaño máximo de respuesta | ~50 MB | Paginar resultados grandes |
| Cuota diaria de requests a Sheets | 100.000 operaciones de lectura | Cachear resultados cuando sea posible |
| Cuota de tiempo de ejecución | 6 h/día para cuentas gratuitas | Suficiente para uso interno |

---

## Diagnóstico de errores comunes

| Síntoma | Causa probable | Solución |
|---------|---------------|---------|
| `{"ok":false,"error":"Script property not found"}` | Falta una Script Property | Verificar nombres exactos en la configuración |
| Frontend recibe respuesta opaca (`no-cors`) | Formulario usa `fetch` con `mode: 'no-cors'` | Verificar en el Sheet directamente; ver logs del GAS |
| GAS devuelve error 401 / 403 | App Key o Token incorrecto, o permisos insuficientes | Verificar credenciales y permisos de la API |
| Cambio de código no se refleja | No se hizo redeploy con nueva versión | Ver sección "Actualizar el GAS" |
| Error de timeout en operaciones masivas | Ejecución supera 6 minutos | Reducir el tamaño del batch |
| Script Properties con comillas literales | La UI de GAS guarda `"188"` en vez de `188` | Guardar el valor sin comillas; el código puede defenderse con `.replace(/"/g, '').trim()` |
