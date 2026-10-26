---
description: This document contains all development rules and guidelines for this project, applicable to all AI agents (Claude, Cursor, Codex, Gemini, etc.).
alwaysApply: true
---

## 0. Scope and actual VES architecture

These standards apply across the VES workspace, but project-specific architecture
and conventions documented in each BCCR directory take precedence where they
differ. The three `Bccr.Supen.Ves.*` directories are separate components, not
three equivalent layers of one application:

| Directory | Actual responsibility | Main technology |
|---|---|---|
| `Bccr.Supen.Ves.UI` | Browser UI for STI and SEC; calls the API over HTTP | React 19, JavaScript/JSX, Vite, Vitest |
| `Bccr.Supen.Ves.Api` | HTTP orchestration and service gateway; its `SI.SG` implementations call MAPI | .NET 8 / ASP.NET Core |
| `Bccr.Supen.Ves.Mapi` | Business and data services; its `MAPI` controllers call `BW`, which calls `DA` and SQL Server | .NET 8 / ASP.NET Core, EF Core 8 |

The end-to-end flow is **UI → API → MAPI → database**. Within the API, the
relevant flow is **API Controller → SI.SG → MAPI over HTTP**; within MAPI, it is
**MAPI Controller → BW → DA → SQL Server**. `Abstracciones` contains contracts
and DTOs within the .NET projects; it does not make the UI, API, and MAPI one
shared deployment layer. Consult each component's `README.md`, `AGENT.md`, and
technical documentation before applying a project-specific rule.


### 0.1 Matriz de transformación:  → VES

> Se utiliza como fuente de disciplina de ingeniería y operación de agentes.
> Cuando una regla genérica contradice la arquitectura, lenguaje, herramientas
> o procesos reales de VES, prevalece el estándar VES y la documentación
> específica del componente.

| Regla del documento 1 | Intención útil | Aplicación en VES |
|---|---|---|
| “Small tasks, one at a time” | Reducir riesgo y facilitar revisión | Cambios pequeños, coherentes e incrementales; no avanzar a trabajo ajeno al alcance |
| TDD obligatorio | Proteger comportamiento y regresiones | Preferir pruebas que demuestren comportamiento; test-first cuando aporte valor y encaje con la tarea |
| Todo el código debe estar tipado | Reducir errores por tipos | C# mantiene tipado estático; en UI JS/JSX usar validación, contratos y tooling existente sin imponer TypeScript |
| Naming claro | Facilitar lectura y mantenimiento | Se conserva; aplicar convenciones específicas de frontend/backend |
| Question assumptions | Evitar inferencias incorrectas | Verificar código, documentación y contratos antes de asumir |
| Pattern detection | Detectar duplicación | Señalar patrones repetidos y abstraer solo cuando exista una regla común real |
| English only | Uniformidad | No se traslada literalmente: VES usa principalmente español; conservar nombres externos cuando la compatibilidad lo requiera |
| Skills en `ai-specs/skills` | Reutilización de instrucciones | Se conserva según la estructura documentada del proyecto |
| Planning con modelo específico | Aumentar calidad de razonamiento | No imponer proveedor/modelo; usar el flujo y capacidad disponibles sin cambiar settings por una regla genérica |
| `ai-specs` como fuente canónica universal | Evitar duplicación | Usar la ubicación canónica documentada para cada artefacto; no asumir que todo debe vivir en `ai-specs` |
| Symlinks multiagente | Mantener portabilidad | Verificar referencias/symlinks cuando existan; no exigirlos donde no sean portables |
| OpenSpec post-apply | Mantener especificación como fuente de verdad | Se conserva para cambios OpenSpec activos y se detalla el orden obligatorio |

### 0.2 Jerarquía de fuentes de verdad

Ante una discrepancia, usar este orden:

1. Requerimiento explícito del usuario o tarea actual.
2. Artefactos activos de OpenSpec, cuando el trabajo pertenezca a un cambio OpenSpec.
3. Documentación específica del componente afectado (`README.md`, `AGENT.md`,
   `SPEC.md`, guías técnicas y estándares backend/frontend/documentación).
4. Este `base-standards.md`.
5. Buenas prácticas genéricas o conocimiento del agente.

No utilizar una recomendación genérica para invalidar una convención confirmada
por el repositorio.


## 1. Core Principles

