# ADR Template — Decisiones de Arquitectura

**Versión:** 1.0  
**Alcance:** Todos los proyectos bajo `C:\Users\gluna\Documents\Repos`

---

## Convención de nombres de archivo

Dos formatos válidos — elegir uno por proyecto y mantenerlo:

| Formato | Cuándo usarlo | Ejemplo |
|---------|--------------|---------|
| `NNN-descripcion-corta.md` | Proyectos donde el orden o la numeración importa | `001-arquitectura-tres-capas.md` |
| `YYYY-MM-DD-descripcion-corta.md` | Proyectos donde las decisiones son más independientes entre sí | `2026-06-06-normalizacion-archivos.md` |

Usar `docs/decisions/` como carpeta dentro del proyecto.

---

## Template

Copiar desde aquí hacia abajo y guardar como `docs/decisions/<nombre>.md`.

---

# [ADR NNN / YYYY-MM-DD] — [Título corto de la decisión]

## Estado

[Propuesto | Adoptado | Deprecado | Supersedido por ADR-NNN] — [Mes YYYY]

## Contexto

¿Qué problema, necesidad o restricción motivó esta decisión? Describir el estado del sistema antes de la decisión y por qué era necesario tomar una.

## Decisión

¿Qué se decidió hacer? Ser específico: qué patrón, qué tecnología, qué convención, qué cambio estructural.

## Consecuencias

### Positivas

- ...

### Negativas / Limitaciones conocidas

- ...

## Alternativas descartadas

| Alternativa | Motivo del rechazo |
|-------------|-------------------|
| ... | ... |

## Notas

*(Opcional)* Referencias, tickets, contexto adicional, links a documentación externa.

---

## Cuándo escribir un ADR

Escribir un ADR cuando se toma una decisión que:

- **Afecta la arquitectura o la estructura del proyecto** — agregar una capa, cambiar el patrón de autenticación, reorganizar carpetas.
- **Descarta una alternativa válida** — si no quedan razones documentadas, el equipo futuro repetirá el análisis.
- **Tiene consecuencias conocidas que otro desarrollador necesita saber** — limitaciones de performance, timeouts, contratos implícitos.
- **Cierra una investigación** — después de un debugging profundo o una exploración de API que reveló algo no obvio.

No escribir un ADR para:
- Decisiones triviales o reversibles con bajo costo
- Cambios puramente visuales sin impacto en arquitectura
- Convenciones ya documentadas en `style_guide.md` o `ai_rules.md`
