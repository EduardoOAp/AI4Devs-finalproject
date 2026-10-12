# 01. Alcance y principios — migración del módulo de documentos STI

> **Estado:** propuesta de migración; no implica funcionalidad implementada. **Decisión de alcance:** únicamente documentos y tipos de documento; **comentarios excluidos completamente**. Referencias: `data-model.md`, `base-standards.md`, `backend-standards.md`, `frontend-standards.md`, `development_guide.md`, `config.yaml` y documentación específica de los tres componentes VES.

## 1. Propósito y problema

Modernizar incrementalmente la consulta y administración de documentos de STI, hoy documentada con Web Forms, eventos de servidor, `DataSet`, servicios SOAP y procedimientos SQL, preservando comportamiento contractual comprobado y coexistiendo con el módulo legado hasta completar la transición. No confundir los documentos STI con los archivos de transferencia de STI/SEC: son dominios distintos.

## 2. Alcance positivo

| Capacidad | Incluida | Resultado esperado |
|---|---|---|
| Listar documentos | Sí | Resultado filtrable, ordenado y paginado sin cargar binarios |
| Consultar tipos activos | Sí | Catálogo a partir de `STI_TiposDocumento` |
| Ver detalle | Sí | Metadatos de `STI_Documentos`, sin binario |
| Crear documento | Sí | Alta de metadatos tras validar campos e invariantes |
| Actualizar metadatos | Sí | Modificar un registro existente y confirmar que hubo efecto |
| Subir/reemplazar archivo | Sí | Adjuntar binario validado con semántica documentada |
| Descargar archivo | Sí | Transmitir contenido y nombre/tipo correctos |
| Eliminar documento | Sí, condicionado | Validación de reglas y permisos *simulados*; operación destructiva controlada |
| Coexistencia y piloto | Sí | Mantener reversibilidad y pruebas de regresión |

La publicación, caducidad y visibilidad `Publico` se conservan como atributos del documento; su significado funcional exacto y transiciones deben contrastarse con código legado y usuarios antes de imponer estados o reglas nuevos. `Publico` es `S/N`, no un campo `status` de un modelo ajeno.

## 3. Fuera de alcance

- **Comentarios, respuestas, hilos, menciones, estados o historial de comentarios**: ninguna pantalla, ruta, entidad, tabla, procedimiento, DTO, prueba funcional ni integración nueva para comentarios.
- Autenticación/autorización productiva, login, JWT, integración GAUDI/OpenID o `SeguridadVES`: el prototipo funcional emplea **mock de identidad y permisos**, no seguridad efectiva.
- Sustituir Web Forms de otros módulos, migrar STI/SEC completo, reescribir SOAP globalmente o cambiar toda la infraestructura VES.
- Cambiar de forma masiva React/JSX a TypeScript, introducir ORM nuevo, migraciones EF o cambiar el almacenamiento sin decisión explícita y DBA.
- Desarrollar un sistema general de versionado, workflows editoriales, auditoría integral o rediseño visual corporativo sin especificación aprobada.

## 4. Frontera técnica y fuentes de verdad

La arquitectura de referencia confirmada en los estándares es `Bccr.Supen.Ves.UI` (React 19, JS/JSX, Vite) → `Bccr.Supen.Ves.Api` (.NET 8: Controller → SI.SG → IRequestsHandler) → `Bccr.Supen.Ves.Mapi` (.NET 8: MAPI Controller → BW → DA → SQL Server). Son **tres componentes desplegables separados**; `Abstracciones` en API y MAPI no es una biblioteca única compartida. La UI jamás accede a MAPI, SOAP ni SQL de manera directa.

Prioridad práctica: requerimiento de esta migración → artefactos OpenSpec activos aprobados → código/contratos y documentación del componente → estándares VES → propuesta de migración. Los documentos de diseño no sustituyen pruebas de la implementación.

## 5. Reglas de datos verificables y puntos por confirmar