- **Focused changes**: Keep changes incremental and limited to the requested scope.
- **Testing**: Prefer tests that demonstrate new behavior or prevent regressions; follow the test framework and patterns already used by the project.
- **Type safety**: Preserve compile-time and runtime validation supported by the project's language and tooling. Do not impose TypeScript or convert the UI's existing JavaScript/JSX just to satisfy a generic rule.
- **Clear Naming**: Use clear, descriptive names for all variables and functions.
- **Question Assumptions**: Always question assumptions and inferences.
- **Consistency**: Reuse established project patterns and call out meaningful duplication before introducing a new abstraction.

The useful practices from the original standards are retained: focused and
incremental work, testing new behavior and regressions, clear naming,
questioning assumptions, and checking for repeated patterns. They are applied
using each component's actual language, test framework, and architecture rather
than as one-size-fits-all rules.


### 1.1 Cambios pequeños y enfocados

> “Baby steps” se conserva como principio de reducción de riesgo, 
> no como obligación de detener la tarea después de cada microcambio.

Reglas:

- Mantener cada cambio limitado al objetivo solicitado.
- Preferir modificaciones que puedan revisarse y validarse de forma aislada.
- No mezclar refactorizaciones, upgrades, cambios de formato o limpieza general
  con una funcionalidad no relacionada.
- Si una tarea requiere varios archivos o capas, completar el conjunto mínimo
  coherente necesario para que el comportamiento quede correcto.
- No avanzar hacia una segunda funcionalidad solo porque se detectó durante la
  implementación.

**Evitar**

```text
“Tengo que agregar un endpoint, así que también voy a actualizar paquetes,
renombrar carpetas y reestructurar toda la solución.”
```

**Recomendado**

```text
“Implemento el endpoint, su contrato, la integración requerida y las pruebas
pertinentes. Los demás hallazgos se documentan aparte.”
```

### 1.2 Pruebas y TDD

Aplicar test-first cuando:

- el comportamiento puede expresarse claramente antes de implementar;
- se corrige una regresión reproducible;
- una regla de negocio tiene entradas/salidas verificables;
- el componente ya posee una suite que facilita ese flujo.

No forzar TDD cuando:

- la tarea es únicamente documental o de configuración;
- se trata de una investigación/exploración;
- el componente no tiene infraestructura de prueba apropiada y crearla excede
  el alcance;
- el cambio es puramente visual y la validación adecuada es otra, aunque deben
  agregarse pruebas de comportamiento si existe riesgo relevante.

Siempre que se cambie comportamiento existente, evaluar qué prueba previene la
regresión.

### 1.3 Seguridad de tipos y contratos

- En .NET/C#, conservar tipos estáticos, nullable annotations, contratos y DTOs.
- En la UI JavaScript/JSX, no introducir TypeScript como requisito transversal.
  Usar las validaciones, contratos de API, linting y patrones documentados para
  reducir errores.
- No usar escapes del sistema de tipos (`dynamic`, `object`, casts o supresiones
  injustificadas) en backend.
- No inventar shapes de respuesta del API en frontend; verificar el contrato
  existente.

### 1.4 Nombres claros

Usar nombres que expresen intención de negocio o responsabilidad.

**Evitar**

```text
x
tmp
data1
proc
flag
helper2
```

**Preferir**

```text
codigoEntidad
estaCargando
ObtengaFondosAsync
mensajeDeError
configuracionEndpoint
```

Las reglas exactas de casing, idioma, sufijos y nombres de archivo se definen en
`backend-standards.md` y `frontend-standards.md`.

### 1.5 Cuestionar supuestos

> Antes de implementar una inferencia, buscar evidencia en el repositorio.

Un agente o desarrollador debe distinguir entre:

- **confirmado**: aparece en código, configuración o documentación vigente;
- **inferido**: parece probable, pero no está confirmado;
- **propuesto**: mejora sugerida que aún no forma parte del sistema.

Ejemplo:

```text
Confirmado: la UI consume Bccr.Supen.Ves.Api.
Inferido: un endpoint nuevo probablemente seguirá una ruta similar a otro.
Propuesto: crear una abstracción común para tres flujos repetidos.
```

No presentar una inferencia o propuesta como si fuera parte existente de VES.

### 1.6 Detección de patrones sin sobrearquitectura

> Detectar duplicación no implica abstraer inmediatamente.

Cuando se detecte código repetido:

