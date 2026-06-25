# Project Workflow — [Nombre del Proyecto]

| Campo | Detalle |
|-------|---------|
| Versión | v1.0 |
| Actualizado | [YYYY-MM-DD] |
| Estado | Activo |
| Documentos relacionados | `../project-standards/ai_rules.md` · `../project-standards/style_guide.md` · `CLAUDE.md` |

> **Cómo usar este template:** copiar a `docs/project_workflow.md` del proyecto. Completar las secciones marcadas con `[...]`. Eliminar o adaptar lo que no aplique. Las secciones §1–§11 son genéricas y se pueden usar sin cambios. §7.1, §12 y §13 requieren personalización.

---

## Índice

1. [Propósito de este documento](#1-propósito-de-este-documento)
2. [Documentos maestros — dónde vive cada regla](#2-documentos-maestros--dónde-vive-cada-regla)
3. [Tipos de cambios y riesgo](#3-tipos-de-cambios-y-riesgo)
4. [Flujo de trabajo estándar](#4-flujo-de-trabajo-estándar)
5. [Flujo de release](#5-flujo-de-release)
6. [Criterios de cierre y deploy](#6-criterios-de-cierre-y-deploy)
7. [Freeze zones del proyecto](#7-freeze-zones-del-proyecto)
8. [Auditoría vs implementación](#8-auditoría-vs-implementación)
9. [Smoke visual y QA](#9-smoke-visual-y-qa)
10. [Manejo de errores y casos nuevos](#10-manejo-de-errores-y-casos-nuevos)
11. [Documentación técnica vs operativa](#11-documentación-técnica-vs-operativa)
12. [Preparación de un proyecto nuevo](#12-preparación-de-un-proyecto-nuevo)
13. [Aprendizajes — [Nombre del Proyecto]](#13-aprendizajes--nombre-del-proyecto)

---

## 1. Propósito de este documento

Este documento define el **workflow operativo** del proyecto: cómo se trabaja, qué pasos se siguen, qué está congelado, qué debe validarse y cómo se cierran etapas.

**No repite** reglas que ya están en los documentos maestros. Referencia directa a ellos cuando aplica.

---

## 2. Documentos maestros — dónde vive cada regla

| Necesito saber... | Ir a... |
|-------------------|---------|
| Cómo trabaja cada IA (Claude, Codex, ChatGPT) | `../project-standards/ai_rules.md` |
| Qué modelo usar, cuándo pedir confirmación, seguridad | `../project-standards/ai_rules.md` |
| Checklist antes de finalizar una tarea | `../project-standards/ai_rules.md` §15 |
| Handoff entre IAs o sesiones | `../project-standards/ai_rules.md` §14 |
| Colores, tipografía, componentes CSS | `../project-standards/style_guide.md` |
| Convenciones de nombre, estructura de carpetas | `../project-standards/style_guide.md` §14 |
| Git: formato de commits, branches | `../project-standards/style_guide.md` §13 |
| Versionado: qué bump aplica a qué cambio | `CLAUDE.md` |
| Instrucciones específicas para Claude Code en este proyecto | `CLAUDE.md` |

---

## 3. Tipos de cambios y riesgo

Cuando sea posible, tratar cada tipo por separado — commits distintos, sesiones distintas.

| Tipo | Descripción | Riesgo | Requiere |
|------|-------------|--------|----------|
| **Documentación** | README, guías, changelogs | Bajo | Commit claro |
| **Visual/UI** | CSS, layout, tipografía, colores | Bajo–Medio | Smoke visual |
| **Funcional** | JS, lógica, render dinámico | Medio | Smoke + consola |
| **Estructural** | Reorganización de carpetas/archivos | Alto | Auditoría previa + smoke de rutas |
| **Refactor** | Cambio interno sin impacto visible | Alto | Auditoría + etapas + validación full |
| **Release** | Versión, deploy, changelog | Alto | Checklist completo |
| **Freeze zone** | [Archivos críticos del proyecto] | Crítico | Ver §7 |

---

## 4. Flujo de trabajo estándar

### 4.1 Secuencia de etapas

```
1. Descubrimiento    → entender problema, usuarios, restricciones
2. Diseño funcional  → qué hace, para quién, con qué alcance (ChatGPT)
3. Diseño técnico    → cómo se construye, qué archivos cambian
4. Implementación    → cambios pequeños, controlados, verificables (Claude Code)
5. Validación        → smoke, QA, consola
6. Documentación     → README, config.js, decisiones
7. Release           → versión, changelog, deploy
8. Handoff           → resumen para retomar (ver ai_rules.md §14)
9. Mantenimiento     → casos nuevos, mejora continua
```

No todas las tareas recorren todas las etapas. Un fix pequeño puede ir directo de implementación a validación. Un refactor grande necesita cada una.

### 4.2 Regla general de auditoría previa

Cuando el alcance no está completamente claro, siempre:

1. Auditoría sin modificar archivos
2. Identificación de riesgos y dependencias
3. Definición de alcance acotado
4. Implementación
5. Validación

Mezclar auditoría e implementación en el mismo paso genera cambios fuera de alcance y regresiones no anticipadas.

### 4.3 Prompt de auditoría recomendado

```
Auditar [módulo o archivo] sin modificar ningún archivo.
Entregar:
- diagnóstico actual
- riesgos detectados
- dependencias críticas
- quick wins
- cambios recomendados priorizados
- próximo paso sugerido de menor riesgo
```

### 4.4 Prompt de implementación recomendado

```
Implementar [cambio concreto].
Modificar solo: [lista de archivos]
No modificar: [restricciones]
Validar: [criterios esperados]
Devolver: archivos modificados, resumen, validaciones, riesgos.
```

---

## 5. Flujo de release

1. Auditoría final del estado del código
2. Definición del alcance exacto del release
3. Implementación pendiente (si quedó algo)
4. Validaciones: smoke, consola, flujo principal
5. Bump de versión (ver `CLAUDE.md` para la regla de bump)
6. Entrada en `CHANGELOG`
7. Commit de release
8. Push a `main`
9. Verificación post-deploy (GitHub Pages, 2–3 min de propagación)
10. Documentación si quedó algo pendiente

**No publicar si:**
- Hay errores críticos sin resolver
- No se validó el flujo principal
- No se sabe exactamente qué cambio se está publicando
- No hay forma de identificar o revertir el cambio

---

## 6. Criterios de cierre y deploy

### Una tarea puede cerrarse cuando:

- El alcance fue cumplido completamente
- Las validaciones definidas pasaron
- La documentación relevante fue actualizada
- No hay errores críticos conocidos
- Los riesgos están documentados
- El resultado está listo para revisión o deploy

### Antes de hacer push a main:

```
[ ] Build / servidor sin errores
[ ] Consola del browser sin errores relacionados con el cambio
[ ] Flujo principal funciona end-to-end
[ ] Versión actualizada (si aplica)
[ ] Entrada en CHANGELOG
[ ] Rollback identificable (commit anterior conocido)
[ ] Riesgos conocidos comunicados
```

---

## 7. Freeze zones del proyecto

Las freeze zones son archivos o módulos que **no deben modificarse** sin auditoría previa y aprobación explícita.

### 7.1 Zonas congeladas

| Zona | Razón |
|------|-------|
| `[archivo crítico 1]` | [razón] |
| `[archivo crítico 2]` | [razón] |
| `[backend / GAS]` | Lógica de negocio y/o credenciales |
| `[datos reales / ejemplos en producción]` | Afectan datos reales o contratos activos |

> **Personalizar esta tabla para el proyecto.** Ejemplos típicos: sistema de autenticación, configuración crítica compartida, contratos con APIs externas, formularios de escritura activos.

### 7.2 Protocolo antes de tocar una freeze zone

1. Auditoría del módulo afectado (sin modificar)
2. Identificación de todas las dependencias
3. Definición de alcance acotado con archivos explícitos
4. Implementación en etapas, no de una sola vez
5. Validación estricta antes de commitear
6. Documentar el cambio en `CLAUDE.md` o `docs/decisions/`

**Si la IA propone tocar una freeze zone fuera del alcance declarado: detener y renegociar.**

### 7.3 Cómo declarar freeze zones en un prompt

```
Modificar solo:
- [archivo permitido 1]
- [archivo permitido 2]

No modificar:
- [freeze zone 1]
- [freeze zone 2]
```

---

## 8. Auditoría vs implementación

La separación entre auditoría e implementación es un principio central, no una sugerencia.

### Por qué siempre separar

Mezclarlas genera:
- Cambios fuera de alcance no detectados
- Regresiones no anticipadas
- Mayor consumo de tokens
- Deuda técnica accidental
- Dificultad para revertir

### Cuándo es obligatorio separar

- Refactors de cualquier escala
- Cambios en JS, rutas, formularios o config
- Cambios que afecten más de 2 módulos
- Proyectos con deuda técnica o estructura heredada
- Cualquier cambio donde el alcance no esté completamente claro

### Cuándo puede omitirse la auditoría formal

- Correcciones de texto, labels o copy
- Ajustes visuales menores bien definidos
- Documentación que no toca lógica
- Cambios que el equipo comprende completamente

Incluso en estos casos: smoke test mínimo post-cambio.

---

## 9. Smoke visual y QA

### 9.1 Checklist de smoke visual (post-cambio UI)

```
[ ] Página carga sin error
[ ] Consola sin errores críticos
[ ] Layout sin elementos rotos o superpuestos
[ ] Navegación funciona (sidebar, links, botones de volver)
[ ] Mobile sin overflow ni elementos cortados
[ ] Formularios visibles con label y submit operativo
[ ] Datos cargan correctamente si aplica
[ ] Branding consistente con el resto del proyecto
```

### 9.2 QA para cambios funcionales (JS, API, datos)

```
[ ] Flujo principal funciona de inicio a fin
[ ] Casos de error son visibles y comprensibles
[ ] Datos enviados/recibidos correctamente
[ ] Sin loops, freezes ni comportamientos inesperados
[ ] Validaciones activas y mensajes claros
[ ] Submit o acción principal ejecuta correctamente
[ ] Comportamiento consistente con distintos estados (vacío, cargando, error, éxito)
```

### 9.3 Específico de GitHub Pages

- Verificar rutas relativas vs absolutas post-deploy
- Verificar que assets (CSS, JS) cargan correctamente
- Dar 2–3 minutos de propagación antes de validar
- Revisar caché del browser si los cambios no aparecen

### 9.4 Validación humana vs asistida por IA

| Usar IA cuando... | Usar validación manual cuando... |
|-------------------|----------------------------------|
| Hay validaciones estáticas o de diff | El smoke es principalmente visual |
| Se revisan payloads o contratos | La prueba requiere criterio UX |
| Se puede hacer smoke mockeado | Implica GAS, Sheets o datos reales |
| El usuario no tiene el entorno abierto | El usuario puede validar más rápido en browser |

Si el usuario dice que validará manualmente: la IA no debe insistir con smoke visual ni gastar tokens adicionales.

---

## 10. Manejo de errores y casos nuevos

Cuando aparece un error o caso no previsto:

1. Guardar evidencia (screenshot, mensaje de consola, pasos para reproducir)
2. Clasificar severidad:
   - **Crítico**: bloquea el flujo principal
   - **Alto**: afecta flujo importante, tiene workaround
   - **Medio**: afecta casos secundarios
   - **Bajo**: mejora menor, no bloqueante
3. Identificar impacto (módulos afectados, datos comprometidos)
4. Corregir con alcance controlado (solo el bug, sin "mejoras de paso")
5. Validar que no haya regresiones en el módulo afectado
6. Documentar la corrección en el commit (patch bump si aplica)

---

## 11. Documentación técnica vs operativa

```
Documentación técnica  →  cómo está construido (para devs, arquitectura, APIs)
Documentación operativa → cómo se usa y para qué (para usuarios, agentes, soporte)
```

### Técnica incluye:
- Arquitectura del proyecto
- Estructura de carpetas y archivos
- Decisiones de diseño técnico (`docs/decisions/`)
- Setup del entorno y credenciales
- Versiones y changelog

### Operativa incluye:
- Para qué sirve cada módulo
- Cuándo usarlo
- Cómo usarlo paso a paso
- Qué resultado esperar
- Qué hacer ante errores
- Lenguaje simple, sin tecnicismos

No mezclar ambas en el mismo documento. El README puede tener una sección de cada tipo, pero los docs detallados van separados.

---

## 12. Preparación de un proyecto nuevo

### 12.1 Orden recomendado

1. **ChatGPT primero** si la idea no está definida: convertir la idea en alcance claro, identificar usuarios y restricciones
2. **Claude Code después**: crear la carpeta, inicializar git, generar archivos base
3. Si el alcance ya está claro desde el inicio, saltar el paso de ChatGPT

### 12.2 Archivos mínimos desde el día 1

```
README.md                → qué es, cómo se usa, cómo se valida
CHANGELOG.md             → historial de versiones (o en config.js)
CLAUDE.md                → instrucciones para la IA: stack, versionado, convenciones
docs/project_workflow.md → este documento, adaptado al proyecto
.gitignore               → credenciales, dependencias, archivos generados
docs/roadmap.md          → próximos pasos
```

### 12.3 Estructura de directorios

Adaptar según el tipo de proyecto. Tres variantes comunes:

**Tipo A — Multi-página con sidebar (dashboard):**
```
/modules/<area>/     → páginas agrupadas por dominio
/assets/             → CSS, JS compartidos, imágenes
/apps-script/        → backend GAS (si aplica)
/docs/               → decisiones, roadmap, workflow
config.js            → configuración central, versión, changelog
app.js               → helpers compartidos del frontend
auth.js              → sesión y autenticación (si aplica)
styles.css           → estilos globales
```

**Tipo B — SPA (single-page application):**
```
/src/js/             → módulos JS organizados por responsabilidad
/src/css/            → estilos por módulo o globales
/src/data/           → JSON mock para desarrollo
/apps-script/        → backend GAS (si aplica)
/docs/               → decisiones, roadmap, workflow
index.html           → punto de entrada único
```

**Tipo C — Herramienta estática sin auth:**
```
/modules/<nombre>/   → cada módulo con su propio index.html
/templates/          → archivos de producción (freeze zone)
/examples/           → datos de ejemplo
/assets/             → CSS y JS compartidos
/docs/               → decisiones, roadmap, workflow
config.js            → catálogo central y configuración
```

### 12.4 Configuración crítica antes de escribir lógica

- Definir dónde viven las credenciales (Script Properties del GAS si aplica — nunca en código)
- Documentar en `docs/gas-setup.md` cómo configurar el entorno desde cero (si aplica)
- Validar un flujo mínimo end-to-end antes de construir módulos encima

### 12.5 Convenciones específicas del proyecto

> Completar esta tabla con las convenciones de nombres, rutas y patrones adoptados en este proyecto.

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| [Tipo de archivo] | [Convención] | [Ejemplo] |
| [Tipo de función] | [Convención] | [Ejemplo] |
| [Variable de config] | [Convención] | [Ejemplo] |

---

## 13. Aprendizajes — [Nombre del Proyecto]

> Documentar aquí los aprendizajes concretos del proyecto: patrones que funcionaron, errores que se repitieron, hallazgos técnicos no obvios. Estos aprendizajes son la memoria operativa del proyecto y evitan repetir los mismos análisis en futuras sesiones.

### [13.1 Primer aprendizaje]

Descripción del aprendizaje, qué pasó, cuál es la regla derivada.

### [13.2 Segundo aprendizaje]

...
