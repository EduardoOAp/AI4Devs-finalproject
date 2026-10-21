> Detalla en esta sección los prompts principales utilizados durante la creación del proyecto, que justifiquen el uso de asistentes de código en todas las fases del ciclo de vida del desarrollo. Esperamos un máximo de 3 por sección, principalmente los de creación inicial o  los de corrección o adición de funcionalidades que consideres más relevantes.
Puedes añadir adicionalmente la conversación completa como link o archivo adjunto si así lo consideras

**Proyecto**: Modernización y evolución del Sistema de Ventanilla Electrónica de Servicios (VES).

**Metodología**: Desarrollo asistido por inteligencia artificial, análisis de código existente y desarrollo guiado por especificaciones (SDD), utilizando asistentes de programación y OpenSpec.

La documentación refleja dos contextos tecnológicos que deben distinguirse:
- Sistema VES legado: ASP.NET Web Forms, VB.NET, servicios SOAP/ASMX, ADO.NET y SQL Server.
- Arquitectura VES moderna: React 19 con JavaScript/JSX y Vite; API y MAPI independientes en .NET 8; EF Core 8, procedimientos almacenados y SQL Server.

## Índice

1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 1. Descripción general del producto

**Prompt 1: Análisis inicial del sistema existente**
"Actúa como un arquitecto de software sénior especializado en análisis de sistemas heredados y modernización de aplicaciones empresariales.

Analiza integralmente el repositorio VES, incluyendo VES.sln, PortalVES, VESDATOS, PortalVES.Autenticacion, archivos de configuración, servicios y componentes relacionados.

Identifica el propósito del sistema, sus principales módulos, usuarios, funcionalidades, tecnologías, integraciones y dependencias.

Elabora una descripción general del producto que permita comprender su funcionamiento sin necesidad de conocer previamente el código fuente.

Distingue entre funcionalidades verificadas, comportamientos inferidos y funcionalidades propuestas. No inventes requisitos ni modifiques código.

Genera un documento Markdown que sirva como base para el README del proyecto."

**Prompt 2: Identificación de funcionalidades del módulo documental**
"Analiza el módulo de documentos del sistema VES, particularmente ListarDocumentos.aspx, EditorDocumentos.aspx, BajarDocumento.aspx, sus archivos .vb y los servicios relacionados.

Identifica los procesos de consulta, creación, modificación, carga, descarga y eliminación de documentos, así como sus reglas de negocio, validaciones y dependencias.

Describe los flujos desde la perspectiva del usuario y relaciona cada funcionalidad con los componentes técnicos que la implementan.

Genera una descripción funcional del módulo, indicando las evidencias encontradas y las funcionalidades cuya existencia no pueda confirmarse."

**Prompt 3: Consolidación de la documentación del producto**
"Revisa los documentos generados durante el análisis de VES y consolida la información en READMEVES.md.

Organiza el documento con descripción del producto, propósito, funcionalidades, arquitectura, componentes, modelo de datos, seguridad, tecnologías y oportunidades de modernización.

Elimina duplicaciones, resuelve contradicciones únicamente cuando exista evidencia y conserva las referencias al código fuente.

Diferencia claramente el sistema legado de la arquitectura moderna y evita presentar propuestas de migración como funcionalidades ya implementadas."
---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**

**Prompt 1: Reconstrucción de la arquitectura heredada**

"Actúa como arquitecto de soluciones especializado en ASP.NET Web Forms, VB.NET y sistemas empresariales.

Examina la estructura de VES y reconstruye su arquitectura real, identificando presentación, code-behind, adaptadores, servicios SOAP, autenticación y acceso a datos.

Analiza cómo interactúan PortalVES, VESDATOS, PortalVES.Autenticacion y los demás servicios institucionales.

Genera un diagrama Mermaid que represente el flujo de solicitudes desde el navegador hasta la base de datos, acompañado de una explicación de cada componente.

No introduzcas capas que no existan en el código y señala las relaciones que no puedan verificarse."

**Prompt 2: Análisis de la arquitectura moderna**