1. Determinar si representa la misma regla o solo una similitud superficial.
2. Verificar si cambia por las mismas razones.
3. Revisar si el proyecto ya tiene una abstracción equivalente.
4. Extraer solo si mejora mantenibilidad, prueba o consistencia.
5. Evitar frameworks internos, factories, wrappers o helpers genéricos sin una
   necesidad demostrada.

“DRY” no debe convertirse en “abstraer todo”.

### 1.7 Consistencia antes que preferencia personal

No sustituir patrones existentes por la preferencia del desarrollador o agente
cuando ambos sean técnicamente válidos.

Ejemplos:

- No cambiar MUI por otra biblioteca solo por preferencia.
- No migrar MSTest a xUnit sin una tarea específica.
- No convertir JS/JSX a TypeScript por una regla global.
- No reemplazar contratos BCCR por una abstracción genérica propia.
- No cambiar el idioma de identificadores públicos solo por estilo.


## 2. Language Standards
- Use the language and naming conventions already established in the component.
  The BCCR VES UI and API code/documentation primarily use Spanish, including
  identifiers in existing code. Do not translate or rename established APIs,
  database objects, or identifiers to satisfy a workspace-wide English-only rule.
- Keep new technical writing clear and consistent with nearby documentation;
  Spanish is valid and is the prevailing language in these VES projects.
- Use descriptive names, messages, and tests. Preserve externally defined names
  and conventions where compatibility requires them.


### 2.1 Idioma y compatibilidad

> El documento 1 exige inglés para código, documentación, Azure DevOps esquemas,
> configuración, commits y tests. Esa regla no aplica totalmente  al repositorio VES.

Reglas VES:

- Mantener el idioma ya establecido en cada componente.
- Español es válido y predominante en documentación y código de dominio VES.
- Preservar nombres definidos externamente por contratos, frameworks,
  bibliotecas, SQL, APIs o integraciones.
- Evitar mezclar español e inglés dentro del mismo concepto sin necesidad.
- No traducir nombres existentes si ello altera compatibilidad.
- Los mensajes de error, tests y logs deben ser descriptivos y consistentes con
  el módulo donde viven.

**Ejemplo**

```text
Correcto en un módulo VES en español:
codigoEntidad
mensajeDeError
ObtengaFondosAsync
```

No renombrar estos identificadores a inglés únicamente para satisfacer una
regla transversal antigua.

### 2.2 Documentación y commits

- La documentación nueva debe usar el idioma predominante del área.
- Los commits deben seguir la convención real del repositorio/equipo.
- No imponer inglés ni español si existe una convención explícita distinta.
- Los nombres de archivos, artefactos OpenSpec o claves requeridas por una
  herramienta deben conservar el formato que esa herramienta espera.


## 3. Specific standards

For detailed standards and guidelines specific to different areas of the project, refer to:

- [Backend Standards](/docs/backend-standards.md) - API development, database patterns, testing, security and backend best practices
- [Frontend Standards](/docs/frontend-standards.md) - React components, UI/UX guidelines, and frontend architecture
- [Documentation Standards](/docs/documentation-standards.md) - Technical documentation structure, formatting, and maintenance guidelines, including AI standards like this document
- [OpenSpec Tasks Mandatory Steps](/docs/openspec-tasks-mandatory-steps.md) - Required checklist and execution rules when creating or updating OpenSpec `tasks.md` files


### 3.1 Selección del estándar correcto

| Tipo de cambio | Documento principal |
|---|---|
| React, UI/UX, hooks, pruebas frontend | `frontend-standards.md` |
| API .NET, MAPI, EF Core, SQL Server, pruebas backend | `backend-standards.md` |
| Markdown, SPEC, README, guías y artefactos técnicos | `documentation-standards.md` |
| Creación/actualización de `tasks.md` de OpenSpec | `openspec-tasks-mandatory-steps.md` |
| Reglas transversales y portabilidad multiagente | `base-standards.md` |

Si una tarea cruza varias áreas, aplicar todos los estándares pertinentes sin
forzar una arquitectura común entre componentes independientes.


## 4. Project Skills

- Skills live in `ai-specs/skills`.
- When a request matches a skill, load and follow the corresponding `SKILL.md` automatically before continuing.
- Also load any referenced files in the skill folder (for example, `references/*.md`) when the skill requires them.


### 4.1 Uso seguro de skills

> La carga automática de skills se conserva, pero una skill no puede sobreescribir silenciosamente
> contratos reales del repositorio.

