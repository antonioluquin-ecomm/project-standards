# Apps Script Standards — Backend Maestro

**Versión:** 1.2.0
**Fecha:** 2026-07-03
**Alcance:** Todos los proyectos del ecosistema (Commerce Hub · VTEX Control Center · Marketplace Portal · Customer Service · Herramientas internas)

> **Complementa:** `google_sheets_standards.md` (modelo de datos) · `ai_rules.md §11` (seguridad y credenciales) · `style_guide.md §10` (frontend que consume este backend)

---

## Índice

1. [Principios generales](#1-principios-generales)
2. [Estructura de archivos .gs](#2-estructura-de-archivos-gs)
3. [Entry points — doGet y doPost](#3-entry-points--doget-y-dopost)
4. [Patrón de router](#4-patrón-de-router)
5. [Formato de respuesta estándar](#5-formato-de-respuesta-estándar)
6. [Validaciones](#6-validaciones)
7. [Auth y seguridad](#7-auth-y-seguridad)
8. [Error handling y logging](#8-error-handling-y-logging)
9. [Helpers de Sheets](#9-helpers-de-sheets)
10. [Naming conventions](#10-naming-conventions)
11. [Límites, quotas y performance](#11-límites-quotas-y-performance)
12. [Deployment](#12-deployment)
13. [Pruebas y diagnóstico](#13-pruebas-y-diagnóstico)

---

## 1. Principios generales

| Principio | Descripción |
|-----------|-------------|
| **Las credenciales viven en Script Properties** | Ninguna credencial en código fuente. Siempre `PropertiesService.getScriptProperties().getProperty("CLAVE")` |
| **Un entry point, un router** | Toda la lógica de dispatch pasa por `doPost()`. `doGet()` es solo para health checks o datos públicos de solo lectura. |
| **Respuesta siempre en JSON** | Toda respuesta usa el formato `{ ok: true, data }` / `{ ok: false, error }`. Sin excepciones. |
| **Validar todo lo que viene del frontend** | El frontend puede estar comprometido. Validar tipo, longitud, dominio de valores y tokens antes de ejecutar cualquier lógica. |
| **Batch siempre** | Nunca leer o escribir Sheets celda por celda dentro de un loop. Leer todo, transformar en memoria, escribir todo. |
| **Logging obligatorio** | Toda operación de escritura genera una fila en la hoja `LOGS`. Los errores van a `ERRORS`. |
| **Sin estado en GAS** | GAS es stateless por ejecución. No confiar en variables globales entre requests. El estado vive en Sheets o Script Properties. |

---

## 2. Estructura de archivos .gs

### 2.1 Archivos obligatorios

```
Code.gs          → entry points (doGet / doPost) y router
Auth.gs          → validación de tokens y roles
Helpers.gs       → funciones de bajo nivel para Sheets (rowToObj, getNextId, findRowById, etc.)
Config.gs        → constantes del sistema (sheet names, col numbers, etc.)
Validators.gs    → validaciones de campos y reglas de negocio
Logger.gs        → writeLog(), writeError()
```

### 2.2 Archivos de dominio (uno por entidad o área)

```
Acciones.gs      → CRUD de la entidad ACCIONES
Pedidos.gs       → lógica de pedidos
Usuarios.gs      → gestión de usuarios internos
Sync.gs          → sincronización con APIs externas (VTEX, etc.)
```

### 2.3 Reglas de organización

- Un archivo por dominio de negocio. No mezclar entidades en el mismo archivo.
- `Code.gs` solo contiene `doGet()`, `doPost()` y el router. Sin lógica de negocio.
- `Helpers.gs` no tiene conocimiento de entidades — solo operaciones genéricas de Sheets.
- `Config.gs` contiene SOLO constantes (sheet names, column numbers, allowed values). Sin funciones.

### 2.4 Ejemplo de Config.gs

```javascript
// Config.gs — solo constantes, sin funciones

const SHEETS = {
  ACCIONES:  "ACCIONES",
  USUARIOS:  "USUARIOS",
  LOGS:      "LOGS",
  ERRORS:    "ERRORS",
  CONFIG:    "CONFIG",
};

const ACCIONES_COLS = {
  id:               1,  // A
  titulo:           2,  // B
  estado:           3,  // C
  tipo:             4,  // D
  activo:           14, // N
  fecha_creacion:   17, // Q
  fecha_modificacion: 18, // R
  creado_por:       19, // S
  modificado_por:   20, // T
};

const ESTADOS_ACCION = ["Borrador", "Confirmado", "Activo", "Finalizado", "Cancelado"];
const TIPOS_ACCION   = ["Lanzamiento", "Campaña Propia", "Fecha Especial", "Liquidación"];
```

---

## 3. Entry points — doGet y doPost

### 3.1 doPost — entry point principal

```javascript
// Code.gs
function doPost(e) {
  try {
    const body   = JSON.parse(e.postData.contents);
    const token  = body.token;
    const accion = body.accion;
    const params = body.params || {};

    const user = validateToken_(token);      // lanza error si inválido
    return routePost_(accion, params, user);

  } catch (err) {
    return jsonResponse_({ ok: false, error: err.message });
  }
}
```

### 3.2 doGet — solo health check o datos públicos

```javascript
// Code.gs
function doGet(e) {
  const accion = e.parameter.accion;

  if (accion === "health") {
    return jsonResponse_({ ok: true, status: "running", timestamp: new Date().toISOString() });
  }

  // Datos de solo lectura que no requieren auth (ej: catálogos públicos)
  if (accion === "getCatalogo") {
    return jsonResponse_({ ok: true, data: getCatalogo_() });
  }

  return jsonResponse_({ ok: false, error: "Acción no permitida por GET" });
}
```

**Regla:** los endpoints que escriben, borran o leen datos privados van por `doPost()` siempre. `doGet()` no recibe token en headers — no usarlo para datos sensibles.

### 3.3 Serialización de respuesta

```javascript
function jsonResponse_(data) {
  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 4. Patrón de router

### 4.1 Estructura del router

```javascript
// Code.gs
function routePost_(accion, params, user) {
  // Solo lectura — cualquier rol
  if (accion === "getAcciones")       return getAcciones_(params, user);
  if (accion === "getAccionById")     return getAccionById_(params.id, user);
  if (accion === "getKpi")            return getKpi_(params, user);

  // Escritura — solo admin
  requireRole_(user, "admin");

  if (accion === "createAccion")      return createAccion_(params, user);
  if (accion === "updateAccion")      return updateAccion_(params.id, params, user);
  if (accion === "deleteAccion")      return deleteAccion_(params.id, user);
  if (accion === "bulkUpdate")        return bulkUpdate_(params, user);

  return jsonResponse_({ ok: false, error: `Acción desconocida: ${accion}` });
}
```

### 4.2 Reglas del router

- El router **solo** hace dispatch. Sin lógica de negocio dentro.
- La validación de rol (`requireRole_`) va en el router, antes del dispatch a funciones de escritura.
- Cada `accion` mapea 1:1 a una función del dominio.
- Acción no reconocida → `{ ok: false, error: "Acción desconocida" }`.

### 4.3 Patrón multi-store

Si el proyecto tiene múltiples tiendas, el `storeId` va en el body y se extrae en el router:

```javascript
function doPost(e) {
  try {
    const body    = JSON.parse(e.postData.contents);
    const token   = body.token;
    const accion  = body.accion;
    const storeId = body.storeId;
    const params  = body.params || {};

    if (!storeId) throw new Error("storeId requerido");
    validateStore_(storeId);                     // verifica que la tienda sea permitida

    const user = validateToken_(token, storeId);
    return routePost_(accion, params, user, storeId);

  } catch (err) {
    return jsonResponse_({ ok: false, error: err.message });
  }
}
```

---

## 5. Formato de respuesta estándar

### 5.1 Respuesta exitosa

```javascript
// Lista de registros
{ ok: true, data: [ {...}, {...} ] }

// Registro único
{ ok: true, data: { id: 1, titulo: "...", ... } }

// Operación de escritura
{ ok: true, data: { id: 42 } }    // devuelve el id del registro creado/modificado

// Operación sin dato de retorno
{ ok: true }
```

### 5.2 Respuesta de error

```javascript
// Error de negocio o validación (error esperado)
{ ok: false, error: "Estado inválido: Pendiente. Valores permitidos: Borrador, Confirmado" }

// Error inesperado (no revelar stack al cliente)
{ ok: false, error: "Error interno del servidor" }
```

**Regla:** el campo `error` es un string legible por humanos. No incluir stack traces, IDs internos ni mensajes de Sheets en respuestas de error de producción.

### 5.3 Wrappers de respuesta

```javascript
function okResponse_(data) {
  return jsonResponse_({ ok: true, ...(data !== undefined ? { data } : {}) });
}

function errorResponse_(mensaje) {
  return jsonResponse_({ ok: false, error: mensaje });
}
```

---

## 6. Validaciones

### 6.1 Regla principal

Todo valor que viene del frontend se trata como potencialmente corrupto. Validar antes de leer o escribir.

### 6.2 Validaciones mínimas por tipo

```javascript
// Requerido
function requireParam_(value, name) {
  if (value === undefined || value === null || value === "")
    throw new Error(`Parámetro requerido: ${name}`);
}

// String con longitud máxima
function validateString_(value, name, maxLen) {
  requireParam_(value, name);
  if (typeof value !== "string") throw new Error(`${name} debe ser texto`);
  if (value.trim().length === 0) throw new Error(`${name} no puede estar vacío`);
  if (maxLen && value.length > maxLen) throw new Error(`${name} supera el máximo de ${maxLen} caracteres`);
  return value.trim();
}

// Entero positivo
function validateId_(value, name) {
  const id = parseInt(value, 10);
  if (isNaN(id) || id <= 0) throw new Error(`${name} debe ser un ID numérico positivo`);
  return id;
}

// Dominio controlado
function validateEnum_(value, name, allowed) {
  if (!allowed.includes(value))
    throw new Error(`${name} inválido: "${value}". Permitidos: ${allowed.join(", ")}`);
  return value;
}

// Booleano de Sheets (SI/NO)
function validateBooleanSI_(value, name) {
  if (value !== "SI" && value !== "NO") throw new Error(`${name} debe ser SI o NO`);
  return value;
}
```

### 6.3 Validar al inicio de cada función de escritura

```javascript
function createAccion_(params, user) {
  // 1. Validar parámetros primero
  const titulo = validateString_(params.titulo, "titulo", 200);
  const tipo   = validateEnum_(params.tipo, "tipo", TIPOS_ACCION);
  const estado = validateEnum_(params.estado, "estado", ESTADOS_ACCION);

  // 2. Luego lógica de negocio
  const ss    = getSpreadsheet_();
  const sheet = ss.getSheetByName(SHEETS.ACCIONES);
  const id    = getNextId_(sheet);

  sheet.appendRow([
    id, titulo, estado, tipo, /* ... */,
    new Date(), new Date(), user.email, user.email
  ]);

  writeLog_("createAccion", "ACCIONES", id, "OK", titulo, user.storeId);
  return okResponse_({ id });
}
```

---

## 7. Auth y seguridad

### 7.1 Modelo de autenticación — RBAC por sesión

El acceso se controla por **sesión de usuario** (no hay token admin en el frontend). Hojas del sistema:

| Hoja | Columnas |
|------|----------|
| `USUARIOS` | id, nombre, email, password_hash, salt, id_rol, activo, fecha_creacion, ultimo_acceso, creado_por |
| `ROLES` | id, nombre, descripcion, activo, es_sistema · seed: 1=Administrador (`es_sistema=SI`, fijo) + roles personalizados |
| `PERMISOS_MODULOS` | id_rol, modulo, puede_ver, puede_editar · 3 estados UI: Oculto / Solo ver / Ver + editar |
| `SESIONES` | session_token, id_usuario, email, id_rol, expira_en, fecha_creacion, activa |

- **Password:** almacenado `SHA256(salt + password_hash)`, con `password_hash = SHA256(plainPassword)` enviado por el frontend. Salt nuevo en cada cambio de contraseña.
- **Sesión:** UUID en hoja `SESIONES` (no Script Properties — sobrevive redeploys y queda auditable), TTL 8h, `activa=SI`. El rol y el estado `activo` se **releen en vivo** de `USUARIOS` en cada validación, así un cambio de rol o una baja aplican sin re-login.

```javascript
// Users.gs — valida el session_token contra la hoja SESIONES
function _validateSessionToken(session_token) {
  if (!session_token) return { ok: false, error: 'Token requerido' };
  const data = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('SESIONES').getDataRange().getValues();
  const h = data[0], tok = h.indexOf('session_token'), exp = h.indexOf('expira_en'), act = h.indexOf('activa');
  for (let i = 1; i < data.length; i++) {
    if (String(data[i][tok]) === session_token) {
      if (data[i][act] !== 'SI')               return { ok: false, error: 'Sesión inactiva' };
      if (new Date(data[i][exp]) < new Date()) return { ok: false, error: 'Sesión expirada' };
      // … releer id_rol y activo actuales de USUARIOS antes de devolver …
      return { ok: true, id_usuario, email, id_rol };
    }
  }
  return { ok: false, error: 'Sesión no encontrada' };
}
```

### 7.2 Autorización por rol y por módulo

El router exige **solo `session_token`** (sin token legacy). Tres niveles: lecturas (cualquier sesión), gestión (solo Administrador `id_rol===1`), y escrituras de dominio (validadas **por módulo** contra `puede_editar`).

```javascript
// Code.gs (router)
if (!session_token) return jsonResp({ ok: false, error: 'Unauthorized', code: 401 });
const ses = _validateSessionToken(session_token);
if (!ses.ok) return jsonResp({ ok: false, error: ses.error, code: 401 });
// Gestión (usuarios/roles/permisos) → solo Administrador
if (ADMIN_SESSION_ACTIONS.indexOf(action) !== -1 && ses.id_rol !== 1)
  return jsonResp({ ok: false, error: 'Requiere Administrador', code: 403 });
// Escrituras de dominio → el rol debe poder editar algún módulo que gobierne la acción
if (isWrite && ses.id_rol !== 1) {
  const mods = ACTION_MODULE_MAP[action] || [];
  const perm = getPermisosForRol(ses.id_rol);
  if (!mods.some(m => perm[m] && perm[m].editar === true))
    return jsonResp({ ok: false, error: 'Forbidden', code: 403 });
}
```

- **Roles:** `1=Administrador` es el único `es_sistema=SI` (acceso total, no editable/desactivable, el sistema nunca queda sin ≥1 admin activo). Los demás son personalizados: se crean/editan/desactivan vía `createRol`/`updateRol`. `updatePermisos` acepta cualquier rol no-sistema y persiste `puede_ver`+`puede_editar`.
- **Permisos por módulo:** `getPermisosForRol(id_rol)` lee `PERMISOS_MODULOS` y devuelve `{ modulo: { ver, editar } }`; el frontend lo consume en `canView`/`canEdit` (3 estados = combinación de los dos booleanos). `ACTION_MODULE_MAP` mapea cada escritura a los módulos que la habilitan.
- **Acciones canónicas:** públicas `login`/`checkSetup`; sesión `validateSession`/`logout`/`getPermisos`/`changePassword`; admin `getUsuarios`/`createUsuario`/`updateUsuario`/`getRoles`/`createRol`/`updateRol`/`updatePermisos`.

### 7.2.1 Gestión de roles — creación, migración e invariantes

Reglas de implementación obligatorias para el modelo flexible (no alcanza con exponer las acciones; el backend debe garantizar estas invariantes):

| Regla | Implementación |
|-------|----------------|
| **Rol nuevo arranca sin acceso** | `createRol` siembra en `PERMISOS_MODULOS` **una fila por cada módulo en estado Oculto** (`puede_ver=NO`, `puede_editar=NO`). El admin habilita módulos después desde la matriz. Nunca dejar un rol sin filas de permisos. |
| **El rol de sistema es intocable** | `updateRol` y `updatePermisos` **bloquean** (`code:403`) si `es_sistema==='SI'`. El Administrador no se renombra, no se desactiva, no se le editan permisos. |
| **Coherencia ver/editar** | `updatePermisos` fuerza `puede_editar=NO` cuando `puede_ver=NO` (no puede existir "edita pero no ve"). Los 3 estados de UI derivan de estos dos booleanos. |
| **Siempre ≥1 Administrador activo** | `updateUsuario` cuenta los usuarios con `id_rol===1 && activo==='SI'`; si la operación dejaría el sistema sin ninguno (cambio de rol o baja del último admin), responde `code:409` y no aplica. |
| **Rol desactivado = sin acceso** | `login` y `_validateSessionToken` deniegan si el rol del usuario tiene `activo='NO'` (el rol de sistema se considera siempre activo). Como el rol se relee en vivo, desactivar un rol expulsa a sus usuarios en la siguiente request. |

**Migración de schema (proyectos con `ROLES` preexistente):** una hoja `ROLES` vieja (sin `activo`/`es_sistema`, o con nombres de rol antiguos) se actualiza con una función **idempotente** `migrateRolesSchema(sheet)` que corre dentro de `setupAll()` **antes** de sembrar:

```javascript
// Setup.gs — idempotente: se puede correr N veces sin efectos colaterales
function migrateRolesSchema(sheet) {
  // 1. Agregar headers activo / es_sistema si faltan (append, no reordena)
  // 2. es_sistema = 'SI' para id===1, 'NO' para el resto
  // 3. activo = 'SI' donde esté vacío
  // 4. Renombrar SOLO nombres viejos conocidos (no pisa nombres ya personalizados)
  //    p.ej. 'Agente Admin'→'Administrador'. La tabla de renombres es por proyecto.
}
```

> Cada proyecto define su propia tabla de renombres según los nombres de rol que tenía. Lo que **no** cambia entre proyectos: `id=1` es siempre el Administrador de sistema, y la migración nunca toca nombres que no estén en la lista de "nombres viejos conocidos".

> Implementación de referencia: `commerce-hub/apps-script/{Code,Users,Setup}.gs`.

### 7.3 Credenciales de APIs externas

```javascript
function getVtexCredentials_(storeId) {
  const props  = PropertiesService.getScriptProperties();
  const prefix = storeId.toUpperCase();

  const account = props.getProperty(`${prefix}_VTEX_ACCOUNT`);
  const appKey  = props.getProperty(`${prefix}_VTEX_APP_KEY`);
  const appToken = props.getProperty(`${prefix}_VTEX_APP_TOKEN`);

  if (!account || !appKey || !appToken)
    throw new Error(`Credenciales no configuradas para tienda: ${storeId}`);

  return { account, appKey, appToken };
}
```

**Reglas de seguridad:**
- Las credenciales **nunca** aparecen en el código fuente ni en respuestas al frontend.
- No logear valores de Script Properties, ni siquiera en `ERRORS`.
- Un error de credencial devuelve `"Credenciales no configuradas"` — no revela el nombre de la propiedad.

### 7.4 Sanitización

Toda cadena que se guarda en Sheets y que podría provenir del usuario debe ser `trim()`-eada:

```javascript
const titulo = params.titulo.trim();
```

No se usan fórmulas de usuario — no hay riesgo de inyección de fórmulas en el patrón `appendRow()`/`setValues()` con arrays. Sin embargo, si por alguna razón se construye un string de fórmula, nunca interpolar input del usuario en él.

---

## 8. Error handling y logging

### 8.1 Try/catch en el entry point

El try/catch de `doPost()` captura cualquier excepción no manejada y devuelve `{ ok: false }`. Esto garantiza que el frontend siempre recibe JSON válido, nunca un HTML de error de GAS.

### 8.2 Errores de negocio (esperados) vs errores técnicos (inesperados)

| Tipo | Qué hacer | Loguear en ERRORS |
|------|-----------|-------------------|
| Validación fallida | `throw new Error("mensaje legible")` | No (es esperado) |
| Registro no encontrado | `throw new Error("No encontrado")` | No |
| Error de Sheets API | Re-throw con mensaje genérico | Sí |
| Error de fetch a API externa | Re-throw con mensaje genérico | Sí |
| Error de quota de GAS | Re-throw con mensaje genérico | Sí |

```javascript
function updateAccion_(id, params, user) {
  try {
    const rowNum = findRowNumber_(SHEETS.ACCIONES, id);
    if (!rowNum) throw new Error(`Registro ${id} no encontrado`);   // error esperado

    // ... lógica de update
    writeLog_("updateAccion", "ACCIONES", id, "OK", "", user.storeId);
    return okResponse_({ id });

  } catch (err) {
    if (err.message.includes("no encontrado")) throw err;           // re-throw esperados

    writeError_("updateAccion", err.message, err.stack, user.storeId);  // loguear inesperados
    throw new Error("Error actualizando registro");                 // mensaje genérico al cliente
  }
}
```

### 8.3 writeLog_ y writeError_ (ver estructura completa en `google_sheets_standards.md §11`)

```javascript
// Logger.gs

function writeLog_(accion, entidad, entidadId, resultado, detalle, storeId) {
  const sheet = getSpreadsheet_().getSheetByName(SHEETS.LOGS);
  sheet.appendRow([
    getNextId_(sheet),
    new Date(),
    accion,
    entidad,
    entidadId || "",
    Session.getActiveUser().getEmail(),
    resultado,
    detalle || "",
    storeId || ""
  ]);
}

function writeError_(accion, mensaje, stack, storeId) {
  const sheet = getSpreadsheet_().getSheetByName(SHEETS.ERRORS);
  sheet.appendRow([
    getNextId_(sheet),
    new Date(),
    accion,
    Session.getActiveUser().getEmail(),
    mensaje,
    stack || "",
    storeId || ""
  ]);
}
```

---

## 9. Helpers de Sheets

Estas funciones van en `Helpers.gs`. Son genéricas — no conocen entidades de negocio.

### 9.1 getSpreadsheet_

```javascript
function getSpreadsheet_() {
  const props = PropertiesService.getScriptProperties();
  const id    = props.getProperty("SPREADSHEET_ID");
  if (!id) throw new Error("SPREADSHEET_ID no configurado en Script Properties");
  return SpreadsheetApp.openById(id);
}
```

**Por qué `openById` y no `getActiveSpreadsheet`:** `getActiveSpreadsheet()` falla cuando el GAS se ejecuta como Web App desatado de un Sheet. `openById()` funciona siempre.

### 9.2 rowToObj_

Convierte una fila plana (array) en un objeto usando el mapa de columnas.

```javascript
function rowToObj_(row, colMap) {
  const obj = {};
  Object.entries(colMap).forEach(([key, colNum]) => {
    const val = row[colNum - 1];  // colNum es 1-indexed
    // Normalizar fechas a ISO string
    obj[key] = (val instanceof Date) ? val.toISOString() : (val === "" ? null : val);
  });
  return obj;
}
```

Uso:
```javascript
const accion = rowToObj_(row, ACCIONES_COLS);
```

### 9.3 getAllRows_

Carga todas las filas de una hoja como array de objetos.

```javascript
function getAllRows_(sheetName, colMap) {
  const sheet  = getSpreadsheet_().getSheetByName(sheetName);
  const data   = sheet.getDataRange().getValues();
  data.shift();  // eliminar header
  return data
    .filter(row => row[0] !== "" && row[0] !== null)  // ignorar filas vacías
    .map(row => rowToObj_(row, colMap));
}
```

### 9.4 findRowById_

Devuelve el número de fila (1-indexed) de un registro por ID.

```javascript
function findRowNumber_(sheetName, id) {
  const sheet = getSpreadsheet_().getSheetByName(sheetName);
  const ids   = sheet.getRange("A:A").getValues().flat();
  const index = ids.indexOf(parseInt(id, 10));
  return index > 0 ? index + 1 : null;  // +1 porque los rangos son 1-indexed
}
```

### 9.5 getNextId_

Autoincremental: max(id) + 1.

```javascript
function getNextId_(sheet) {
  const ids = sheet.getRange("A:A").getValues().flat()
    .map(v => parseInt(v, 10))
    .filter(v => !isNaN(v) && v > 0);
  return ids.length > 0 ? Math.max(...ids) + 1 : 1;
}
```

### 9.6 updateFields_

Actualiza campos específicos de una fila sin sobrescribir el resto.

```javascript
function updateFields_(sheetName, rowNum, colMap, updates) {
  const sheet = getSpreadsheet_().getSheetByName(sheetName);
  Object.entries(updates).forEach(([field, value]) => {
    const colNum = colMap[field];
    if (!colNum) throw new Error(`Campo desconocido: ${field}`);
    sheet.getRange(rowNum, colNum).setValue(value);
  });
  // Actualizar timestamp de modificación
  if (colMap.fecha_modificacion) {
    sheet.getRange(rowNum, colMap.fecha_modificacion).setValue(new Date());
  }
}
```

### 9.7 getConfig_

Lee la hoja CONFIG como objeto.

```javascript
function getConfig_() {
  const sheet  = getSpreadsheet_().getSheetByName(SHEETS.CONFIG);
  const data   = sheet.getDataRange().getValues();
  const config = {};
  data.forEach(row => { if (row[0]) config[row[0]] = row[1]; });
  return config;
}
```

---

## 10. Naming conventions

### 10.1 Funciones

| Tipo | Sufijo | Ejemplo |
|------|--------|---------|
| Función privada (interna al GAS, no llamada desde doGet/doPost) | `_` al final | `validateToken_()`, `getNextId_()` |
| Función pública (entry point o callable desde trigger) | Sin sufijo | `doPost()`, `onEdit()` |
| Handler de dominio (llamado desde router) | Sin sufijo, verbo + nombre | `getAcciones_()`, `createAccion_()` |
| Helper genérico | `_` al final | `rowToObj_()`, `writeLog_()` |

**Por qué el `_` al final:** GAS marca como función pública (visible en Triggers y ejecutable manualmente desde el editor) toda función sin el sufijo. El `_` al final la hace privada y no aparece en la UI de GAS.

### 10.2 Variables

```javascript
const ss       = getSpreadsheet_();    // SpreadsheetApp
const sheet    = ss.getSheetByName();  // hoja específica
const data     = sheet.getDataRange().getValues();  // array 2D crudo
const rows     = data.map(r => rowToObj_(r, COLS)); // array de objetos
const props    = PropertiesService.getScriptProperties();
```

### 10.3 Constantes

```javascript
const SHEETS = { ... }         // MAYUSCULAS — objeto de nombres de hojas
const ACCIONES_COLS = { ... }  // MAYUSCULAS — mapa de columnas
const ESTADOS_VALIDOS = [...]  // MAYUSCULAS — arrays de dominio
```

### 10.4 Script Properties

Prefijo por store para multi-store:

```
SPORTING_VTEX_ACCOUNT
SPORTING_VTEX_APP_KEY
SPORTING_VTEX_APP_TOKEN
WOKER_VTEX_ACCOUNT
...
SPREADSHEET_ID           → ID del Sheet principal (sin prefijo de store)
SESSION_{uuid}           → tokens de sesión activos
```

---

## 11. Límites, quotas y performance

### 11.1 Límites críticos de GAS

| Límite | Valor | Estrategia |
|--------|-------|-----------|
| Tiempo máximo de ejecución | 6 minutos | Dividir operaciones bulk en chunks |
| Tiempo máximo de Web App | 30 segundos (respuesta HTTP) | Para procesos largos, responder 200 inmediatamente y continuar async |
| `UrlFetchApp` por día | 20.000 requests | Cachear respuestas de APIs externas cuando sea posible |
| `PropertiesService` — tamaño por propiedad | 9KB | No guardar listas grandes en Properties (usar Sheets) |
| `SpreadsheetApp` por día | Sin límite duro, pero I/O lento | Minimizar lecturas con batch |

### 11.2 Patrón batch para operaciones sobre muchas filas

```javascript
function bulkUpdateEstado_(params, user) {
  const { ids, nuevoEstado } = params;
  validateEnum_(nuevoEstado, "estado", ESTADOS_ACCION);

  const sheet  = getSpreadsheet_().getSheetByName(SHEETS.ACCIONES);
  const data   = sheet.getDataRange().getValues();  // leer TODO una sola vez

  const CHUNK  = 100;
  let updated  = 0;

  for (let i = 0; i < ids.length; i += CHUNK) {
    const chunk = ids.slice(i, i + CHUNK);
    chunk.forEach(id => {
      const rowIdx = data.findIndex(r => r[0] === parseInt(id, 10));
      if (rowIdx < 0) return;
      // Modificar en memoria
      data[rowIdx][ACCIONES_COLS.estado - 1]             = nuevoEstado;
      data[rowIdx][ACCIONES_COLS.fecha_modificacion - 1] = new Date();
      data[rowIdx][ACCIONES_COLS.modificado_por - 1]     = user.email;
      updated++;
    });
    SpreadsheetApp.flush();  // forzar escritura del chunk
  }

  // Escribir TODO de vuelta en un solo setValues
  sheet.getRange(1, 1, data.length, data[0].length).setValues(data);

  writeLog_("bulkUpdateEstado", "ACCIONES", ids.join(","), "OK", `${updated} filas`, user.storeId);
  return okResponse_({ updated });
}
```

### 11.3 Cache de datos frecuentes

Para catálogos o config que se leen en cada request:

```javascript
let _configCache = null;

function getConfigCached_() {
  if (_configCache) return _configCache;  // devuelve si ya está en memoria de esta ejecución
  _configCache = getConfig_();
  return _configCache;
}
```

Nota: el cache vive solo durante la ejecución del request. GAS no tiene estado entre requests.

### 11.4 No anidar reintentos sobre un helper que ya reintenta

La cuota de `UrlFetchApp` (ver §11.1) es **por proyecto de Apps Script, compartida por todos los módulos** que cuelgan del mismo Web App — no por módulo ni por usuario. Un módulo con retry mal diseñado puede agotarla para todos los demás.

**Anti-patrón:** un módulo agrega su propio wrapper con backoff (`miRetry_(path) { ... vtexFetch_(path) ... }`) por encima de un helper compartido (`vtexFetch_`/`vtexRequest_`) que **también** reintenta internamente. El resultado es reintentar el reintento: si el helper base ya hace hasta 3 intentos con backoff, y el wrapper local hace 2 intentos más por encima, una sola llamada puede terminar disparando hasta 6 requests reales con una espera acumulada de más de un minuto. Con timeouts así de largos, el proxy del Web App de Apps Script puede devolver una respuesta no-JSON con status inesperado (404 u otro) en vez de esperar — un síntoma que parece un bug de red pero es, en realidad, un problema de reintentos anidados.

```javascript
// ❌ MAL — vtexFetch_ ya reintenta 429 internamente; este wrapper duplica la espera
function miRetryConBackoff_(path, method, payload, storeId) {
  for (let attempt = 0; attempt < 2; attempt++) {
    try { return vtexFetch_(storeId, path, { method, body: payload }); }
    catch (e) {
      if (!String(e.message).includes('429') || attempt >= 1) throw e;
      Utilities.sleep(1500 * Math.pow(2, attempt));
    }
  }
}

// ✅ BIEN — el helper base ya maneja 429; llamarlo directo
vtexFetch_(storeId, path, { method, body: payload });
```

**Regla:** si un helper compartido (`vtexFetch_`, `vtexRequest_`, etc.) ya reintenta ante 429/5xx, ningún módulo debe agregar una capa de retry propia por encima. Si un módulo necesita un comportamiento de retry distinto (más intentos, backoff distinto), se ajusta el helper compartido con un parámetro opcional — no se duplica la lógica localmente.

### 11.5 Cache de escaneos pesados (chunked)

Los escaneos que paginan un catálogo/cola completa (ej. todos los productos, toda la cola de sugerencias pendientes) son el principal consumidor de la cuota compartida de `UrlFetchApp`. Cachear el resultado evita repetir el costo completo cuando:
- La pantalla se recarga o el usuario cambia de filtro sin que los datos hayan cambiado (TTL corto, 30-60s).
- El usuario reintenta un botón de "Escanear" varias veces mientras revisa la tabla (TTL medio, 5-10 min).
- Una operación de escritura necesita datos que ya se trajeron al cargar una pantalla relacionada — reusar en vez de re-escanear (ver ejemplo de aprobación en `vtex-control-center/docs/project_workflow.md §13.12`).

`CacheService` limita cada entrada a 100KB, así que un resultado de escaneo (que suele superar ese límite) necesita partirse en chunks y reensamblarse:

```javascript
// Utils.gs — genérico, sin conocimiento de dominio
var CACHE_CHUNK_SIZE_DEFAULT = 90000; // margen bajo el límite real de 100KB

function cachePutChunked_(key, value, ttlSec, chunkSize) {
  try {
    var size = chunkSize || CACHE_CHUNK_SIZE_DEFAULT;
    var json = JSON.stringify(value);
    var chunks = [];
    for (var i = 0; i < json.length; i += size) chunks.push(json.slice(i, i + size));
    var cache = CacheService.getScriptCache();
    var map = {};
    map[key + '_meta'] = String(chunks.length);
    chunks.forEach(function(c, idx) { map[key + '_' + idx] = c; });
    cache.putAll(map, ttlSec);
  } catch (e) { /* cache best-effort: si falla, simplemente no cachea */ }
}

function cacheGetChunked_(key) {
  try {
    var cache = CacheService.getScriptCache();
    var count = parseInt(cache.get(key + '_meta') || '', 10);
    if (!count) return null;
    var keys = [];
    for (var i = 0; i < count; i++) keys.push(key + '_' + i);
    var got = cache.getAll(keys);
    var json = '';
    for (var j = 0; j < count; j++) {
      var part = got[key + '_' + j];
      if (part === undefined || part === null) return null; // chunk vencido/incompleto
      json += part;
    }
    return JSON.parse(json);
  } catch (e) { return null; }
}

function cacheInvalidateChunked_(key) {
  try { CacheService.getScriptCache().remove(key + '_meta'); } catch (e) {}
}
```

Uso típico dentro de una función de escaneo:

```javascript
function scanCatalog_(storeId) {
  var cacheKey = 'scan_' + storeId;
  var cached = cacheGetChunked_(cacheKey);
  if (cached) return cached;

  var result = /* ... escaneo pesado real ... */;

  cachePutChunked_(cacheKey, result, 300); // 5 min
  return result;
}

// Invalidar tras una escritura real que cambia lo cacheado
function executeUpdate_(items, storeId) {
  /* ... */
  cacheInvalidateChunked_('scan_' + storeId);
}
```

**Reglas:**
- `ttlSec` y `chunkSize` son por-llamada — cada módulo elige el TTL según su patrón de uso (§11.5, primer párrafo), no hay un TTL global correcto para todos los casos.
- Definir estas 3 funciones **una sola vez** en `Utils.gs` (o el helper compartido del proyecto) y reutilizarlas — no duplicar una copia local por módulo con un nombre distinto (ej. `scoreCachePutChunked_`, `simColorCachePutChunked_`). GAS mergea todos los `.gs` en un solo namespace global; copias locales "casi idénticas" no rompen nada mientras sean idénticas, pero divergen en silencio en el primer fix aplicado a una sin tocar las demás.
- El cache es best-effort: si `CacheService` falla (payload enorme, cuota de cache agotada), la función debe seguir funcionando sin cache, no lanzar una excepción.

### 11.6 fetch a APIs externas

```javascript
function vtexFetch_(storeId, path, options) {
  const creds = getVtexCredentials_(storeId);
  const url   = `https://${creds.account}.vtexcommercestable.com.br${path}`;

  const response = UrlFetchApp.fetch(url, {
    method:            options.method || "GET",
    headers: {
      "X-VTEX-API-AppKey":   creds.appKey,
      "X-VTEX-API-AppToken": creds.appToken,
      "Content-Type":        "application/json",
      "Accept":              "application/json",
    },
    payload:           options.body ? JSON.stringify(options.body) : undefined,
    muteHttpExceptions: true,
  });

  const code = response.getResponseCode();
  const body = response.getContentText();

  if (code < 200 || code >= 300) {
    throw new Error(`API error ${code}: ${body.substring(0, 200)}`);
  }

  return JSON.parse(body);
}
```

**`muteHttpExceptions: true` es obligatorio** — sin esto, GAS lanza una excepción ante cualquier respuesta 4xx/5xx, perdiendo el body del error. Con `muteHttpExceptions: true` se recibe el body y se puede loguear el detalle.

---

## 12. Deployment

### 12.1 Configurar el Web App

1. `Implementar > Nueva implementación > Web App`
2. **Ejecutar como:** `Yo (email del propietario)` — no como "usuario que accede"
3. **Quién puede acceder:** `Cualquier usuario` — para que el frontend sin auth de Google pueda llamarlo

**Importante:** el sistema maneja su propia autenticación por token UUID. No delegar auth a Google (Ejecutar como "usuario que accede" rompe el acceso desde el frontend).

### 12.2 Versiones de deployment

Cada cambio funcional en GAS requiere **nueva implementación** (no reusar la anterior):

```
Implementar > Administrar implementaciones > ✏️ editar > Nueva versión
```

La URL del Web App **no cambia** entre versiones, pero el código sí se actualiza.

### 12.3 Actualizar WEBHOOK_URL en CONFIG del Sheet

Después de un re-deployment, si la URL cambió (primera vez que se publica, o si se destruyó la implementación), actualizar:

1. `CONFIG` en el Sheet → clave `WEBHOOK_URL` → nuevo valor
2. Frontend → `config.js` → `APPS_SCRIPT_URL` → mismo nuevo valor

### 12.4 Script Properties — configuración inicial

Al deployar en un proyecto nuevo, configurar manualmente en GAS (`Proyecto > Script Properties`):

```
SPREADSHEET_ID          → ID de la Google Sheet (visible en la URL del Sheet)
[STORE]_VTEX_ACCOUNT    → nombre de cuenta VTEX (ej: "sportingarg")
[STORE]_VTEX_APP_KEY    → App Key de VTEX
[STORE]_VTEX_APP_TOKEN  → App Token de VTEX
```

**Nunca commitear** un archivo de configuración con estos valores. Script Properties es el almacén seguro.

---

## 13. Pruebas y diagnóstico

### 13.1 Pruebas manuales desde el editor

GAS no tiene framework de test. La estrategia es funciones de test corridas manualmente:

```javascript
// En Code.gs — solo para testing, eliminar antes de production o marcar con comentario
function _testCreateAccion() {
  const result = createAccion_({
    titulo: "Test desde editor",
    tipo:   "Lanzamiento",
    estado: "Borrador"
  }, { email: "test@test.com", role: "admin", storeId: "sporting" });

  Logger.log(JSON.stringify(result));
}
```

Ejecutar desde la UI de GAS: seleccionar `_testCreateAccion` en el dropdown y click "Ejecutar".

### 13.2 Ver logs de ejecución

`Ver > Registros` (Ctrl+Enter) en el editor de GAS muestra los `Logger.log()` de la última ejecución.

Para logs históricos del Web App: abrir la hoja `LOGS` o `ERRORS` directamente.

### 13.3 Simular un request de doPost

```javascript
function _testDoPost() {
  const fakeEvent = {
    postData: {
      contents: JSON.stringify({
        token:   "test-token-aqui",
        accion:  "getAcciones",
        storeId: "sporting",
        params:  {}
      })
    }
  };

  const result = doPost(fakeEvent);
  Logger.log(result.getContent());
}
```

### 13.4 Diagnóstico de performance

Si una operación supera los 30s, revisar en orden:

1. ¿Cuántos accesos a `getSpreadsheet_()` / `getSheetByName()` hay en el flujo? → debería ser máximo 1 por request.
2. ¿Hay `getValue()` o `setValue()` dentro de loops? → reemplazar con `getValues()` / `setValues()` fuera del loop.
3. ¿Hay llamadas a `UrlFetchApp` dentro de un loop de filas? → agrupar en batch o paginar.
4. ¿El Sheet tiene > 50.000 filas en la hoja? → archivar filas viejas a una hoja `HISTORICO_20XX`.

---

*Documento maestro. Actualizar cuando se establezca un nuevo patrón de GAS que aplique a 2+ proyectos. Referencias de implementación proyecto-específica: `commerce-hub/docs/commerce_hub_db_structure.md` · `vtex-control-center/docs/project_workflow.md §12`.*