"Analiza los proyectos Bccr.Supen.Ves.UI, Bccr.Supen.Ves.Api y Bccr.Supen.Ves.Mapi.

Reconstruye su arquitectura de extremo a extremo, identificando tecnologías, responsabilidades, contratos, protocolos y dependencias.

Verifica especialmente el flujo UI → API → MAPI → SQL Server, la orquestación mediante SI.SG e IRequestsHandler, y las capas Controller → BW → DA del MAPI.

Genera diagramas Mermaid de componentes y secuencia. Distingue los servicios desplegables de las capas internas y no asumas que los tres proyectos forman una única aplicación."

**Prompt 3: Comparación arquitectónica y alcance de modernización**
Compara la arquitectura del sistema VES legado con la arquitectura moderna documentada.

Identifica qué responsabilidades del módulo documental deben conservarse, qué componentes necesitan adaptación y cuáles pueden reutilizarse.

Propón un mapa de correspondencia entre páginas ASPX, clases VB.NET, servicios SOAP y componentes modernos.

No propongas reemplazar toda la plataforma ni migrar módulos ajenos al alcance. Documenta riesgos, dependencias y decisiones pendientes."

### **2.2. Descripción de componentes principales:**

**Prompt 1: Análisis de frontend**

"Examina el frontend de VES y documenta sus tecnologías, estructura, navegación, componentes reutilizables, manejo de estado, validaciones y comunicación con servicios.

Para el sistema legado, analiza páginas ASPX, controles Web Forms, Master Pages y code-behind.

Para la interfaz moderna, identifica React, JavaScript/JSX, Vite, Material UI, React Router y las convenciones realmente utilizadas.

Genera frontend-guidelines.md, diferenciando las características observadas de las recomendaciones de modernización."

**Prompt 2: Análisis de backend**
"Analiza los componentes backend de VES e identifica controladores, servicios, contratos, adaptadores, lógica de negocio y mecanismos de persistencia.

Documenta por separado la arquitectura SOAP/VB.NET heredada y las soluciones .NET 8 de API y MAPI.

Explica las responsabilidades de SI.SG, IRequestsHandler, BW, DA y Abstracciones.

Genera backend-guidelines.md, incluyendo dependencias, patrones de comunicación, manejo de errores y puntos de extensión para el módulo documental."

**Prompt 3: Estandarización de componentes**
"Utiliza el análisis técnico y los documentos frontend-standards.md, backend-standards.md y base-standards.md para definir las reglas de desarrollo de VES.

Conserva las convenciones existentes y adapta las buenas prácticas de SOLID, DRY, validación, separación de responsabilidades, pruebas y mantenibilidad."

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

**Prompt 1: Inventario del repositorio**
"Recorre recursivamente el repositorio VES y genera un inventario de sus proyectos, directorios y archivos principales.

Para cada carpeta relevante, identifica su propósito, tecnologías, responsabilidades y relación con otros componentes.

Diferencia código fuente, configuración, documentación, pruebas, infraestructura y artefactos generados.

Presenta un árbol de directorios acompañado de explicaciones que permitan a un desarrollador nuevo comprender la organización del proyecto." 

**Prompt 2: Documentación de los flujos del código**
"AAnaliza el flujo de ejecución del módulo documental desde las pantallas ListarDocumentos.aspx y EditorDocumentos.aspx hasta los servicios y procedimientos almacenados.

Identifica eventos de interfaz, métodos de code-behind, clases adaptadoras, servicios SOAP y operaciones de base de datos.

Genera design-flow.md con diagramas de secuencia, rutas de archivos y explicación paso a paso.

Documenta también la correspondencia funcional con la arquitectura moderna, sin asumir que la migración ya fue realizada."

**Prompt 3: Guía para nuevos desarrolladores y asistentes IA**
"A partir del inventario y la arquitectura verificada, genera development_guide.md.

Incluye prerrequisitos, herramientas, versiones, configuración, ejecución de UI, API y MAPI, pruebas, solución de problemas y convenciones de desarrollo.