Cuando una tarea coincide con una skill:

1. Leer su `SKILL.md`.
2. Leer referencias requeridas por esa skill.
3. Verificar que las instrucciones aplican al componente actual.
4. Resolver conflictos mediante la jerarquía de fuentes definida arriba.
5. Ejecutar únicamente acciones compatibles con el alcance solicitado.

No copiar una skill de otro stack como patrón obligatorio de VES sin validarla.

### 4.2 Artefactos compartidos entre agentes

Cuando una skill, agente o referencia esté diseñada para múltiples asistentes:

- mantener una única fuente canónica documentada;
- evitar copias divergentes;
- usar referencias o symlinks solo cuando sean compatibles con el sistema
  operativo y repositorio;
- documentar cualquier excepción necesaria.

## 5. Planning workflows

Use the planning workflow and model supported by the active tools and project.

Planning workflows must run with Opus high reasoning o similar para planificación y decisiones complejas.

This requirement applies to:
- `enrich-us`
- `openspec-ff-change`
- `openspec-continue-change`

Do not change agent or model settings merely to satisfy a vendor-specific workspace rule.

### 5.1 Estrategia explícita de modelos: planificación avanzada y ejecución eficiente

> Se conserva explícitamente la intención de utilizar un modelo de razonamiento avanzado, similar a
> **Opus**, para planificación y decisiones complejas, y un modelo de ejecución
> sólido pero menos costoso, similar a **Sonnet**, para implementar trabajo cuya
> especificación, arquitectura y reglas ya están claras. La regla se expresa por
> **capacidad**, no por proveedor, para mantener portabilidad entre Claude,
> GitHub Copilot, Cursor, Codex, Gemini u otras plataformas.

#### Principio general

Usar dos niveles lógicos de modelo:

```text
                 ┌──────────────────────────────┐
                 │  MODELO DE PLANIFICACIÓN    │
                 │  AVANZADO / OPUS-EQUIVALENT │
                 │                              │
                 │  Analiza • Diseña • Decide  │
                 │  Detecta riesgos • Planifica│
                 └──────────────┬───────────────┘
                                │
                         plan aprobado
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │   MODELO DE EJECUCIÓN       │
                 │   SONNET-EQUIVALENT         │
                 │                              │
                 │ Implementa • Prueba • Ajusta│
                 │ siguiendo reglas explícitas │
                 └──────────────┬───────────────┘
                                │
                         duda / desviación
                                │
                                ▼
                    ESCALAR AL PLANIFICADOR
```

Los nombres **Opus-equivalent** y **Sonnet-equivalent** describen niveles de
capacidad, no productos obligatorios.

- **Opus-equivalent**: el modelo de mayor capacidad de razonamiento disponible
  y adecuado a la tecnología involucrada.
- **Sonnet-equivalent**: un modelo de desarrollo general competente, más
  eficiente, capaz de seguir con precisión especificaciones, estándares,
  contratos y tareas ya definidas.

La selección debe considerar además el stack de la tarea. Para una modificación
.NET, el modelo avanzado debe demostrar buen desempeño en C#/.NET/arquitectura;
para frontend, en React/JavaScript/Vite/MUI; para datos, en SQL Server/EF Core;
para OpenSpec, en análisis de requisitos y consistencia documental.

#### Cuándo usar obligatoriamente un modelo Opus-equivalent

Usar el modelo avanzado para **planificar o revisar** cuando la tarea incluya
uno o más de estos factores:

- cambios arquitectónicos;
- cambios que atraviesan UI → API → MAPI → base de datos;
- definición o modificación de contratos entre componentes;
- decisiones de seguridad, autenticación, autorización, certificados o datos
  sensibles;
- diseño de nuevos patrones, abstracciones o componentes compartidos;
- cambios de esquema, stored procedures o integración con DBA;
- migraciones tecnológicas;
- cambios de dependencias con impacto transversal;
- análisis de concurrencia, rendimiento o resiliencia;
- refactorizaciones amplias;
- requisitos ambiguos o con varias interpretaciones válidas;
- enriquecimiento de historias, especificaciones o cambios OpenSpec complejos;
- elaboración del plan de implementación para una funcionalidad de varias
  capas;
- revisión de una solución cuando un error persiste después de intentos
  razonables;
- evaluación de hallazgos que puedan cambiar el alcance original.

El modelo avanzado debe producir, como mínimo cuando aplique:

