# AI Rules — Estándar Maestro de Colaboración con IA

**Versión:** 1.0.0  
**Fecha:** 2026-06-16  
**Autor:** Gabriel Luna — Productos Web  
**Alcance:** Claude Code · Codex · ChatGPT · Futuras herramientas de IA  
**Proyectos:** Marketplace Portal · Commerce Hub · VTEX Control Center · VTEX Bookmarklets · Project Control Center · Customer Service Control Center · Correos Transaccionales · eComm Hub · Herramientas internas  
**Documentación compartida:** `../project-standards/` (relativo a cada proyecto)

---

## Índice

1. [Principios generales](#1-principios-generales)
2. [Filosofía de desarrollo](#2-filosofía-de-desarrollo)
3. [Reglas de análisis previo](#3-reglas-de-análisis-previo)
4. [Reglas para Claude Code](#4-reglas-para-claude-code)
5. [Reglas para Codex](#5-reglas-para-codex)
6. [Reglas para ChatGPT](#6-reglas-para-chatgpt)
7. [Control de cambios](#7-control-de-cambios)
8. [Refactors](#8-refactors)
9. [Documentación obligatoria](#9-documentación-obligatoria)
10. [Git](#10-git)
11. [Seguridad](#11-seguridad)
12. [Gestión de contexto](#12-gestión-de-contexto)
13. [Aprendizajes](#13-aprendizajes)
14. [Handoff entre IAs](#14-handoff-entre-ias)
15. [Checklist obligatorio antes de finalizar](#15-checklist-obligatorio-antes-de-finalizar)

---

## 1. Principios generales

Estas reglas aplican a **cualquier IA** que colabore en estos proyectos, independientemente de la herramienta, el modelo o el contexto de la conversación.

### 1.1 Jerarquía de prioridades

Toda IA debe respetar el siguiente orden cuando dos principios entren en conflicto:

```
1. Estabilidad del sistema en producción
2. Funcionalidad solicitada por el usuario
3. Calidad y mantenibilidad del código
4. Eficiencia y performance
5. Elegancia técnica
```

La estabilidad siempre gana. Un sistema que funciona imperfectamente es mejor que uno que no funciona.

### 1.2 Reglas no negociables

| # | Regla | Descripción |
|---|-------|-------------|
| R1 | **Cambios incrementales** | Todo cambio debe ser lo más pequeño posible y verificable de forma aislada. |
| R2 | **Sin reescrituras no autorizadas** | Nunca reescribir un archivo completo cuando el problema se puede resolver editando 5 líneas. |
| R3 | **Compatibilidad hacia atrás** | Un cambio nuevo no puede romper funcionalidad existente salvo que el usuario lo autorice explícitamente. |
| R4 | **Alcance cerrado** | Resolver únicamente lo que se pidió. No "mejorar de paso" código que no fue mencionado. |
| R5 | **Confirmación antes de destruir** | Antes de eliminar archivos, borrar datos, hacer push o ejecutar acciones irreversibles, pedir confirmación. |
| R6 | **Documentar decisiones no obvias** | Si se toma una decisión técnica que no es evidente, dejarla registrada en el código o en el documento correspondiente. |
| R7 | **Respetar el style guide** | Todo código generado debe seguir `style_guide.md`. Si no existe en el proyecto, seguir los patrones visuales y técnicos ya establecidos en los archivos existentes. |
| R8 | **Versionado obligatorio** | Todo cambio funcional debe reflejarse en `config.js` o el archivo de versión del proyecto antes del commit. |

### 1.3 Lo que una IA nunca debe hacer por iniciativa propia

- Cambiar el stack tecnológico (agregar React, Tailwind, npm, etc.)
- Modificar el sistema de autenticación o sesiones
- Cambiar la paleta de colores o la tipografía del proyecto
- Reordenar o renombrar carpetas y archivos sin autorización
- Publicar, subir o compartir contenido del proyecto en servicios externos
- Interpretar silencio como aprobación: si algo no está claro, preguntar

### 1.4 Entorno de desarrollo

| Dato | Valor |
|------|-------|
| **Carpeta raíz de todos los proyectos** | `C:\Users\gluna\Documents\Repos` |
| **Documentación estándar compartida** | `../project-standards/` (relativo a cada proyecto) |
| **Versionado y colaboración** | GitHub (repositorio por proyecto) |
| **Hosting de frontend** | GitHub Pages (rama `main`) |

- El desarrollo se realiza **exclusivamente** desde `C:\Users\gluna\Documents\Repos`. No usar OneDrive ni SharePoint como carpeta de trabajo.
- La documentación estándar compartida vive en `../project-standards/` relativo a cada proyecto. No copiarlos al proyecto — referenciarlos desde `CLAUDE.md` y `AGENTS.md`.
- OneDrive y SharePoint quedan reservados para documentación funcional: PDFs, presentaciones, actas, imágenes y archivos compartidos con terceros.

---

## 2. Filosofía de desarrollo

### 2.1 Resolver el problema, no el problema imaginario

Cuando el usuario pide "arreglar el filtro de búsqueda", la IA debe:
- Arreglar el filtro de búsqueda ✓
- No aprovechar para refactorizar el módulo completo ✗
- No cambiar el diseño visual de la pantalla ✗
- No agregar funcionalidades "que podrían ser útiles" ✗

### 2.2 Proponer antes de implementar (cuando hay incertidumbre)

Si el cambio solicitado tiene múltiples formas de implementarse y la elección afecta la arquitectura, la IA debe **proponer el enfoque y esperar aprobación** antes de escribir código. Una respuesta de 2–3 líneas con una recomendación clara es suficiente.

Esto aplica cuando:
- El cambio afecta más de 3 archivos
- La solución requiere agregar una dependencia externa
- Hay que decidir entre un cambio localizado y un cambio estructural
- El comportamiento solicitado podría entrar en conflicto con funcionalidad existente

### 2.3 Simplicidad sobre inteligencia

Un código simple que cualquier desarrollador entiende en 5 minutos es mejor que código "elegante" que requiere conocer 3 patrones avanzados. Los proyectos internos los mantiene un equipo pequeño a lo largo del tiempo; la complejidad tiene costo real.

```
Preferir:
  → función directa de 10 líneas
  → sobre clase abstracta + interfaz + factory de 80 líneas

Preferir:
  → CSS con clases explícitas
  → sobre sistema de tokens + mixins + variables dinámicas

Preferir:
  → callApi() con try/catch (el helper del proyecto)
  → sobre wrappers de promesas con retry logic innecesario
```

### 2.4 No agregar lo que no se pide

- Sin comentarios que expliquen lo que hace el código (los nombres ya lo dicen)
- Sin docstrings de múltiples líneas
- Sin manejo de errores para casos imposibles
- Sin validaciones redundantes
- Sin abstracciones para un caso de uso único
- Sin "TODO: mejorar esto" sin contexto de cuándo ni cómo

### 2.5 Respeto por las decisiones previas

Si el código existente hace algo de una forma particular, asumir que hay una razón. Antes de proponer un cambio de patrón, preguntar si la forma actual es intencional.

---

## 3. Reglas de análisis previo

Antes de proponer o implementar cualquier cambio, la IA **debe leer** los siguientes archivos si existen en el proyecto:

### 3.1 Documentos a revisar (en orden)

| Prioridad | Archivo | Qué buscar |
|-----------|---------|-----------|
| 1 | `CLAUDE.md` | Instrucciones específicas del proyecto para la IA |
| 2 | `style_guide.md` | Sistema visual, patrones de código, convenciones |
| 3 | `README.md` | Stack, arquitectura, cómo correr el proyecto |
| 4 | `config.js` | Versión actual, stores, configuración global |
| 5 | `project_workflow.md` | Flujo operativo y decisiones de diseño |
| 6 | `CHANGELOG.md` | Historial de cambios recientes |

### 3.2 Preguntas que la IA debe responder antes de actuar

1. ¿Qué archivo(s) debo modificar?
2. ¿Qué funcionalidad existente puede verse afectada?
3. ¿El cambio requiere actualizar documentación?
4. ¿El cambio requiere bump de versión?
5. ¿Hay algo en la arquitectura actual que contradiga lo que se pide?

Si la respuesta a la pregunta 5 es "sí", comunicarlo antes de implementar.

### 3.3 Cuando el proyecto no tiene documentación

Si no existen los archivos de referencia, la IA debe:
1. Inferir los patrones del código existente (colores usados, naming, estructura de archivos)
2. Mantener exactamente ese estilo en el código nuevo
3. Sugerir crear los documentos faltantes al finalizar la tarea

---

## 4. Reglas para Claude Code

Claude Code es la **herramienta principal de desarrollo** en estos proyectos. Tiene acceso al filesystem, puede ejecutar comandos y tiene visibilidad completa del codebase.

### 4.1 Responsabilidades principales

| Responsabilidad | Descripción |
|-----------------|-------------|
| **Desarrollo de features** | Implementación completa de nuevas funcionalidades |
| **Refactors controlados** | Mejoras de código aprobadas previamente, en etapas |
| **Debugging** | Identificar y corregir bugs con diagnóstico completo |
| **Documentación técnica** | Generar y mantener README, CLAUDE.md, style_guide |
| **Arquitectura local** | Proponer y evaluar estructuras de carpetas y módulos |
| **Preview y verificación** | Usar el servidor de preview para confirmar cambios antes de reportar la tarea como completa |

### 4.2 Selección de modelo

| Modelo | Cuándo usarlo |
|--------|--------------|
| **Sonnet** (default) | Tareas de desarrollo cotidiano: bugfixes, nuevas features, edición de archivos, commits. Es el modelo más eficiente para la mayoría de las sesiones. |
| **Opus** | Análisis arquitectónico complejo, refactors de múltiples archivos, decisiones que afectan toda la base de código, revisiones críticas de seguridad. Usar cuando Sonnet no está alcanzando la profundidad de análisis necesaria. |

### 4.3 Flujo de trabajo estándar en Claude Code

```
1. Leer CLAUDE.md y documentos relevantes
2. Explorar los archivos afectados (no el proyecto completo innecesariamente)
3. Si la tarea es ambigua o de gran alcance → proponer enfoque primero
4. Implementar el cambio mínimo necesario
5. Verificar visualmente con preview server
6. Revisar consola del browser por errores
7. Actualizar config.js con el bump de versión si aplica
8. Reportar: archivos modificados, qué cambió, qué validar
```

### 4.4 Reglas específicas de Claude Code

- **Leer antes de editar.** Siempre usar Read antes de Write o Edit en archivos desconocidos.
- **Editar, no reescribir.** Usar Edit (diff) en vez de Write (full rewrite) salvo que sea un archivo nuevo o el cambio sea > 70% del archivo.
- **Commits solo cuando se pide.** No hacer commit automáticamente salvo instrucción explícita.
- **Push solo con confirmación explícita.** Preguntar siempre antes de `git push`.
- **No usar `--no-verify` ni `--force`** salvo autorización explícita y justificación documentada.
- **Actualizar versión antes del commit.** `config.js` se actualiza en el mismo commit que el cambio funcional.
- **No subagentes innecesarios.** Usar subagentes (Agent tool) solo cuando la tarea requiere paralelismo real o proteger el contexto. No delegar tareas que se pueden resolver directamente.

### 4.5 Comunicación durante la tarea

Claude Code debe dar actualizaciones breves y concretas mientras trabaja:
- Una línea al iniciar: qué va a hacer
- Una línea al encontrar algo relevante: qué encontró
- Al finalizar: resumen de 1–2 líneas de qué cambió y qué sigue

No narrar cada herramienta que usa. El usuario ve los resultados, no necesita comentario de cada `Read` o `Grep`.

---

## 5. Reglas para Codex

Codex actúa como herramienta de **implementación acotada y correcciones puntuales**.

### 5.1 Responsabilidades

| Responsabilidad | Descripción |
|-----------------|-------------|
| **Correcciones específicas** | Arreglar un bug concreto en un archivo concreto |
| **Implementaciones atómicas** | Agregar una función, un componente, un endpoint bien definido |
| **Ajustes de estilo** | Correcciones visuales que no afectan lógica |
| **Completar código parcial** | Rellenar funciones con firma ya definida |
| **Tests y validaciones** | Escribir validaciones para lógica ya existente |

### 5.2 Alcance recomendado por tarea

- Preferir cambios pequeños y focalizados: un problema, un archivo, una función
- Si la tarea requiere modificar muchos archivos o involucra lógica compartida → escalar a Claude Code

### 5.3 Lo que Codex NO debe hacer

- Proponer cambios de arquitectura
- Modificar `auth.js`, `config.js` o `app.js` sin instrucción explícita (son archivos críticos compartidos)
- Cambiar el sistema de estilos o variables CSS
- Hacer commits o push
- Leer más de 5 archivos para resolver una tarea: si necesita más contexto, pedir clarificación
- Introducir dependencias externas

### 5.4 Formato de respuesta esperado de Codex

```
**Archivo:** ruta/al/archivo.js
**Cambio:** descripción de una línea de qué se modificó
**Código:**
[bloque de código]
**Riesgo:** bajo / medio / alto — justificación breve
```

---

## 6. Reglas para ChatGPT

ChatGPT actúa como herramienta de **diseño, planificación y análisis funcional**. No tiene acceso directo al código ni al filesystem, por lo que su rol es estratégico y conceptual.

### 6.1 Responsabilidades

| Responsabilidad | Descripción |
|-----------------|-------------|
| **Diseño funcional** | Definir qué debe hacer una feature antes de implementarla |
| **Arquitectura** | Proponer estructuras de datos, flujos, relaciones entre módulos |
| **Roadmaps** | Ordenar y priorizar el backlog, definir fases de entrega |
| **Requerimientos** | Convertir ideas vagas en especificaciones técnicas concretas |
| **Auditorías** | Revisar documentación, detectar inconsistencias, validar completitud |
| **Documentación narrativa** | Redactar README, guías de uso, changelogs, presentaciones |
| **Análisis de impacto** | Evaluar consecuencias de un cambio antes de implementarlo |
| **Debugging conceptual** | Ayudar a formular hipótesis sobre un bug cuando no hay herramientas de código |

### 6.2 Cómo proveer contexto a ChatGPT

Al iniciar una conversación de diseño o planificación, incluir siempre:

```
Stack: HTML/CSS/Vanilla JS + Google Apps Script + Google Sheets
UI: DM Sans + DM Mono, variables CSS (--primary: #1a3f6b, etc.)
Sin frameworks. Sin npm. Sin dependencias externas.
Backend: Google Apps Script (REST-like, JSON)
Auth: sessionToken en localStorage, roles admin/agente
Versión actual: vX.Y.Z (ver config.js)
```

### 6.3 Output esperado de ChatGPT

Para diseño funcional:
```markdown
## Feature: [nombre]
**Objetivo:** qué problema resuelve
**Flujo:** pasos del usuario
**Datos necesarios:** qué debe devolver la API
**UI:** descripción de componentes (sin CSS, solo estructura)
**Validaciones:** qué debe verificarse
**Roles:** qué puede hacer admin vs agente
**Riesgos:** qué puede fallar
```

Para arquitectura:
```markdown
## Propuesta: [nombre]
**Problema actual:** descripción
**Solución propuesta:** descripción
**Archivos afectados:** lista
**Alternativas descartadas:** por qué no
**Impacto en otros módulos:** lista
**Esfuerzo estimado:** bajo / medio / alto
```

### 6.4 Lo que ChatGPT NO debe hacer

- Generar CSS o JS directamente para pegar en producción (ese es el rol de Claude Code / Codex)
- Proponer cambios de stack sin entender el contexto completo
- Definir arquitecturas que ignoren la restricción "sin dependencias externas"
- Asumir que hay un backend más complejo que Google Apps Script

---

## 7. Control de cambios

Cada vez que una IA modifica el proyecto, debe reportar lo siguiente al finalizar.

### 7.1 Reporte de cambios obligatorio

```markdown
## Cambios realizados

**Archivos modificados:**
- `ruta/archivo1.ext` — descripción del cambio
- `ruta/archivo2.ext` — descripción del cambio

**Impacto:**
- Funcionalidad X ahora hace Y
- La sección Z puede verse afectada (verificar manualmente)

**Riesgos identificados:**
- [ninguno / descripción del riesgo y su probabilidad]

**Validaciones realizadas:**
- [ ] Preview visual en browser
- [ ] Sin errores en consola
- [ ] Funcionalidad principal probada
- [ ] Versión actualizada en config.js

**Próximos pasos sugeridos:**
- [qué queda pendiente si la tarea es parte de algo más grande]
```

### 7.2 Niveles de riesgo

| Nivel | Criterio | Acción recomendada |
|-------|----------|--------------------|
| **Bajo** | Cambio aislado, sin dependencias, fácilmente reversible | Comunicar y continuar |
| **Medio** | Cambio que afecta un flujo existente, más de 2 archivos, lógica de negocio | Documentar, hacer commit atómico, verificar en preview |
| **Alto** | Cambio en auth, config global, estructura de datos, acciones bulk irreversibles | Pedir confirmación explícita antes de ejecutar |

---

## 8. Refactors

Los refactors son cambios de alto riesgo porque modifican código que funciona. Deben seguir un protocolo estricto.

### 8.1 Cuándo está permitido un refactor

Un refactor está justificado cuando:
- El código tiene un bug recurrente que se origina en la estructura actual
- Una feature nueva es imposible o muy costosa con la estructura actual
- El usuario lo solicita explícitamente con un objetivo claro

Un refactor **no está justificado** por:
- "El código podría ser más elegante"
- "Yo lo haría de otra manera"
- "Esta función hace demasiado" (si funciona correctamente)

### 8.2 Protocolo obligatorio para refactors

```
Paso 1: PROPUESTA
  → Describir qué se va a refactorizar y por qué
  → Listar todos los archivos afectados
  → Estimar el riesgo (bajo/medio/alto)
  → Esperar aprobación explícita del usuario

Paso 2: PLAN EN ETAPAS
  → Dividir el refactor en etapas de máximo 1-2 archivos cada una
  → Cada etapa debe ser funcional por sí sola (no dejar el sistema roto entre etapas)
  → Definir cómo se va a validar cada etapa

Paso 3: EJECUCIÓN INCREMENTAL
  → Ejecutar una etapa
  → Verificar que funciona
  → Hacer commit atómico
  → Reportar resultado
  → Continuar solo si el usuario confirma

Paso 4: VALIDACIÓN FINAL
  → Verificar que toda la funcionalidad previa sigue funcionando
  → Documentar los cambios arquitectónicos relevantes
  → Hacer bump de versión si aplica
```

### 8.3 Reglas hard de refactor

- **Nunca** reescribir más de un módulo en una sola sesión
- **Nunca** hacer un refactor al mismo tiempo que una nueva feature (son commits separados)
- **Nunca** asumir que el código antiguo "no sirve" porque parece subóptimo
- **Siempre** mantener el comportamiento observable del usuario exactamente igual después del refactor
- **Siempre** hacer commit antes de empezar un refactor (punto de retorno limpio)

---

## 9. Documentación obligatoria

### 9.1 Archivos requeridos por proyecto

| Archivo | Obligatorio | Cuándo actualizar |
|---------|-------------|-------------------|
| `README.md` | Sí | Al crear el proyecto, al cambiar el stack o la estructura |
| `CLAUDE.md` | Sí (proyectos con Claude Code) | Al agregar reglas nuevas, al cambiar el stack o el versionado |
| `config.js` | Sí | En cada cambio funcional (versión + changelog) |
| `CHANGELOG.md` | Recomendado | Alternativa a changelog en config.js para proyectos sin JS |
| `docs/project_workflow.md` | Recomendado | Al definir freeze zones, convenciones y flujo de trabajo del proyecto |
| `docs/decisions/NNN-nombre.md` | Para decisiones arquitectónicas | Usar `adr_template.md` como base — ver §9.2 |
| `docs/gas-setup.md` | Si el proyecto tiene GAS | Documentar archivos .gs, Script Properties y bootstrap |
| `ai_rules.md` | Este archivo | Al agregar una nueva herramienta de IA o cambiar el workflow |

> **Proyectos nuevos:** seguir `new_project_guide.md` paso a paso para crear la estructura base, el repo en GitHub y el setup de GAS.

> **Templates disponibles en `../project-standards/`:** `project_workflow_template.md` (base para `docs/project_workflow.md`) · `adr_template.md` (base para decisiones en `docs/decisions/`) · `gas_setup_template.md` (setup genérico de GAS) · `new_project_guide.md` (guía completa de proyecto nuevo)

### 9.2 Cuándo la IA debe sugerir actualizar documentación

Siempre que se realice alguno de estos cambios:

- Se agrega un módulo nuevo → actualizar README (estructura de carpetas) y config.js (versión)
- Se cambia una convención de nombre → actualizar style_guide.md
- Se agrega un patrón técnico reutilizable → actualizar style_guide.md
- Se toma una decisión arquitectónica importante → documentar en `docs/decisions/` usando `adr_template.md` (o en CLAUDE.md si es menor)
- Se descubre un gotcha o limitación del stack → documentar para que la siguiente IA no lo repita

### 9.3 Formato del changelog en config.js

```javascript
CHANGELOG: [
  { v: '1.5.0', date: '2026-06-16', desc: 'Descripción del cambio — presente, concisa, sin punto final' },
  { v: '1.4.1', date: '2026-06-10', desc: 'Fix: corrección en exportación CSV con caracteres especiales' },
]
```

Prefijos recomendados en `desc`:
- Sin prefijo → feature nueva visible para el usuario
- `Fix:` → corrección de bug
- `Refactor:` → cambio interno sin impacto visible
- `Docs:` → solo documentación (no requiere bump, pero se puede registrar)

---

## 10. Git

Las convenciones de commits, branches y releases están documentadas en [`style_guide.md §13`](style_guide.md#13-git). Esta sección cubre únicamente **las restricciones específicas para las IAs**.

### 10.1 Lo que la IA no debe hacer en Git sin autorización

- `git push` — siempre pedir confirmación explícita
- `git push --force` — nunca, salvo excepción documentada
- `git reset --hard` — confirmar antes, describir qué se pierde
- `git branch -D` — confirmar antes
- Modificar `.gitignore` para ignorar archivos permanentemente — discutirlo primero

---

## 11. Seguridad

### 11.1 Reglas de datos sensibles

| Dato | Regla |
|------|-------|
| App Keys / Secret Keys de VTEX | Solo en Script Properties de GAS, nunca en frontend |
| Tokens de sesión | Solo en localStorage del cliente, nunca en código fuente |
| Contraseñas | Siempre hasheadas con SHA-256 antes de enviar al backend |
| Credenciales de terceros | Nunca en archivos versionados |
| Emails de usuarios | Solo mostrarlos escapados, nunca como parte de URLs o queries sin encode |

### 11.2 Reglas de seguridad en código generado

Toda IA debe:

```javascript
// ✓ Siempre escapar datos externos antes de insertar en el DOM
el.innerHTML = escapeHtml(userData);
// o mejor:
el.textContent = userData;

// ✗ Nunca
el.innerHTML = userData;                    // XSS
eval(userInput);                           // Code injection
document.write(apiResponse);              // XSS
new Function(userInput)();                // Code injection
```

### 11.3 Acciones que requieren autorización explícita

La IA debe **pedir confirmación antes** de:

- Eliminar cualquier archivo del proyecto
- Modificar `auth.js` (sistema de autenticación)
- Modificar `config.js` (cambia comportamiento global)
- Modificar `.gitignore` (puede exponer archivos sensibles accidentalmente)
- Hacer push a `main` o a cualquier rama
- Ejecutar comandos que afecten datos en producción (GAS, Google Sheets)
- Instalar dependencias externas

### 11.4 Reporte de vulnerabilidades detectadas

Si durante el trabajo en una tarea la IA detecta una vulnerabilidad en código no relacionado con la tarea (XSS, credenciales expuestas, etc.), debe:

1. Mencionar la vulnerabilidad al usuario en el mismo turno
2. No modificar ese código sin autorización (no es parte de la tarea actual)
3. Describir el archivo y línea donde está el problema

---

## 12. Gestión de contexto

### 12.1 Al inicio de una sesión

La IA debe leer los siguientes archivos antes de comenzar a trabajar (si existen):

```
1. CLAUDE.md / ai_rules.md     → reglas del proyecto
2. config.js                   → versión y configuración actual
3. El/los archivos directamente relacionados con la tarea
```

**No leer el proyecto completo** si la tarea es acotada. Leer solo lo necesario.

### 12.2 Mantener coherencia con decisiones previas

Si en la conversación actual o en archivos del proyecto existe una decisión tomada previamente sobre cómo resolver algo, la IA debe:
- Respetarla aunque no sea la forma que elegiría por defecto
- Si cree que la decisión previa fue un error, mencionarlo antes de proponer algo diferente
- No cambiarla silenciosamente

### 12.3 Cuando el contexto se pierde (sesión larga o nueva conversación)

Indicadores de que la IA está operando sin contexto suficiente:
- Propone cosas que ya se implementaron
- Contradice decisiones que ya se tomaron
- Sugiere cambios de stack que se descartaron antes

Cuando esto ocurre, la IA debe reconocerlo y pedir al usuario que provea el contexto relevante en lugar de asumir.

### 12.4 Registrar decisiones importantes

Cuando en una sesión se toma una decisión de diseño o arquitectura importante, la IA debe sugerir documentarla. Formato mínimo:

```markdown
## Decisión: [nombre corto]
**Fecha:** YYYY-MM-DD
**Contexto:** por qué surgió esta decisión
**Decisión tomada:** qué se eligió hacer
**Alternativas descartadas:** qué se evaluó y por qué no
**Impacto:** qué cambia o condiciona en el futuro
```

---

## 13. Aprendizajes

### 13.1 Qué registrar como aprendizaje

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| **Gotcha técnico** | Comportamiento del stack que no es evidente | "GAS tiene límite de 6 min de ejecución por trigger" |
| **Patrón probado** | Solución que funcionó bien y es reutilizable | "normalizeText() con NFD para búsqueda con acentos" |
| **Antipatrón detectado** | Algo que se intentó y causó problemas | "innerHTML con datos de API sin escapar → XSS en staging" |
| **Limitación conocida** | Restricción del entorno que afecta el diseño | "Excel en Windows requiere BOM en CSV para abrir correctamente" |

### 13.2 Dónde registrar aprendizajes

| Alcance | Dónde |
|---------|-------|
| Específico del proyecto | `CLAUDE.md` sección "Notas técnicas" o `project_workflow.md` |
| Reutilizable en todos los proyectos | `style_guide.md` o `ai_rules.md` (este archivo) |
| Decisión puntual con justificación | Comentario inline en el código (solo si el *por qué* no es obvio) |

### 13.3 Aprendizajes acumulados de este stack

Registrar aquí los aprendizajes identificados que aplican a todos los proyectos:

#### GAS (Google Apps Script)
- Tiempo máximo de ejecución: 6 minutos por llamada (plan gratuito), 30 minutos (Workspace)
- Las Script Properties son la única forma segura de almacenar credenciales
- Los Web Apps de GAS requieren `doGet` / `doPost` como punto de entrada
- Cambiar permisos de un Web App desplegado requiere una nueva versión del deployment
- `ContentService.createTextOutput().setMimeType(ContentService.MimeType.JSON)` es el patrón correcto para respuestas JSON

#### Google Sheets
- Los rangos en GAS son 1-indexed (no 0-indexed)
- `getValues()` devuelve un array 2D; `getValue()` devuelve el valor de una celda
- Operaciones de escritura en batch son mucho más eficientes que fila por fila

#### Vanilla JS / Browser
- `normalize('NFD')` + regex para comparación de texto con acentos
- BOM (`﻿`) necesario en CSV para compatibilidad con Excel en Windows
- `URL.createObjectURL` + `URL.revokeObjectURL` para descargas sin servidor
- `position: sticky` en `th` requiere `overflow: auto` en el contenedor, no `hidden`

#### CSS
- `transform: translateX(-100%)` + `transition` es mejor que `display: none` para sidebars (evita layout shift)
- `min-width: 0` en flex children evita overflow en grids con texto largo

---

## 14. Handoff entre IAs

Cuando una IA completa una tarea y el trabajo continuará en otra sesión o con otra herramienta, debe generar un **handoff document** que permita retomar el trabajo sin pérdida de contexto.

### 14.1 Formato del handoff

```markdown
## Handoff — [Nombre de la tarea o feature]
**Fecha:** YYYY-MM-DD
**Herramienta:** Claude Code / Codex / ChatGPT
**Estado:** en progreso / completado / bloqueado

---

### Qué se hizo en esta sesión
- [lista de cambios realizados con archivos afectados]

### Estado actual del sistema
- Versión: vX.Y.Z
- Branch: main / feature/nombre
- ¿Tiene cambios sin commitear? Sí / No
- ¿Está en estado funcional? Sí / No / Parcialmente (explicar)

### Próximos pasos
1. [descripción del próximo paso con archivos a modificar]
2. [...]

### Decisiones pendientes (requieren input del usuario)
- [decisión 1 y las opciones disponibles]
- [...]

### Riesgos y advertencias
- [descripción del riesgo + probabilidad + mitigación sugerida]

### Contexto adicional para la próxima sesión
- [cualquier información que no sea evidente leyendo el código]

### Archivos clave a leer antes de continuar
- `ruta/archivo.ext` — por qué es relevante
```

### 14.2 Cuándo generar un handoff

- Al finalizar una sesión de Claude Code que deja trabajo pendiente
- Antes de pasar una tarea de ChatGPT (diseño) a Claude Code (implementación)
- Cuando una tarea se bloquea por una decisión que requiere al usuario
- Al finalizar una fase de un proyecto más largo

### 14.3 Transferencia ChatGPT → Claude Code

Este es el flujo más común. El handoff debe incluir adicionalmente:

```markdown
### Especificación para implementación
**Feature:** nombre
**Archivos a crear:** lista
**Archivos a modificar:** lista con descripción del cambio
**Lógica de negocio:** descripción de las reglas que debe implementar
**API calls necesarios:** qué acciones del GAS hay que llamar
**UI esperada:** descripción de la pantalla / componente
**Validaciones requeridas:** lista
**Rol de acceso:** admin-only / todos los usuarios
```

---

## 15. Checklist obligatorio antes de finalizar

Toda IA debe validar estos puntos antes de reportar una tarea como completada.

### 15.1 Checklist técnico

```
REQUERIMIENTO
[ ] La funcionalidad solicitada fue implementada completamente
[ ] El comportamiento es exactamente el que se describió (ni más, ni menos)
[ ] Si hubo ambigüedad, se resolvió consultando al usuario

CÓDIGO
[ ] No se rompió ninguna funcionalidad existente
[ ] El código nuevo sigue las convenciones de style_guide.md
[ ] No se introdujeron dependencias externas no autorizadas
[ ] Los datos externos al DOM están escapados con escapeHtml()
[ ] No hay credenciales, tokens ni datos sensibles en el código

VERIFICACIÓN
[ ] Se probó en el preview server (si aplica)
[ ] La consola del browser no muestra errores relacionados con el cambio
[ ] La funcionalidad principal del flujo afectado funciona end-to-end

DOCUMENTACIÓN
[ ] config.js tiene el bump de versión correcto (si aplica)
[ ] El CHANGELOG en config.js tiene la entrada correspondiente
[ ] Si se creó un patrón nuevo, está documentado en style_guide.md
[ ] Si se tomó una decisión arquitectónica, está registrada en `docs/decisions/` (usar `adr_template.md`)

GIT
[ ] Los archivos a commitear están revisados (sin archivos accidentales)
[ ] El mensaje de commit describe el propósito, no el mecanismo
[ ] El commit es atómico (un solo propósito)
```

### 15.2 Checklist de comunicación al finalizar

La IA debe reportar al usuario:

```
✓ QUÉ se hizo (resumen en 1-2 líneas)
✓ QUÉ archivos se modificaron
✓ QUÉ debe validar el usuario manualmente (si aplica)
✓ QUÉ queda pendiente (si la tarea es parte de algo más grande)
✗ NO incluir: explicaciones largas de cómo funciona el código
✗ NO incluir: resumen de cada herramienta que se usó
✗ NO incluir: disculpas o frases de cortesía innecesarias
```

### 15.3 Niveles de completitud

Si la tarea no pudo completarse al 100%, la IA debe ser explícita:

| Estado | Descripción | Qué informar |
|--------|-------------|-------------|
| **Completo** | Todo implementado y verificado | Resumen normal |
| **Parcial** | Implementado pero sin verificar o con limitaciones | Qué falta y por qué |
| **Bloqueado** | No se puede continuar sin input del usuario | Cuál es el bloqueo exacto y qué necesita |
| **Descartado** | Se descubrió que la solución propuesta no es viable | Por qué no es viable y qué alternativas hay |

---

## Apéndice A — Resumen de responsabilidades por herramienta

| Tarea | ChatGPT | Claude Code | Codex |
|-------|---------|-------------|-------|
| Definir requerimientos | ✓ principal | — | — |
| Diseñar arquitectura | ✓ principal | ✓ colabora | — |
| Escribir documentación | ✓ principal | ✓ colabora | — |
| Desarrollar features | — | ✓ principal | ✓ tareas acotadas |
| Corregir bugs | — | ✓ principal | ✓ bugs simples |
| Refactors | ✓ planifica | ✓ ejecuta | — |
| Debugging complejo | ✓ hipótesis | ✓ diagnóstico | — |
| Commits y git | — | ✓ (con confirmación) | — |
| Roadmaps y planificación | ✓ principal | — | — |
| Auditorías de código | ✓ colabora | ✓ principal | — |
| Ajustes de estilo CSS | — | ✓ colabora | ✓ principal |

---

## Apéndice B — Señales de alerta

Situaciones que indican que una IA está operando fuera de estas reglas:

| Señal | Qué hacer |
|-------|-----------|
| La IA reescribió un archivo completo sin pedirlo | Revisar diff, descartar si no es necesario |
| La IA instaló o sugirió una dependencia externa | Rechazar, pedir alternativa en vanilla |
| La IA hizo push sin confirmación | Verificar que no se publicó nada problemático |
| La IA cambió la paleta de colores o la fuente | Revertir, agregar la regla a CLAUDE.md |
| La IA no actualizó config.js después de un cambio funcional | Recordarle la regla, actualizar manualmente |
| La IA propone algo que ya se descartó antes | Indicarle el contexto, evaluar si cambió algo que lo justifique |
| La IA da respuestas muy largas para tareas simples | Pedirle que sea más concisa en el siguiente mensaje |

---

## Apéndice C — Plantilla de contexto inicial para ChatGPT

Copiar y pegar al inicio de conversaciones de diseño:

```
Trabajo en proyectos internos de eCommerce (Marketplace, VTEX, PIM, OMS).
Stack: HTML/CSS/Vanilla JS + Google Apps Script + Google Sheets.
Sin frameworks (no React, no Vue, no npm).
UI: DM Sans + DM Mono. Colores: --primary #1a3f6b, --success #0a7040, --danger #991b1b.
Backend: Google Apps Script (REST-like), datos en Google Sheets.
Auth: roles admin/agente con sessionToken en localStorage.
Prioridades: simplicidad, mantenibilidad, bajo costo de mantenimiento.

Necesito ayuda con: [descripción de la tarea]
```

---

*Documento maestro. Actualizar cuando se incorpore una nueva herramienta de IA, cuando cambie el workflow de desarrollo, o cuando se identifique un patrón recurrente que deba normalizarse.*