Indica qué documentación debe consultar un desarrollador o asistente IA antes de modificar cada componente.

No incluyas credenciales, secretos ni comandos incompatibles con las tecnologías reales."

### **2.4. Infraestructura y despliegue**

**Prompt 1: Identificación de dependencias de infraestructura**
"Analiza los archivos de configuración, guías de despliegue y proyectos de VES.

Identifica servidores web, runtimes, servicios, conexiones a bases de datos, dependencias de red, configuración por ambiente y mecanismos de publicación.

Documenta únicamente la infraestructura respaldada por archivos o evidencias disponibles.

Distingue la configuración local, de pruebas y de producción cuando exista información suficiente."

**Prompt 2: Definición del procedimiento de despliegue**
"Elabora una guía de despliegue para los tres componentes modernos de VES.

Considera React/Vite para UI, ASP.NET Core 8 para API y MAPI, SQL Server, configuración por ambiente y dependencias institucionales.

Explica el orden de inicialización y verificación de los servicios, el manejo seguro de configuración y las comprobaciones posteriores al despliegue.

Conserva los mecanismos institucionales existentes y no inventes pipelines o servicios de infraestructura."

**Prompt 3: Validación de ambientes**
"Diseña una lista de verificación para validar los ambientes de desarrollo y pruebas de VES.

Incluye disponibilidad de dependencias, configuración, conectividad UI–API–MAPI, acceso autorizado a SQL Server, health checks, pruebas de humo y registro de incidencias.

Separa verificaciones automatizables de aquellas que requieren intervención institucional.

Genera instrucciones reproducibles y criterios objetivos de aceptación."

### **2.5. Seguridad**

**Prompt 1: Análisis de la seguridad heredada**
"Analiza el mecanismo de autenticación y autorización del sistema VES original.

Revisa Forms Authentication, sesiones, menús dinámicos, perfiles, permisos y servicios SeguridadVES.

Identifica cómo se determina el acceso a módulos y operaciones documentales.

Genera un mapa funcional de identidad y autorización, señalando dependencias y comportamientos no confirmados."

**Prompt 2: Definición del alcance de seguridad para el prototipo**
"Para el proyecto de modernización del módulo documental, considera que la autenticación y autorización reales están fuera de alcance.

Diseña un mock de seguridad que simule usuarios, identidad, perfiles, permisos y respuestas de autorización.

Permite probar escenarios de usuario autorizado, usuario sin permisos y sesión no válida, sin conectarse a SeguridadVES ni implementar un sistema de autenticación productivo.

Mantén interfaces desacopladas para permitir una integración futura, pero no implementes dicha integración."

**Prompt 3: Revisión de seguridad y consistencia documental**
"Revisa los documentos de arquitectura, API, frontend, backend y pruebas para verificar que respeten el alcance de seguridad definido.

Identifica cualquier referencia que implique implementar autenticación real, integración efectiva con SeguridadVES o administración productiva de permisos.

Ajusta esas referencias para distinguir el comportamiento simulado del comportamiento institucional existente.

Conserva los controles de validación de entradas y manejo seguro de archivos; no presentes el mock como un mecanismo de seguridad apto para producción."

### **2.6. Tests**

**Prompt 1: Diseño de estrategia de pruebas**
"Actúa como ingeniero QA especializado en React, ASP.NET Core y sistemas empresariales.

Analiza las funcionalidades documentales y define una estrategia de pruebas unitarias, de integración y de comportamiento.

Utiliza las herramientas existentes en cada componente, incluyendo Vitest y React Testing Library en frontend, y los frameworks de pruebas .NET ya configurados.

Define escenarios positivos, negativos, límites, manejo de errores y pruebas de regresión.

No introduzcas herramientas adicionales sin necesidad."

**Prompt 2: Generación de pruebas del módulo documental**
"Genera los casos de prueba necesarios para las operaciones de listar tipos, listar documentos, consultar metadatos, guardar, subir, descargar y eliminar documentos.