1. entendimiento del requerimiento;
2. supuestos confirmados y pendientes;
3. arquitectura/componentes afectados;
4. contratos que deben preservarse;
5. archivos o áreas probables de cambio;
6. riesgos técnicos;
7. estrategia de pruebas;
8. orden de implementación;
9. criterios de aceptación/verificación;
10. restricciones explícitas sobre lo que **no** debe modificarse.

#### Cuándo usar un modelo Sonnet-equivalent

Una vez que el plan, reglas y contratos están claros, utilizar un modelo de
ejecución eficiente para:

- crear o modificar código siguiendo un plan aprobado;
- implementar endpoints ya diseñados;
- crear componentes React con comportamiento especificado;
- adaptar DTOs o mappings ya definidos;
- escribir pruebas cuyos escenarios ya están determinados;
- aplicar cambios repetitivos o mecánicos;
- corregir linting, formato o warnings bien comprendidos;
- actualizar documentación conforme a decisiones ya tomadas;
- ejecutar refactorizaciones locales explícitamente descritas;
- implementar tareas de `tasks.md` suficientemente detalladas;
- realizar ajustes derivados de una revisión cuando la corrección sea
  inequívoca.

El modelo de ejecución **no debe reinterpretar la arquitectura ni ampliar el
alcance por iniciativa propia**. Su función principal es materializar de forma
consistente las decisiones ya establecidas.

#### Regla de escalamiento al modelo avanzado

El modelo Sonnet-equivalent debe detener la decisión local y escalar nuevamente
a un Opus-equivalent cuando aparezca cualquiera de estas situaciones:

- la implementación contradice el plan;
- descubre una arquitectura diferente de la asumida;
- falta un contrato necesario;
- debe decidir entre varias alternativas arquitectónicas relevantes;
- encuentra impacto no previsto en otro componente;
- el cambio exige modificar seguridad o permisos;
- requiere introducir una nueva dependencia;
- necesita cambiar una interfaz pública;
- encuentra una posible pérdida de compatibilidad;
- un test revela que la especificación puede estar equivocada;
- el cambio solicitado afecta artefactos OpenSpec no contemplados;
- se requieren cambios sustanciales en stored procedures/esquema;
- existe riesgo significativo de datos, rendimiento o concurrencia;
- la solución implica una excepción a los estándares del proyecto.

El escalamiento no significa abandonar la tarea. Significa devolver la nueva
información a la etapa de razonamiento avanzado, actualizar el plan y luego
continuar la ejecución.

#### Matriz de selección de modelo

| Tipo de trabajo | Modelo recomendado | Motivo |
|---|---|---|
| Cambio textual simple y acotado | Sonnet-equivalent | Reglas claras, bajo riesgo |
| Corrección local con causa conocida | Sonnet-equivalent | Ejecución directa |
| Implementación de tarea OpenSpec detallada | Sonnet-equivalent | El diseño ya está definido |
| Creación de tests para comportamiento especificado | Sonnet-equivalent | Escenarios conocidos |
| Nueva historia con ambigüedad | Opus-equivalent | Requiere análisis |
| Diseño API/MAPI | Opus-equivalent | Contratos y arquitectura |
| Cambio UI + API + MAPI | Opus-equivalent | Impacto transversal |
| Seguridad/autenticación | Opus-equivalent | Alto riesgo |
| Modelo de datos/SQL complejo | Opus-equivalent | Riesgo de integridad/rendimiento |
| Refactorización transversal | Opus-equivalent | Decisiones estructurales |
| Investigación de bug difícil | Opus-equivalent inicialmente | Necesita diagnóstico |
| Fix mecánico después del diagnóstico | Sonnet-equivalent | Solución ya determinada |
| Revisión final de cambio de alto riesgo | Opus-equivalent | Validación independiente de decisiones |

#### Ejemplos por tecnología

**Backend .NET / API / MAPI**

```text
Opus-equivalent:
- decide dónde pertenece una nueva regla entre API, SG, MAPI, BW y DA;
- revisa contratos, seguridad, transacciones y estrategia de pruebas.

Sonnet-equivalent:
- implementa las clases/métodos definidos;
- agrega DI siguiendo patrones existentes;
- crea las pruebas MSTest/xUnit;
- ejecuta correcciones locales.
```

**Frontend React**

