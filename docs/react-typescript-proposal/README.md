# Modernización del módulo de documentos STI — VES

> **Estado:** propuesta de migración incremental, no funcionalidad implementada ni contrato desplegado. **Alcance exclusivo:** documentos y tipos documentales. **Comentarios de usuarios excluidos completamente.**

## 1. Propósito

Migrar gradualmente la consulta y mantenimiento del módulo `STI_Documentos` de ASP.NET Web Forms a la arquitectura vigente de VES, preservando el comportamiento documental existente y validándolo por etapas, sin reemplazo masivo de VES ni interrupción inmediata de Web Forms.

Se contemplan **siete capacidades**: listado y filtros, detalle, alta de metadatos, edición de metadatos, carga de archivo, descarga de archivo y eliminación condicionada a reglas verificadas. El catálogo de tipos documentales sirve de apoyo a esos flujos.

## 2. Arquitectura de referencia

```text
Navegador
   ↓
Bccr.Supen.Ves.UI — React 19, JavaScript/JSX, Vite
   ↓ HTTP/HTTPS
Bccr.Supen.Ves.Api — .NET 8 / ASP.NET Core
   API Controller → SI.SG → IRequestsHandler
   ↓ HTTP/HTTPS
Bccr.Supen.Ves.Mapi — .NET 8 / ASP.NET Core
   MAPI Controller → BW → DA (EF Core 8 y procedimientos existentes)
   ↓
SQL Server — objetos administrados por DBA
```

Los tres proyectos se despliegan por separado. La interfaz solo consume **API**, nunca MAPI, servicios SOAP o SQL directamente. Los modelos `Abstracciones` de API y MAPI son propios de cada solución, no una única biblioteca compartida.

**Tecnologías:** no imponer TypeScript por el título histórico `03-frontend-react-typescript.md`, ni introducir Node/Express, Prisma, PostgreSQL, un ORM alternativo o migraciones EF. La UI actual está documentada en React/JSX. Cualquier proyecto nuevo con TypeScript requiere una decisión explícita independiente.

## 3. Alcance y exclusiones

| Dentro de la migración | Fuera de la migración |
|---|---|
| Catálogo de tipos, listado, filtros y detalle de documentos | Comentarios, respuestas, hilos, estados e historial de comentarios |
| Creación y edición de metadatos STI | Entidades, DTOs, rutas, pantallas, tablas o pruebas de comentarios |
| Carga y descarga de documentos | Rediseño general de VES, migración masiva de BD u otros módulos |
| Eliminación controlada conforme a política confirmada | Autenticación y autorización reales o integración con SeguridadVES |
| Pruebas con identidad y permisos simulados en entorno aislado | Sustituir la seguridad institucional existente con un mock |
| Convivencia y piloto reversible con Web Forms | Despliegue productivo sin controles de seguridad aprobados |

**La exclusión de comentarios es definitiva para este cambio.** El archivo `06-manejo-y-mantenimiento-de-comentarios.md` permanece únicamente como constancia de esta decisión, no como especificación de una funcionalidad.

## 4. Modelo de datos y contrato

La referencia documentada para `STI_Documentos` incluye `CodigoDocumento`, `CodigoTipoDoc` (nullable), `Titulo` (100), `Resumen` (255), `NombreArchivo` (30), `Publico` (`S/N`), `FechaPublicacion`, `UsuarioPublica`, `FechaCaduca` (nullable) y el binario `Documento`. `STI_TiposDocumento` proporciona el catálogo de tipos y su estado `A/I`. Son referencias documentales que deben validarse con los objetos efectivamente desplegados.

El contrato objetivo está en [`api-spec.yml`](./api-spec.yml). Documenta **8 operaciones** distribuidas en cuatro rutas lógicas: `/api/document-types`, `/api/documents`, `/api/documents/{id}` y `/api/documents/{id}/file`. **Las rutas, el versionado y los DTOs definitivos deben contrastarse con los controllers existentes** antes de comenzar a implementarlas. No se incluyen endpoints `/comments`.