Incluye validación de formatos .doc, .docx, .pdf, .ppt y .pptx, límite de 10 MB, errores de servicios y permisos simulados.

Utiliza mocks para dependencias externas y evita modificar datos productivos.

Relaciona cada prueba con su requisito y criterio de aceptación"

**Prompt 3: Revisión y ejecución de pruebas**
"Ejecuta las pruebas disponibles para los componentes afectados, utilizando los comandos definidos por el repositorio.

Analiza los resultados, identifica fallos, determina si son regresiones o problemas preexistentes y propone correcciones mínimas.

No declares pruebas exitosas si no pudieron ejecutarse.

Genera un informe con pruebas ejecutadas, resultados, cobertura funcional, limitaciones y evidencias."

---

### 3. Modelo de Datos

**Prompt 1: Ingeniería inversa del modelo de datos**
"Actúa como arquitecto de datos especializado en SQL Server y sistemas empresariales heredados.

Analiza los archivos Documentos.vb, Service.vb, los servicios de VESDATOS, las consultas SQL y los procedimientos almacenados utilizados por VES.

Identifica entidades, atributos, claves, relaciones, operaciones CRUD y reglas de persistencia.

Reconstruye el modelo conceptual de las tablas VES.STI_Documentos y VES.STI_TiposDocumento.

Genera DataBase_flow.md y dataModel.md, incluyendo un diagrama entidad-relación Mermaid.

Diferencia las estructuras verificadas de las inferidas y no supongas que los DTOs representan necesariamente tablas físicas."

**Prompt 2: Análisis de procedimientos almacenados**
"Analiza detalladamente los procedimientos ListarTiposDocumentos, ListarDocumentos, ObtenerDocumento, GuardarDocumento, SubirArchivo, BajarArchivo y EliminarDocumento.

Para cada procedimiento documenta parámetros, tipos, operaciones SQL, valores de retorno, reglas de negocio, validaciones, tratamiento de valores nulos y posibles errores.

Identifica dependencias entre procedimientos y determina cómo se utilizan desde el módulo documental.

Señala posibles riesgos de concurrencia, integridad, seguridad y rendimiento sin modificar la lógica existente.

Genera una matriz de trazabilidad entre funcionalidades, procedimientos y entidades."

**Prompt 3: Consolidación del modelo de datos moderno**
"Revisa data-model.md y contrástalo con los DTOs de Bccr.Supen.Ves.Abstracciones, la documentación de MAPI y las definiciones SQL disponibles.

Documenta las entidades, contratos de transferencia, relaciones conceptuales y restricciones del módulo documental.

Distingue el esquema físico de SQL Server de los modelos de transporte y las clases de EF Core.

Mantén los procedimientos almacenados existentes y no generes migraciones de EF Core ni modificaciones de tablas.

Registra explícitamente las relaciones y validaciones que requieren confirmación con el DBA."

**Prompt 4. Corrección de inconsistencias del modelo documental**
"Revisa las reglas de persistencia documentadas para el módulo documental y verifica su consistencia con los procedimientos SQL de referencia.

Considera el tratamiento de FechaCaduca = NULL como vigencia indefinida, el filtro de todos los tipos mediante -1, la generación de identificadores y los valores especiales devueltos por los procedimientos.

No sustituyas comportamientos heredados que deban conservarse sin aprobación.

Identifica los riesgos de dichas decisiones, documenta los casos límite y define las pruebas necesarias para verificar su comportamiento."

---

### 4. Especificación de la API

**Prompt 1: Ingeniería inversa de endpoints y contratos**
"Actúa como arquitecto de APIs REST especializado en ASP.NET Core 8.

Analiza los controladores de Bccr.Supen.Ves.Api, los DTOs de Abstracciones, las implementaciones SI.SG y la comunicación con MAPI.

Identifica las operaciones públicas, métodos HTTP, rutas, parámetros, modelos de solicitud y respuesta, códigos de estado, versionamiento y mecanismos de autorización.

Genera una especificación OpenAPI 3.0.3 en api-spec.yml.