```text
Opus-equivalent:
- determina ownership del estado;
- decide composición de componentes;
- analiza integración con API, accesibilidad y riesgos de UX.

Sonnet-equivalent:
- implementa JSX/hooks según el diseño;
- crea pruebas Vitest/RTL;
- aplica tokens MUI y reglas responsive ya establecidas.
```

**SQL Server / EF Core**

```text
Opus-equivalent:
- analiza frontera transaccional, impacto del SP, rendimiento y compatibilidad.

Sonnet-equivalent:
- implementa el acceso DA parametrizado ya diseñado;
- agrega mappings y pruebas definidas.
```

**OpenSpec / requisitos**

```text
Opus-equivalent:
- enriquece requerimientos;
- detecta ambigüedades y escenarios;
- diseña el cambio y las tareas.

Sonnet-equivalent:
- ejecuta tasks.md una vez que requisitos, escenarios y restricciones están
  suficientemente definidos.
```

#### No asociar “modelo más avanzado” con “hacer todo”

El modelo Opus-equivalent no debe utilizarse para ejecutar mecánicamente todo el
cambio solo por ser más capaz. Su mayor valor está en:

- razonamiento;
- planificación;
- arquitectura;
- ambigüedad;
- revisión de alto riesgo.

El modelo Sonnet-equivalent debe recibir instrucciones suficientemente claras
para “maquilar” —implementar de forma disciplinada— sin tener que rediseñar la
solución.

#### Independencia del proveedor

Ejemplos de nombres comerciales pueden cambiar con el tiempo. Por ello, la
regla normativa es:

```text
PLANIFICACIÓN COMPLEJA
→ modelo de razonamiento avanzado disponible para el stack
→ Opus-equivalent

EJECUCIÓN CON REGLAS CLARAS
→ modelo competente y eficiente de propósito general/código
→ Sonnet-equivalent
```

Si una plataforma no ofrece dos modelos distintos, usar el **nivel de
razonamiento/configuración** que más se aproxime a estas dos funciones.

No modificar automáticamente configuraciones persistentes del usuario,
repositorio o IDE para cambiar de modelo salvo que el flujo de la herramienta
lo requiera y esté expresamente autorizado. La selección de modelo debe hacerse
preferentemente mediante el mecanismo normal de la sesión/agente.

### 5.2 Patrón de ejecución recomendado

Para tareas de código o estándares:

```text
1. INSPECCIONAR
   ↓
2. CONFIRMAR ARQUITECTURA/CONTRATOS
   ↓
3. DEFINIR CAMBIO MÍNIMO
   ↓
4. IMPLEMENTAR
   ↓
5. VALIDAR
   ↓
6. REVISAR CAMBIOS COLATERALES
   ↓
7. INFORMAR RESULTADO
```

#### Inspeccionar

Leer primero los archivos directamente relacionados y la documentación
específica del componente.

#### Confirmar

Distinguir hechos de inferencias. Verificar versiones, contratos, nombres y
patrones antes de generar código.

#### Cambio mínimo

Definir qué archivos deben cambiar y cuáles no.

#### Implementar

Seguir convenciones existentes y no introducir dependencias innecesarias.

#### Validar

Ejecutar las comprobaciones documentadas por el componente: build, lint, tests,
análisis estático o validación de artefactos según corresponda.

#### Revisar cambios colaterales

Confirmar que no se modificaron versiones, paquetes, contratos, estilos o
archivos no relacionados accidentalmente.

#### Informar

Resumir qué cambió, qué se validó y cualquier limitación real.


## 6. Symlink Integrity and Multi-Agent Portability

- **Canonical Source**: Keep reusable artifacts in the documented canonical location for the project; use symlinks or other references for agent-specific paths when supported and portable.
- **Update Safety**: Whenever a file is renamed, moved, or its suffix changes, verify and update all symlinks that target it before considering the change complete.
- **New Artifact Linking**: For artifacts intended to be shared across agents, add the references supported by the target repositories and operating systems.
- **Completion Gate**: Verify affected references after moving files. Do not require symlinks where they are unsupported or would make a BCCR checkout non-portable.


### 6.1 Reglas de integridad de referencias

> Se conserva la intención de una fuente canónica y referencias multiagente, pero no se asume que
> `ai-specs` sea la ubicación canónica para todo.

Al mover, renombrar o cambiar la extensión de un artefacto:

1. Buscar referencias internas.
2. Verificar symlinks existentes.
3. Actualizar rutas relativas.
4. Revisar documentación que lo enlace.
5. Verificar que la solución siga funcionando en los sistemas operativos
   soportados por el equipo.