No asumir que `DRAFT`, `PUBLISHED`, `INACTIVE`, `status` o un esquema de historial de comentarios forman parte del modelo actual: `Publico`, fechas y datos existentes requieren reglas aprobadas. El listado paginado y la búsqueda textual son propuestas que requieren comprobar soporte o definir su implementación sin cambiar indebidamente el comportamiento del SQL legado.

## 5. Archivos y reglas de validación propuestas

- Tamaño máximo contractual **25 MiB (26.214.400 bytes)**; comprobar compatibilidad de API, MAPI, infraestructura y SQL antes del piloto.
- Extensiones iniciales: `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.pptx`; validar extensión y MIME en backend y considerar firma/contenido según viabilidad.
- Carga con `multipart/form-data`, archivo obligatorio; descarga binaria con MIME y nombre seguro.
- Errores previstos: `400` solicitud inválida/vacía, `413` tamaño excesivo, `415` tipo no permitido, además de los códigos de operación pertinentes.
- `NombreArchivo` se documenta con **30 caracteres en SQL**; no asumir que la longitud mayor del parámetro de un procedimiento amplía la columna.

Riesgos pendientes de análisis con DBA y pruebas controladas: `GuardarDocumento` (concurrencia de identificadores, modificaciones parciales y NULL), `SubirArchivo` (casos especiales), disponibilidad real de `BajarArchivo` y `EliminarDocumento` (borrado físico). No ejecutar cambios de esquema o procedimientos desde la aplicación.

## 6. Seguridad simulada

Solo durante desarrollo y pruebas **aisladas**, un proveedor mock de identidad/permisos permitirá probar casos con identidad válida, sin identidad (`401`) y sin permiso funcional (`403`). La comprobación debe ocurrir en backend antes de ejecutar la operación. El mock **no constituye seguridad productiva**; no implementar login, tokens, JWT, proveedor de identidad ni integración efectiva con `SeguridadVES` como parte de esta entrega. Tampoco desactivar protecciones institucionales existentes.

## 7. Ruta de trabajo y criterios de aceptación

Seguir [`05-plan-paso-a-paso.md`](./05-plan-paso-a-paso.md) y el flujo OpenSpec vigente: inspección del legado y acuerdos de negocio → especificación y contrato → MAPI/BW/DA → API/SI.SG → UI → pruebas y validación de convivencia → piloto controlado. Respetar la configuración `config.yaml`, los estándares VES, las ramas de cada proyecto y los pasos de pruebas obligatorios. No afirmar pruebas ejecutadas si solo fueron planificadas.

Se considerará aprobada la migración cuando las capacidades acordadas tengan trazabilidad de UI a SQL, se verifiquen límites de archivos, permisos simulados, errores y efecto real de escritura, y exista un plan de retorno. Se requiere resolver o registrar expresamente riesgos de procedimientos y seguridad antes de cualquier liberación. No crear tareas ni código sobre comentarios.

## 8. Documentación para lectura

1. [`01-alcance-y-principios.md`](./01-alcance-y-principios.md) — límites y principios.
2. [`02-arquitectura-objetivo.md`](./02-arquitectura-objetivo.md) — componentes, flujos y transición.
3. [`03-frontend-react-typescript.md`](./03-frontend-react-typescript.md) — interfaz VES React/JSX; nombre histórico conservado.
4. [`04-backend-api.md`](./04-backend-api.md) — operaciones candidatas, validación y persistencia.
5. [`05-plan-paso-a-paso.md`](./05-plan-paso-a-paso.md) — fases, pruebas y decisiones de avance.
6. [`06-manejo-y-mantenimiento-de-comentarios.md`](./06-manejo-y-mantenimiento-de-comentarios.md) — **exclusión formal de comentarios**, no implementación.
7. [`api-spec.yml`](./api-spec.yml) — contrato OpenAPI objetivo exclusivo de documentos.

Fuentes complementarias: `data-model.md`, `base-standards.md`, `frontend-standards.md`, `backend-standards.md`, `documentation-standards.md`, `development_guide.md`, `openspec-tasks-mandatory-steps.md` y `openspec/config.yaml`, además de la documentación específica de UI, API y MAPI.
