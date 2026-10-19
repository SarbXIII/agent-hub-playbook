# agent-hub-playbook

Un modelo operativo práctico para asistentes de IA que coordinan trabajo real en varios proyectos — y no se degradan según crecen.

Casi todos los montajes de agentes empiezan fuertes y se pudren en silencio: el fichero de instrucciones engorda, todas las skills se anuncian a todos los agentes, el contexto se llena de ruido, los permisos viven en frases del prompt y los mismos errores vuelven cada semana. Este playbook describe la alternativa: un **hub** que coordina, **sesiones de proyecto aisladas** que ejecutan, **agentes de rol** con permisos deny-first, **skills** que se cargan bajo demanda y un **libro de lecciones** que convierte cada fallo en un cambio de la máquina.

No es un framework. No hay nada que instalar. Es una especificación, un juego de plantillas y las lecciones de campo que nos costaron incidentes reales.

**[Read this in English →](README.md)** · **[Guía completa en español →](docs/es/modelo.md)**

---

## El modelo en 60 segundos

| Pieza | Qué es | Por qué existe |
|---|---|---|
| **Hub** | Un asistente gerente en un espacio personal, con reglas escritas y memoria | Un solo sitio que decide, despacha y cierra — nunca dos gerentes vivos a la vez |
| **Islas** | Una sesión y una configuración por proyecto | Mezclar contextos hace que las sesiones crezcan, se compacten y pierdan precisión |
| **Agentes de rol** | Fichas Markdown: modelo, límite de pasos, permisos, allowlist de skills | El modelo caro piensa; el barato ejecuta; los lectores no tienen shell |
| **Skills** | Procedimientos bajo demanda: descripción siempre visible, cuerpo solo al invocarlas | Un catálogo de 180 skills cuesta ~31k tokens por arranque en frío; una allowlist por rol lo bajó a ~12k (medido) |
| **Lecciones** | Fecha · error · causa raíz · acción · estado · evidencia | Un error que vuelve es una regresión, no mala suerte |
| **Cierre** | Medir contexto → chequeo de consistencia → traspaso → rotación | Una sesión nueva arranca de un estado escrito, no de releer el historial |

La especificación completa — principios, fases, entrevista, anti-patrones, protocolo operativo, plantillas — está en **[docs/es/modelo.md](docs/es/modelo.md)** (español) y **[docs/model.md](docs/model.md)** (inglés).

---

## Arranque rápido

1. **Lee el modelo** (`docs/es/modelo.md`) y haz la entrevista de la sección 9 con tu propio asistente. No la saltes: el sistema se adapta a tu contexto, no se copia.
2. **Escribe la constitución** (≤200 líneas) a partir de `templates/constitution.md`. Corta: un fichero largo se obedece menos.
3. **Crea dos agentes** desde las plantillas: un explorador de solo lectura y un revisor sin herramienta shell. Comprueba que el revisor no puede ejecutar nada de verdad.
4. **Añade 3 skills reales** y una allowlist por agente. Mide tu arranque en frío antes y después.
5. **Haz un despacho real de principio a fin:** encargo → traspaso en las notas → export de la transcripción → borrado de la sesión.

Las fases, criterios de aceptación y el checklist completo están en las secciones 8 y 16 del modelo.

---

## Qué es — y qué no

**Es:** un modelo operativo, una especificación sanitizada, plantillas y lecciones de uso real (permisos, coste, contexto, cierre).

**No es:** un producto, una librería, un servicio alojado, una colección de prompts ni un sustituto de tu herramienta de agentes. Asume una herramienta con subagentes, skills y reglas de permisos (lo usamos en OpenCode; los conceptos se mapean a otras CLIs).

**Reglas duras que no se negocian:** evidencia antes que afirmaciones, permisos deny-first, sin shell para agentes de solo lectura, secretos fuera del chat y una lección registrada por cada fallo.

---

## Cómo evoluciona este repo

Es una **edición curada**, no un espejo de un sistema privado. Un cambio se publica solo si pasa tres filtros:

1. Es genérico (no atado a una persona, proyecto o proveedor).
2. No contiene datos personales, rutas, costes, nombres de clientes ni incidentes identificables.
3. Está probado en uso real.

Las versiones siguen `CHANGELOG.md`. Las lecciones genéricas se añaden en `lessons/`; la especificación se revisa cuando cambia el modelo. No hay compromiso de soporte — esto es un playbook, no un producto. Se agradecen correcciones y discusión.

---

## Estructura

```
docs/
  model.md            edición en inglés
  es/modelo.md        edición en español
templates/
  constitution.md     reglas siempre cargadas (≤200 líneas)
  agent.readonly.md   agente de solo lectura, sin shell
  agent.reviewer.md   revisor: señala, no arregla
  skill.md            procedimiento bajo demanda
  memory.md           hechos fechados, verificado vs supuesto
  lesson.md           fallo → cambio de máquina
  handoff.md          estado + siguiente paso exacto
lessons/
  agent-shell-permissions.md
  context-budget.md
CHANGELOG.md
LICENSE
```

---

## Lecciones de campo incluidas

- **Las allowlists de shell casan por subcadena** — `*git diff*` también ejecuta `cmd /c "… & git push --force"`, y una allow de agente **anula el deny global**. Un agente de solo lectura no debe tener la herramienta.
- **El catálogo de skills es un impuesto de arranque** — 180+ skills anunciadas ≈ 31k tokens antes de la primera palabra; las allowlists por agente y `autoinvoke: false` lo bajan.
- **Un guard es un tripwire, no una jaula** — las listas de deny globales paran lo evidente; no paran un script que borra por dentro ni una copia fuera del worktree.

---

## Licencia

MIT. Ver [LICENSE](LICENSE).
