# Google Sheets Standards — Base de Datos Maestro

**Versión:** 1.0.0
**Fecha:** 2026-06-17
**Alcance:** Todos los proyectos del ecosistema (Commerce Hub · VTEX Control Center · Marketplace Portal · Customer Service · Herramientas internas)

> **Complementa:** `apps_script_standards.md` (capa de acceso a estos datos) · `style_guide.md` (capa de presentación)

---

## Índice

1. [Principios generales](#1-principios-generales)
2. [Naming — archivos y hojas](#2-naming--archivos-y-hojas)
3. [Naming — columnas](#3-naming--columnas)
4. [Tipos de hojas](#4-tipos-de-hojas)
5. [Columnas obligatorias](#5-columnas-obligatorias)
6. [Tipos de datos y formatos](#6-tipos-de-datos-y-formatos)
7. [Vocabularios controlados](#7-vocabularios-controlados)
8. [Validación de datos](#8-validación-de-datos)
9. [Relaciones entre hojas](#9-relaciones-entre-hojas)
10. [Campos calculados vs almacenados](#10-campos-calculados-vs-almacenados)
11. [Logging y trazabilidad](#11-logging-y-trazabilidad)
12. [Hoja CONFIG](#12-hoja-config)
13. [Patrón multi-store](#13-patrón-multi-store)
14. [Reglas de mantenimiento](#14-reglas-de-mantenimiento)

---

## 1. Principios generales

| Principio | Descripción |
|-----------|-------------|
| **Una fuente de verdad por entidad** | Cada entidad de negocio (producto, pedido, acción comercial) vive en exactamente una hoja. Las demás la referencian, no la duplican. |
| **Estructura primero, datos después** | Los headers son contratos. Agregar columnas solo al final. Nunca reordenar ni renombrar columnas existentes en producción. |
| **Legible para humanos, consumible por código** | Los nombres de columnas deben poder leerse como atributos de un objeto JS. Sin espacios, sin tildes, en snake_case. |
| **Minimal by default** | Agregar columnas cuando hay un caso de uso real, no "por si acaso". |
| **Sin lógica de negocio en Sheets** | Las fórmulas calculan métricas de lectura (`activo`, `% completado`). Las reglas de negocio viven en Apps Script. |

---

## 2. Naming — archivos y hojas

### 2.1 Nombre del archivo Google Sheets

```
[Proyecto] — [Propósito]
```

| Correcto | Incorrecto |
|----------|------------|
| `VTEX Control Center — Base de Datos` | `Base datos vtex nuevo v3` |
| `Commerce Hub — Calendario Comercial` | `HotSale2026_final_FINAL` |
| `Marketplace Portal — Operaciones` | `Copia de portal marketplace` |

Reglas:
- El nombre del proyecto va primero, igual que el nombre en código.
- Un guión largo (`—`) como separador, no un guión corto.
- Sin versiones en el nombre del archivo. Las versiones son de los datos, no del archivo.

### 2.2 Nombre de las hojas (sheets/pestañas)

**Hojas de datos:** `MAYUSCULAS_CON_GUION_BAJO`

```
ACCIONES
PEDIDOS
USUARIOS
TAREAS_HS26
KPI_EVENTOS
```

**Hojas de catálogo:** prefijo `CAT_`

```
CAT_TIPOS
CAT_ESTADOS
CAT_TIENDAS
CAT_AREAS
```

**Hojas de sistema:** nombre descriptivo en mayúsculas

```
CONFIG       → variables y metadatos del sistema
LOGS         → log general de operaciones
ERRORS       → errores con detalle
BACKUPS      → snapshots previos a operaciones bulk
AUDIT_LOG    → registro de cambios por campo
RESUMEN      → dashboard de fórmulas (solo lectura)
```

### 2.3 Reglas de nombres de hojas

- **Sin espacios** — usar `_` como separador.
- **Sin tildes ni caracteres especiales**.
- **Sin prefijos numéricos** (`1_Acciones` → `ACCIONES`).
- El nombre de la hoja coincide con el nombre de la constante en Apps Script:
  ```javascript
  const ACCIONES_SHEET = "ACCIONES";  // mismo nombre que la pestaña
  ```

---

## 3. Naming — columnas

### 3.1 Convención

`snake_case` siempre. Sin tildes, sin mayúsculas, sin espacios.

```
id
titulo
estado
fecha_inicio
creado_por
tienda_sporting    ← boolean por tienda
modificado_por
```

### 3.2 Columnas de ID

| Patrón | Cuándo | Ejemplo |
|--------|--------|---------|
| `id` | Clave primaria autoincremental | `1`, `2`, `3` |
| `id_[entidad]` | FK a otra hoja | `id_evento`, `id_producto` |
| `slug` | Identificador URL-friendly | `hot-sale-2026`, `categoria-zapatillas` |
| `[fuente]_id` | ID de sistema externo | `vtex_id`, `shopify_id` |

### 3.3 Columnas de auditoría

Estas columnas son obligatorias en toda hoja de datos con escritura (ver §5):

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `fecha_creacion` | DateTime | Timestamp de inserción — escrito por Apps Script |
| `fecha_modificacion` | DateTime | Timestamp de última edición — actualizado en cada write |
| `creado_por` | String | Email del usuario que creó el registro |
| `modificado_por` | String | Email del usuario que hizo la última modificación |

### 3.4 Columnas booleanas

Usar `SI / NO` (no `TRUE/FALSE`, no `1/0`, no `sí/no`).

```
activo          → SI / NO
tienda_sporting → SI / NO
tienda_woker    → SI / NO
es_principal    → SI / NO
```

Excepción: campos que GAS necesita comparar como booleanos nativos pueden usar `TRUE/FALSE` internamente, pero la hoja los muestra como `SI/NO` via validación de datos.

### 3.5 Columnas de fecha

| Tipo | Formato en Sheets | Notas |
|------|-------------------|-------|
| Solo fecha | `dd/mm/aaaa` | Formato visual de celda |
| Fecha + hora | Datetime nativo de Sheets | GAS lo serializa como ISO 8601 al servir |
| Timestamp de auditoría | Datetime nativo | Escrito por `new Date()` desde GAS |

---

## 4. Tipos de hojas

### 4.1 Hoja de datos (entidad principal)

Contiene registros de negocio. Es la fuente de verdad para esa entidad.

```
Estructura mínima obligatoria:
col A → id (autoincremental)
col B..N → campos de negocio
col N-3 → fecha_creacion
col N-2 → fecha_modificacion
col N-1 → creado_por
col N   → modificado_por
```

Regla: **las columnas de auditoría siempre van al final**.

### 4.2 Hoja de catálogo (vocabulario controlado)

Listas de valores válidos para campos de tipo `lista`. Sin ID, solo valores.

```
CAT_TIPOS:
Fila 1: CAT_TIPOS    ← header = nombre de la hoja (ayuda a identificar si se abre sola)
Fila 2: Lanzamiento
Fila 3: Campaña Propia
...
```

Las hojas de catálogo alimentan las validaciones de datos de Sheets y los arrays de constantes en GAS.

### 4.3 Hoja CONFIG

Ver §12 para la estructura completa.

### 4.4 Hoja LOGS

Registro de cada operación ejecutada (éxito o error). No se edita manualmente.

```
id | timestamp | accion | entidad | entidad_id | usuario | resultado | detalle | store_id
```

### 4.5 Hoja ERRORS

Registra únicamente errores, con el stack para diagnóstico.

```
id | timestamp | accion | usuario | mensaje | stack | store_id
```

### 4.6 Hoja AUDIT_LOG

Para proyectos con operaciones bulk o datos críticos. Registra cambios campo a campo.

```
id | timestamp | hoja | entidad_id | campo | valor_anterior | valor_nuevo | usuario | store_id
```

### 4.7 Hoja BACKUPS

Snapshots del estado previo a operaciones bulk (UPDATE/DELETE masivos).

```
id_backup | timestamp | accion_origen | total_filas | usuario | store_id | datos_json
```

El campo `datos_json` contiene un array serializado de las filas afectadas antes de la operación. Permite rollback manual.

### 4.8 Hoja RESUMEN

Solo contiene fórmulas. No se escribe desde Apps Script. Sirve para que el equipo vea métricas sin abrir la interfaz.

```
=COUNTA(ACCIONES!A:A)-1        → total de registros
=COUNTIF(ACCIONES!N:N,"SI")    → registros activos
```

---

## 5. Columnas obligatorias

### 5.1 Para hojas con solo lectura (GAS solo lee, nadie escribe desde UI)

| Columna | Obligatoria |
|---------|-------------|
| `id` en col A | ✅ sí |
| Columnas de auditoría | ❌ opcional |

### 5.2 Para hojas con escritura desde GAS o UI

| Columna | Obligatoria |
|---------|-------------|
| `id` en col A | ✅ sí |
| `fecha_creacion` | ✅ sí |
| `fecha_modificacion` | ✅ sí |
| `creado_por` | ✅ sí |
| `modificado_por` | ✅ sí |

### 5.3 Para hojas con estados de ciclo de vida

| Columna | Obligatoria |
|---------|-------------|
| `estado` | ✅ sí (ver §7) |
| `activo` | Recomendado si el estado "activo" se determina por fecha o lógica simple |

---

## 6. Tipos de datos y formatos

| Tipo lógico | Formato en Sheets | Serialización en GAS |
|-------------|-------------------|----------------------|
| Texto corto | Texto, libre | `String` |
| Texto largo | Texto, libre | `String` |
| Número entero | Número sin decimales | `parseInt()` |
| Número decimal | Número con decimales | `parseFloat()` |
| Monto en ARS | Número, formato moneda | `parseFloat()` |
| Boolean | `SI` / `NO` | `=== "SI"` |
| Fecha | `dd/mm/aaaa` | `toISOString().split("T")[0]` |
| Fecha + hora | Datetime nativo | `toISOString()` |
| Lista controlada | Texto, validado por catálogo | `String` (validar contra array) |
| URL | Texto, libre | `String` |
| Email | Texto, libre | `.trim().toLowerCase()` |
| ID externo (VTEX, etc.) | Texto (no número) | `String` — evitar pérdida de ceros |

**Regla crítica:** los IDs de sistemas externos (VTEX, SKUs, códigos de productos) deben almacenarse como **texto**, no como número. Un SKU `00123` en formato número pierde los ceros.

---

## 7. Vocabularios controlados

### 7.1 Estados estándar

Los estados de ciclo de vida de una entidad deben ser consistentes entre proyectos para el mismo concepto.

**Estados de registro (entidades editables):**

```
Borrador       → creado pero no confirmado
Confirmado     → aprobado para producción
Activo         → vigente en este momento
Finalizado     → completado, sin posibilidad de edición
Cancelado      → descartado (soft delete preferido sobre borrado físico)
```

**Estados de tarea u operación:**

```
Por hacer      → pendiente de inicio
En curso       → trabajo activo
Realizado      → completado exitosamente
Bloqueado      → impedimento externo, no puede avanzar
Cancelado      → descartado
```

**Estados de sincronización / proceso:**

```
Pendiente      → en cola
Procesando     → en ejecución
OK             → exitoso
Error          → fallido
Omitido        → saltado intencionalmente
```

### 7.2 Definir vocabularios en hojas CAT_

Todo valor que aparezca en una columna de lista **debe existir en una hoja `CAT_`**. No hardcodear listas en GAS o en HTML sin también tenerlas en el Sheet.

```javascript
// GAS: las constantes replican los valores de la hoja CAT_
const ESTADOS_VALIDOS = ["Borrador", "Confirmado", "Activo", "Finalizado", "Cancelado"];
```

---

## 8. Validación de datos

### 8.1 Configurar validación en Sheets

Para cada columna de lista, configurar en `Datos > Validación de datos`:

| Columna | Tipo | Fuente |
|---------|------|--------|
| `estado` | Lista de rango | `=CAT_ESTADOS!A:A` |
| `tipo` | Lista de rango | `=CAT_TIPOS!A:A` |
| `tienda_*` | Lista de opciones | `SI, NO` |
| `activo` | Lista de opciones | `SI, NO` |

### 8.2 Validación en Apps Script

Antes de escribir, GAS valida que los valores coincidan con los dominios definidos:

```javascript
function validateEstado(estado) {
  const VALIDOS = ["Borrador", "Confirmado", "Activo", "Finalizado", "Cancelado"];
  if (!VALIDOS.includes(estado)) {
    throw new Error(`Estado inválido: "${estado}". Valores permitidos: ${VALIDOS.join(", ")}`);
  }
}
```

Ver patrón completo de validación en `apps_script_standards.md §6`.

---

## 9. Relaciones entre hojas

Google Sheets no tiene foreign keys nativas. El ecosistema usa estas convenciones para modelar relaciones.

### 9.1 Relación muchos a uno (FK simple)

```
ACCIONES.id_evento → EVENTOS.id
```

La columna `id_evento` en ACCIONES almacena el `id` de la fila en EVENTOS. GAS resuelve el join al servir los datos.

```javascript
// En GAS: cargar catálogo y resolver FK al servir
function getAccionById(id) {
  const accion  = findRowById("ACCIONES", id);
  const evento  = accion.id_evento ? findRowById("EVENTOS", accion.id_evento) : null;
  return { ...accion, evento: evento ? evento.nombre : null };
}
```

### 9.2 Relación muchos a muchos

Usar una columna de texto con valores separados por coma (no crear hoja pivot salvo que el volumen lo justifique):

```
TAREAS.areas = "Ecomm, Marketing, TecYDatos"
```

GAS filtra con:
```javascript
rows.filter(r => r.areas.split(",").map(a => a.trim()).includes(areaFiltro));
```

### 9.3 Relación implícita (por convención de nombre)

Para catálogos:

```
ACCIONES.tipo → CAT_TIPOS.valor    (no existe FK, el valor es el mismo string)
```

GAS valida que el valor esté en el catálogo al momento de escritura (§7.2).

### 9.4 Cuándo NO modelar una relación

- Si la relación se usa solo para display y el dato denormalizado es < 100 chars → guardar el valor directo (sin FK).
- Si el catálogo tiene < 10 valores y nunca cambia → array en GAS, sin hoja CAT_.
- Si el join requeriría un segundo request al Sheet → denormalizar en la hoja principal.

---

## 10. Campos calculados vs almacenados

### Regla principal

Un campo **calculado** no debe almacenarse en el Sheet. Se computa en GAS al momento de servir.

| Tipo | Dónde vive | Ejemplos |
|------|-----------|----------|
| Datos originales | Almacenados en Sheet | `obj_facturacion`, `real_facturacion` |
| Cálculos de lectura | Calculados en GAS | `cumplimiento_pct`, `delta_yoy_pct` |
| Estado automático simple | Fórmula en Sheets | `activo = HOY()>=inicio AND HOY()<=fin` |
| Métricas de dashboard | Fórmula en hoja RESUMEN | `=COUNTIF(ACCIONES!N:N,"Activo")` |

### Por qué no almacenar calculados

- Si los datos base se corrigen, los campos calculados guardados quedan inconsistentes.
- Un campo calculado es un derivado — su "verdad" es la fórmula, no el valor en disco.
- Excepciones: si el cálculo tarda > 2s o requiere datos externos, cachear en Sheet con timestamp de expiración.

### Fórmulas permitidas en hojas de datos

Solo fórmulas que no requieren acceso a datos externos y no tienen efecto secundario:

```
Activo:    =SI(Y(HOY()>=fecha_inicio; HOY()<=fecha_fin); "SI"; "NO")
Duración:  =DIAS(fecha_fin; fecha_inicio)
```

Todo lo demás → computar en GAS al servir.

---

## 11. Logging y trazabilidad

### 11.1 Qué registrar siempre

| Evento | Hoja | Cuándo |
|--------|------|--------|
| Cada operación de escritura (create/update/delete) | `LOGS` | Siempre, aunque sea exitosa |
| Errores inesperados de GAS | `ERRORS` | En el bloque `catch` |
| Cambio de valor en campo crítico | `AUDIT_LOG` | En operaciones bulk o datos financieros |
| Snapshot antes de operación bulk | `BACKUPS` | Antes de UPDATE/DELETE masivo |

### 11.2 Estructura de fila en LOGS

```javascript
function writeLog(accion, entidad, entidadId, resultado, detalle, storeId) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("LOGS");
  sheet.appendRow([
    getNextId(sheet),                    // id
    new Date(),                          // timestamp
    accion,                              // "createAccion", "bulkDelete", etc.
    entidad,                             // "ACCIONES", "PEDIDOS"
    entidadId || "",                     // id del registro afectado
    Session.getActiveUser().getEmail(),  // usuario
    resultado,                           // "OK" | "ERROR"
    detalle || "",                       // mensaje descriptivo
    storeId || ""                        // tienda (multi-store)
  ]);
}
```

### 11.3 Separar logs de errores

Los errores tienen su propia hoja (`ERRORS`) con el stack completo, para no contaminar LOGS con datos técnicos:

```javascript
function writeError(accion, mensaje, stack, storeId) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("ERRORS");
  sheet.appendRow([
    getNextId(sheet),
    new Date(),
    accion,
    Session.getActiveUser().getEmail(),
    mensaje,
    stack || "",
    storeId || ""
  ]);
}
```

### 11.4 Backup antes de operación bulk

Antes de cualquier operación que afecte > 10 filas:

```javascript
function backupBeforeBulk(sheetName, rowNumbers, accionOrigen, storeId) {
  const ss      = SpreadsheetApp.getActiveSpreadsheet();
  const sheet   = ss.getSheetByName(sheetName);
  const backup  = ss.getSheetByName("BACKUPS");

  const snapshot = rowNumbers.map(n => sheet.getRange(n, 1, 1, sheet.getLastColumn()).getValues()[0]);

  backup.appendRow([
    getNextId(backup),
    new Date(),
    accionOrigen,
    rowNumbers.length,
    Session.getActiveUser().getEmail(),
    storeId || "",
    JSON.stringify(snapshot)
  ]);
}
```

---

## 12. Hoja CONFIG

Todo proyecto con un Sheet debe tener una hoja `CONFIG`. Es el único lugar donde se almacenan variables del sistema en Sheets (el resto vive en Script Properties).

### 12.1 Estructura

```
Columna A: clave (texto, en MAYUSCULAS)
Columna B: valor
Columna C: descripción (opcional, para humanos)
```

### 12.2 Variables típicas

```
SHEET_VERSION        1.0              Versión del schema del Sheet
LAST_SYNC            (auto)           Timestamp del último sync exitoso con la API
ESTADO_VALUES        Activo,Borrador  Copia de referencia de los estados válidos (para validación en Sheet)
ALLOWED_STORES       sporting,woker   Cuentas habilitadas para este Sheet
WEBHOOK_URL          (ver GAS)        URL del Web App — actualizar tras cada re-deploy
```

### 12.3 Leer CONFIG desde GAS

```javascript
function getConfig_() {
  const sheet  = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("CONFIG");
  const data   = sheet.getDataRange().getValues();
  const config = {};
  data.forEach(row => {
    if (row[0]) config[row[0]] = row[1];
  });
  return config;
}
```

**Regla:** leer CONFIG una sola vez al inicio de una operación, no en cada función. El acceso a Sheets tiene latencia.

---

## 13. Patrón multi-store

Para proyectos con múltiples tiendas o cuentas.

### 13.1 Columna store_id en hojas compartidas

Toda hoja de logs, errores o backups debe tener una columna `store_id`:

```
LOGS.store_id    → "sporting" | "woker" | "fisico"
ERRORS.store_id  → ídem
```

### 13.2 Hojas separadas por store (cuando el volumen lo justifica)

Para hojas de datos con > 5000 filas por cuenta, separar en hojas distintas:

```
PEDIDOS_SPORTING
PEDIDOS_WOKER
```

GAS selecciona la hoja correcta según el `storeId` del request:

```javascript
function getPedidosSheet_(storeId) {
  const name = `PEDIDOS_${storeId.toUpperCase()}`;
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(name);
  if (!sheet) throw new Error(`Hoja no encontrada: ${name}`);
  return sheet;
}
```

### 13.3 Convención de nombres de columnas para stores

Para datasets donde la misma fila tiene datos por tienda (ej: KPIs), usar columnas sufijadas:

```
obj_facturacion_sporting
obj_facturacion_woker
real_facturacion_sporting
real_facturacion_woker
```

Alternativa: filas separadas con columna `tienda` (preferida para tablas de reporting).

---

## 14. Reglas de mantenimiento

### 14.1 Columnas — inmutabilidad de posición

Las posiciones de columna son el contrato entre el Sheet y el código GAS. Cambiarlas sin actualizar el código rompe todo.

**Reglas:**
- Agregar columnas solo al final de la hoja.
- Nunca reordenar columnas existentes.
- Nunca renombrar una columna sin actualizar `*_COLS` en GAS.
- Documentar el mapa columna → número en GAS como constante:
  ```javascript
  const ACCIONES_COLS = {
    id:    1,  // A
    titulo: 2,  // B
    estado: 14, // N
    fecha_creacion: 17 // Q
  };
  ```

### 14.2 Eliminación de registros

**Nunca borrar filas físicamente** en hojas de datos. Usar soft delete:

```javascript
// Soft delete → cambiar estado a "Cancelado"
function deleteRow(sheetName, id) {
  const sheet  = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);
  const rowNum = findRowNumber(sheet, id);
  if (!rowNum) throw new Error("Registro no encontrado");
  sheet.getRange(rowNum, COLS.estado).setValue("Cancelado");
  sheet.getRange(rowNum, COLS.fecha_modificacion).setValue(new Date());
}
```

Excepción: hojas de logs o backups pueden limpiarse por antigüedad (> 90 días), pero con script auditado, no manualmente.

### 14.3 Performance

| Operación | Patrón incorrecto | Patrón correcto |
|-----------|------------------|-----------------|
| Leer todos los datos | `getValue()` por celda en loop | `getDataRange().getValues()` una sola vez |
| Escribir múltiples campos | `setValue()` en cada campo | `setValues()` en un solo `getRange()` |
| Buscar por ID | Iterar fila por fila | Cargar columna A entera con `getRange("A:A").getValues().flat()` |
| Operaciones en batch | Dentro de un trigger de 6 min | Dividir en chunks de 100 filas con `SpreadsheetApp.flush()` intermedio |

### 14.4 Gotchas conocidos

| Gotcha | Mitigación |
|--------|-----------|
| Los rangos son 1-indexed (no 0-indexed) | Siempre `row[0]` para el primer valor, `getRange(1,1)` para celda A1 |
| `getValues()` devuelve array 2D, aunque sea 1 fila | `data[0]` para la primera fila, `data.flat()` para una sola columna |
| Las fechas en GAS son objetos Date, no strings | Serializar con `.toISOString().split("T")[0]` o `.toLocaleDateString()` |
| `getActiveSpreadsheet()` falla si el GAS no está atado al Sheet | Usar `SpreadsheetApp.openById(SHEET_ID)` con el ID desde Script Properties |
| Fórmulas en columnas calculadas se sobreescriben si se hace `setValues()` en un rango que las incluye | Excluir columnas de fórmulas del rango de escritura, o recalcularlas después |
| Escribir una fila completa con `appendRow()` y luego insertar la fórmula en la columna calculada | Hacerlo siempre en dos pasos: `appendRow()` → `setFormula()` en la celda de la última fila |

---

*Documento maestro. Actualizar cuando se establezca un nuevo patrón de datos que aplique a 2+ proyectos. Referencia de implementación: `commerce-hub/docs/commerce_hub_db_structure.md`.*