**Cambio incompleto**

```text
Se renombró ai-specs/skills/x/SKILL.md
pero .claude/... o .cursor/... siguen apuntando al nombre anterior.
```

**Cambio correcto**

```text
Se renombró el artefacto, se actualizaron las referencias afectadas y se
verificó que no quedaron targets obsoletos.
```

### 6.2 Evitar duplicación de fuentes canónicas

No crear copias independientes del mismo estándar para distintos agentes si
pueden referenciar una fuente común.

Si una herramienta no soporta symlinks o el repositorio debe funcionar en un
entorno donde estos no sean portables, usar el mecanismo documentado para esa
herramienta y dejar clara la relación con la fuente original.

## 7. Mandatory OpenSpec Artifact Updates for Post-Apply Changes

When working on an active OpenSpec change after `opsx:apply` and before
`opsx:archive`, update the affected change artifacts before implementing a new
requirement. This workflow does not apply to ordinary work that is not part of
an active OpenSpec change.

For an active change:

1. Update the affected requirements, scenarios, and tasks.
2. Regenerate artifacts when required by the workflow.
3. Implement and verify against the updated artifacts before archiving.

### 7.1 Principio de fuente de verdad durante un cambio OpenSpec

> Entre `opsx:apply` y `opsx:archive`, una petición adicional que cambie requisitos no debe
> implementarse directamente como un parche informal.

La secuencia obligatoria es:

```text
Nuevo requerimiento/cambio
        ↓
Actualizar requisitos/escenarios
        ↓
Actualizar tasks.md
        ↓
Regenerar artefactos si corresponde
        ↓
Implementar
        ↓
Verificar contra artefactos actualizados
        ↓
Archivar cuando el cambio esté completo
```

No etiquetar automáticamente una nueva necesidad funcional como “bugfix” si en
realidad modifica el diseño o alcance del cambio activo.

### 7.2 Cuándo NO aplica esta regla

No aplicar el workflow OpenSpec a:

- trabajo ordinario fuera de un cambio OpenSpec activo;
- consultas o análisis que no modifican el proyecto;
- tareas puramente operativas que la documentación del proyecto trate por otro
  flujo.

No crear artefactos OpenSpec innecesarios únicamente porque este documento los
menciona.

### 7.3 Ejemplo

**Evitar**

```text
Después de apply aparece un nuevo requisito.
Se modifica directamente el código.
tasks.md y escenarios permanecen desactualizados.
```

**Recomendado**

```text
Se actualizan primero los requisitos, escenarios y tareas afectados.
Después se implementa y se valida contra la especificación revisada.
```

## 8. Reglas transversales para cambios de dependencias y configuración

> Las tareas enfocadas no deben introducir cambios de infraestructura accidental.

- No actualizar paquetes como efecto secundario de una funcionalidad.
- No cambiar versiones de runtime/framework sin una tarea explícita.
- No introducir una dependencia nueva si la plataforma ya ofrece una solución
  equivalente.
- Antes de agregar una dependencia, justificar necesidad, alcance, seguridad,
  mantenimiento y compatibilidad.
- No cambiar configuración de agentes, IDEs o herramientas del usuario salvo
  que la tarea lo requiera explícitamente.
- No incluir secretos, tokens o credenciales en estándares, ejemplos o
  configuración versionada.

## 9. Manejo de hallazgos fuera de alcance

Si durante una tarea se detecta un problema no relacionado:

1. No modificarlo automáticamente.
2. Determinar si bloquea la tarea actual.
3. Si bloquea, realizar el cambio mínimo necesario y documentar la razón.
4. Si no bloquea, reportarlo como hallazgo separado.

Ejemplo:

```text
Tarea: agregar validación a un endpoint.
Hallazgo: otro módulo usa un paquete obsoleto.

Acción correcta: no actualizar el paquete dentro del mismo cambio salvo que
impida implementar o validar la tarea.
```

## 10. Criterio de finalización

Una tarea técnica no se considera completa únicamente porque “el código está
escrito”.

Según el tipo de cambio, verificar:

- comportamiento solicitado implementado;
- arquitectura y contratos preservados;
- tests pertinentes agregados/actualizados;
- pruebas/build/lint aplicables ejecutados;
- warnings relevantes revisados;
- documentación/artefactos actualizados cuando cambie el comportamiento o
  proceso;