Según `data-model.md` (basado en SQL de referencia, **sin acreditar despliegue**), `STI_Documentos` usa `CodigoDocumento`, `CodigoTipoDoc` anulable, `Titulo` (`varchar(100)`), `Resumen` (`varchar(255)`), `NombreArchivo` (`varchar(30)`), `Publico` (`S/N`), `FechaPublicacion`, `UsuarioPublica`, `FechaCaduca` opcional y binario `Documento` (`image`). `STI_TiposDocumento` tiene `CodigoTipoDoc`, `Nombre` y `Estado` (`A/I`). Las restricciones exactas de API deben concordar con las reglas reales y el esquema de la base destino.

Riesgos del SQL de referencia que requieren pruebas antes del uso: `GuardarDocumento` usa `MAX(CodigoDocumento)+1` (problemas con tabla vacía y concurrencia), su `UPDATE` depende de comparaciones simultáneas y `NULL`; `SubirArchivo` trata el código `1` de modo especial y puede eliminar filas cuando el nombre queda nulo; `NombreArchivo` admite 30 caracteres aunque el parámetro del SP admite 50; `EliminarDocumento` borra físicamente; `BajarArchivo` requiere verificación de firma/disponibilidad. **No ejecutar correcciones de esquema ni SP desde la aplicación:** registrar hallazgos y coordinar con DBA.

## 6. Seguridad simulada: límites explícitos

- La identidad de pruebas y permisos se configuran solo en entorno aislado; el backend (BW o límite correspondiente) decide cada operación, nunca solamente la UI.
- Escenarios: identidad simulada presente con permiso, presente sin permiso, ausente; los `401/403` son respuestas **simuladas**.
- No incluir secretos, credenciales reales, certificados ni configuraciones de bypass en el repositorio o despliegues reales.
- No presentar el mock como control de acceso apto para producción. La liberación a producción exige diseño de seguridad aparte.

## 7. Principios de ejecución

1. Preservar los contratos legados observados mientras se migra; cambios incompatibles solo con aprobación explícita.
2. Verticales pequeños y trazables con criterios de aceptación, pruebas y posibilidad de retroceder.
3. Separar DTO HTTP, reglas BW, integración SI.SG y persistencia DA; no replicar `DataSet` en frontend.
4. Validaciones de UI para experiencia y validaciones obligatorias en backend.
5. Evitar operaciones destructivas y pruebas mutantes sobre datos productivos.
6. Documentar hallazgos comprobados frente a decisiones pendientes; no confundir propuesta con funcionamiento actual.
7. No añadir dependencias o abstracciones por preferencia genérica; seguir convenciones existentes.

## 8. Aceptación del alcance

Se considera completo cuando las siete capacidades aprobadas funcionan desde UI hacia API/MAPI en ambiente autorizado, existen pruebas de errores, límites y mock, se confirma la equivalencia con el legado, hay plan de retorno, y los documentos OpenSpec/contratos reflejan exclusivamente documentos. Debe ser posible buscar `comments`, `Comentario`, `CommentStatus` en **los artefactos activos de esta migración** y no encontrar requisitos funcionales de comentarios; las menciones que identifiquen expresamente la exclusión sí son válidas.

## 9. Decisiones por validar antes de desarrollar

- Correspondencia efectiva de tablas, SP y firmas desplegadas; transacciones y concurrencia a cargo de DBA.
- Semántica de `Publico`, fecha de caducidad, filtro de documentos sin archivo, eliminación y reemplazo.
- Fuente confiable de tipos y límites de archivo; propuesta contractual inicial 25 MB y PDF/DOC/DOCX/XLS/XLSX/PPTX, sujeta a compatibilidad y aprobación.
- Identificadores de rutas API existentes y estrategia de versionado; no asumir rutas propuestas como desplegadas.
- Si la implementación se incorpora a la UI actual JS/JSX (opción preferida según estándares) o se autoriza un proyecto nuevo TS con despliegue independiente.