No inventes endpoints ni presentes como implementadas las operaciones que solo se encuentren propuestas."

**Prompt 2: Especificación de las operaciones documentales**
"Utilizando exercise.md, el modelo de datos y los procedimientos almacenados del módulo documental, define los contratos REST necesarios para listar tipos, listar documentos, consultar metadatos, guardar, subir archivos, descargar y eliminar documentos.

Respeta el patrón API Controller → SI.SG → IRequestsHandler → MAPI y la separación de responsabilidades existente.

Define entradas, salidas, validaciones, códigos HTTP y ejemplos de respuesta.

Considera multipart/form-data para archivos, límites de tamaño, formatos permitidos y errores de negocio.

Marca las rutas y estructuras propuestas como pendientes de verificación hasta que se contrasten con la implementación real"

**Prompt 3: Corrección y validación de OpenAPI**
"Revisa exhaustivamente api-spec.yml contra los controladores, contratos y convenciones de VES.

Identifica inconsistencias en rutas, nombres, parámetros obligatorios, tipos, propiedades requeridas, respuestas HTTP, seguridad y referencias $ref.

Elimina cualquier contenido heredado de aplicaciones ajenas a VES.

Corrige únicamente los problemas comprobados y valida la estructura OpenAPI.

Entrega un resumen de cambios y una lista de contratos que todavía requieren confirmación técnica."

**Prompt 3: Corrección y validación de OpenAPI**
"Comprueba que los contratos definidos para la API pública y el MAPI sean compatibles.

Verifica métodos HTTP, versionamiento, parámetros, DTOs, serialización JSON, tratamiento de archivos, errores y respuestas.

Identifica cualquier conversión o transformación realizada por SI.SG.

Define pruebas de contrato para evitar incompatibilidades entre componentes y documenta las diferencias encontradas."

---

### 5. Historias de Usuario

**Prompt 1:Generación de historias a partir del sistema existente**
"Actúa como analista funcional y Product Owner especializado en modernización de software.

Utiliza el análisis del sistema VES, READMEVES.md, design-flow.md, dataModel.md y exercise.md para identificar las necesidades funcionales del módulo documental.

Transforma las funcionalidades observadas en historias de usuario con el formato:

"Como [rol], quiero [funcionalidad], para [beneficio]".

Incluye descripción, reglas de negocio, dependencias, prioridad sugerida y criterios de aceptación verificables.

No incorpores funcionalidades que no pertenezcan al alcance aprobado."

**Prompt 2: Refinamiento de historias con criterios de aceptación**
"Revisa las historias de usuario del módulo documental y conviértelas en requisitos verificables.

Utiliza escenarios Given–When–Then para describir condiciones iniciales, acciones y resultados esperados.

Incluye casos de éxito, datos inexistentes, errores de validación, archivos no permitidos, límites de tamaño y permisos simulados.

Relaciona cada historia con las operaciones de API y procedimientos almacenados correspondientes.

Identifica ambigüedades que deban resolverse antes de implementar."

**Prompt 3: Priorización y control del alcance**
"Organiza las historias de usuario del módulo documental en un backlog priorizado.

Considera las dependencias técnicas entre modelo de datos, MAPI, API y UI.

Prioriza las funcionalidades esenciales de consulta, administración y transferencia de documentos.

Excluye comentarios, respuestas a comentarios, flujos adicionales de publicación y seguridad productiva.

Divide historias demasiado grandes y establece una definición de terminado que incluya implementación, pruebas, documentación y revisión."

---

### 6. Tickets de Trabajo

**Prompt 1: Transformación de historias en tareas técnicas mediante OpenSpec**
"Actúa como líder técnico especializado en Spec-Driven Development (SDD) y OpenSpec.

Utiliza las historias de usuario aprobadas y los documentos técnicos de VES para generar los artefactos de planificación de un cambio.

Antes de crear tareas, revisa openspec/config.yaml, base-standards.md, backend-standards.md, frontend-standards.md, development_guide.md y openspec-tasks-mandatory-steps.md.