- referencias y symlinks válidos si se movieron artefactos;
- ningún secreto o dato sensible añadido;
- ningún cambio colateral no relacionado;
- OpenSpec actualizado cuando corresponda.

## 11. Recomendaciones de `base-standards_1.md` que NO se trasladan literalmente

- **English Only:** VES usa principalmente español; se conserva la consistencia
  lingüística, no el inglés obligatorio.
- **TDD obligatorio para toda funcionalidad:** se conserva como práctica
  recomendada cuando aporta valor; las pruebas siguen siendo obligatorias
  según riesgo, comportamiento y estándar del componente.
- **Todo el código completamente tipado:** se aplica naturalmente a C#; no se
  usa para imponer TypeScript a la UI JavaScript/JSX.
- **Nunca avanzar más de un paso:** se transforma en cambios pequeños,
  incrementales y verificables sin obligar a interrumpir un flujo coherente.
- **Claude Opus como producto obligatorio:** se sustituye por la categoría
  **Opus-equivalent**: el modelo de razonamiento más avanzado disponible y
  adecuado al stack para planificación, arquitectura y decisiones complejas.
- **Claude Sonnet como producto obligatorio:** se sustituye por la categoría
  **Sonnet-equivalent**: un modelo competente y eficiente para implementar
  planes, tareas y reglas ya claramente definidas.
- **Modificar `.claude/settings.json` automáticamente:** no se usa como mecanismo
  transversal. La selección de nivel/modelo debe realizarse mediante el flujo
  normal de la herramienta, salvo autorización explícita.
- **Cambio automático Opus/Sonnet por nombre comercial:** no se exige una marca;
  sí se exige respetar la separación lógica entre planificación avanzada y
  ejecución eficiente cuando la plataforma lo permita.
- **`ai-specs` como ubicación canónica para cualquier personalización:** solo
  cuando la documentación del proyecto lo establezca.
- **Symlinks obligatorios en todos los agentes:** se usan cuando sean
  compatibles y estén documentados; la portabilidad prevalece.

## 12. Checklist base para desarrolladores y agentes

### Antes de cambiar

- [ ] Clasifiqué la tarea como planificación compleja o ejecución con reglas
      claras.
- [ ] Seleccioné un modelo Opus-equivalent cuando hay decisiones
      arquitectónicas, ambigüedad o alto riesgo.
- [ ] Si el trabajo ya está suficientemente especificado, puedo usar un modelo
      Sonnet-equivalent para la implementación.
- [ ] Identifiqué el componente real afectado.
- [ ] Leí su estándar específico.
- [ ] Revisé README/AGENT/SPEC o documentación relevante.
- [ ] Distinguí hechos, inferencias y propuestas.
- [ ] Confirmé el alcance solicitado.
- [ ] Verifiqué si existe un cambio OpenSpec activo aplicable.

### Durante el cambio

- [ ] Si el ejecutor encuentra una decisión no prevista, escala al modelo
      Opus-equivalent antes de rediseñar por su cuenta.
- [ ] Mantengo el cambio pequeño y enfocado.
- [ ] Sigo la arquitectura existente.
- [ ] Uso nombres claros.
- [ ] No introduzco una tecnología o patrón por preferencia personal.
- [ ] No actualizo paquetes ni runtime sin necesidad explícita.
- [ ] No modifico contratos públicos por estilo.
- [ ] No duplico lógica sin evaluar primero si representa la misma regla.
- [ ] No creo abstracciones especulativas.
- [ ] Mantengo el idioma y naming del componente.
- [ ] No introduzco secretos.

### Validación

- [ ] Agregué/actualicé pruebas pertinentes al comportamiento.
- [ ] Ejecuté build/lint/tests según el componente.
- [ ] Revisé warnings y análisis estático aplicables.
- [ ] Verifiqué que no haya cambios colaterales.
- [ ] Actualicé documentación si el comportamiento/proceso cambió.
- [ ] Verifiqué symlinks/referencias si moví artefactos.
- [ ] Actualicé OpenSpec antes del código si correspondía.

### Antes de cerrar

- [ ] El resultado satisface el requerimiento original.
- [ ] Puedo explicar qué cambió y por qué.
- [ ] Puedo indicar qué validaciones se ejecutaron.
- [ ] Cualquier limitación o hallazgo fuera de alcance quedó claramente
      documentado.