Genera la propuesta, las especificaciones, el diseño técnico y tasks.md, siguiendo el esquema OpenSpec configurado.

Divide el trabajo en tareas pequeñas, secuenciales y verificables, indicando componentes afectados, dependencias y criterios de finalización.

No generes código hasta que la planificación sea revisada y aprobada."

**Prompt 2: Descomposición técnica por componentes**
"A partir de las historias aprobadas, descompón cada funcionalidad documental en tickets técnicos para MAPI, API y UI.

Para MAPI considera controladores, lógica de negocio, acceso a datos y procedimientos almacenados.

Para API considera controladores, contratos, DTOs, SI.SG y comunicación con MAPI.

Para UI considera componentes React, formularios, consultas, validaciones y estados de carga y error.

Identifica pruebas unitarias, pruebas de integración y actualizaciones documentales.

Cada ticket debe incluir objetivo, archivos afectados, criterios de aceptación, dependencias y resultado esperado."

**Prompt 3: Aplicación de pasos obligatorios de desarrollo**
"Revisa tasks.md y comprueba que cumpla con openspec-tasks-mandatory-steps.md.

Incluye como primer paso la creación de la rama de trabajo correspondiente al repositorio afectado.

Incorpora revisión de pruebas existentes, implementación incremental, ejecución de pruebas unitarias, verificación del estado de la base de datos cuando corresponda, pruebas de endpoints con curl, pruebas E2E con Playwright MCP cuando apliquen y actualización de documentación.

Respeta las ramas base definidas por cada componente y no marques tareas como terminadas sin evidencias de ejecución.

Corrige el plan manteniendo la trazabilidad con las historias de usuario."

---

### 7. Pull Requests

**Prompt 1: Preparación del Pull Request**
"Actúa como ingeniero de software responsable de preparar un Pull Request para el proyecto VES.

Analiza los cambios de la rama actual respecto de su rama base mediante git diff, git status y el historial de commits.

Verifica que los cambios correspondan al ticket aprobado y que no incluyan archivos temporales, credenciales o modificaciones ajenas al alcance.

Genera un título y una descripción del PR con resumen funcional, cambios técnicos, componentes afectados, pruebas ejecutadas, riesgos y referencia al ticket o cambio OpenSpec.

No afirmes que las pruebas fueron ejecutadas si no existe evidencia."

**Prompt 2: Revisión automatizada del código**
"Actúa como revisor técnico sénior de un Pull Request de VES.

Analiza todos los archivos modificados y compáralos con las reglas del proyecto.

Revisa compatibilidad de contratos, separación de responsabilidades, validaciones, manejo de errores, calidad del código, pruebas, seguridad y documentación.

Clasifica los hallazgos por severidad, indicando archivo, ubicación, problema, impacto y corrección sugerida.

Evita recomendaciones genéricas que contradigan las convenciones existentes o amplíen innecesariamente el alcance."

**Prompt 3: Corrección de observaciones de Copilot o resisores automatizados**
"Revisa los comentarios de revisión generados por GitHub Copilot u otros revisores automatizados.

Trata los hallazgos como observaciones que deben verificarse, no como instrucciones automáticamente válidas.

Para cada comentario, comprueba si el problema continúa presente en el código actual.

Corrige únicamente los problemas válidos mediante cambios mínimos y conserva las decisiones arquitectónicas aprobadas.

Ejecuta las pruebas afectadas y documenta qué observaciones se corrigieron, cuáles se descartaron y por qué."

**Prompt 4: Validación final y cierre del Pull Request**
"Realiza una revisión final del Pull Request antes de solicitar su integración.

Verifica cumplimiento de historias de usuario, criterios de aceptación, tareas OpenSpec, estándares técnicos y documentación.

Comprueba que las pruebas requeridas hayan sido ejecutadas y registra sus resultados.

Revisa que el PR tenga como destino la rama correcta del componente.

Genera un informe final con estado de requisitos, evidencias de pruebas, hallazgos pendientes y recomendaciones para la revisión humana."

No apruebes ni fusiones automáticamente el PR sin autorización."

