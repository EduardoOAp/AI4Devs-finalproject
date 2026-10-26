# Ejercicio: módulo original de Documentos para VES

## 1. Objetivo

Planificar la incorporación del módulo original de Documentos a la solución existente `VES/Bccr.Supen.Ves.Api`, siguiendo sus convenciones y capas API, Abstracciones y SI.SG. La API continuará comunicándose con MAPI; el acceso a SQL Server y la invocación de procedimientos almacenados pertenecen al MAPI.

La fuente funcional de referencia es `VES/Tablas_Documentos.md`, que describe las tablas `VES.STI_Documentos` y `VES.STI_TiposDocumento`, y los procedimientos `ListarTiposDocumentos`, `ListarDocumentos`, `ObtenerDocumento`, `GuardarDocumento`, `SubirArchivo` y `EliminarDocumento`. El usuario también proporcionó el procedimiento `[VES].[BajarArchivo]`, que devuelve el binario por código. El servidor SQL no se consultó directamente.

Este documento y la propuesta OpenSpec planifican el trabajo. No implementan código de producción ni resuelven las ambigüedades identificadas.

## 2. Requisitos funcionales

1. Consultar los tipos documentales activos.
2. Listar documentos por tipo documental y permitir la selección legacy de todos los tipos.
3. Consultar los metadatos de un documento por su código.
4. Crear o actualizar metadatos mediante la operación de guardado.
5. Asociar o reemplazar el archivo de un documento. Aceptar `.doc`, `.docx`, `.pdf`, `.ppt` y `.pptx`, hasta 10 MB, con solicitud `multipart/form-data` al API.
6. Descargar el archivo binario mediante `[VES].[BajarArchivo]`, conservando el nombre del archivo asociado a sus metadatos.
7. Eliminar un documento por su código.
8. Exponer esas operaciones a través de la API VES y su gateway a MAPI, reutilizando el esquema de autorización, versionamiento, manejo de errores y configuración existente.
9. Mantener SQL Server y las tablas/procedimientos de referencia; no agregar persistencia paralela.

## 3. Requisitos no funcionales

- El módulo debe respetar la separación existente: controlador/API, contratos en Abstracciones, gateway SI.SG y operaciones de datos en MAPI.
- La API no debe conectarse directamente a SQL Server.
- Las llamadas a procedimientos desde MAPI deben seguir las convenciones seguras de acceso a datos del proyecto MAPI, incluida la parametrización de valores.
- Los endpoints deben mantener las convenciones vigentes de autorización y versión de API.
- Los errores del MAPI o de SQL Server no deben transformarse en respuestas de éxito.
- La operación de archivo debe transportar el contenido de forma compatible con los límites de request, memoria y formatos que se acuerden antes de implementar.
- Los contratos deben conservar compatibilidad con el consumidor VES acordado y sus formatos de fechas, valores opcionales y errores.
- Deben existir pruebas focalizadas de cada operación en las capas MAPI/API correspondientes e integración del flujo API a MAPI.

## 4. Contratos API documentados

> **Todos los contratos de esta sección están pendientes de verificar contra implementación real.** Son borradores funcionales derivados del SQL de referencia y de las convenciones generales de VES; no representan rutas, nombres de campos ni respuestas ya implementados o aprobados. Verificar especialmente las rutas reales del MAPI, rutas públicas de la API, transporte binario, serialización y respuesta de cada procedimiento.

| Operación | Método y ruta candidata | Entrada candidata | Respuesta candidata | Estado |
|---|---|---|---|---|
| Listar tipos activos | `GET /Documentos/Documentos/ListarTiposDocumentosAsync` | Sin parámetros | Colección de `{ codigoTipoDoc: int, nombre: string }` | **Pendiente de verificar contra implementación real** |
| Listar documentos | `GET /Documentos/Documentos/ListarDocumentosAsync?codigoTipo={codigoTipo}` | Código de tipo; el valor legacy `-1` representa todos los tipos en el procedimiento | Colección de `{ codigo: int, nombre: string, tipo: string, titulo: string, fechaPublicacion: string }` | **Pendiente de verificar contra implementación real** |
| Obtener metadatos | `GET /Documentos/Documentos/ObtenerDocumentoAsync?codigoDocumento={codigoDocumento}` | Código de documento | `{ codigoDocumento, codigoTipoDoc, titulo, resumen, nombreArchivo, publico, fechaPublicacion, usuarioPublica, fechaCaduca }` | **Pendiente de verificar contra implementación real** |
| Guardar metadatos | `POST /Documentos/Documentos/GuardarDocumentoAsync` | `{ codigoDocumento, codigoTipoDoc, titulo, resumen, publico, fechaPublicacion, usuarioPublica, fechaCaduca }` | Código resultante u otra respuesta explícita de éxito/error, todavía por acordar | **Pendiente de verificar contra implementación real** |
| Subir o reemplazar archivo | `POST /Documentos/Documentos/SubirArchivoAsync` | `multipart/form-data`: código y archivo | Resultado de persistencia, todavía por acordar | **Pendiente de verificar contra implementación real** |
| Descargar archivo | `GET /Documentos/Documentos/DescargarArchivoAsync?codigoDocumento={codigoDocumento}` | Código de documento | Archivo binario con nombre y tipo de contenido inferido de la extensión permitida | **Pendiente de verificar contra implementación real** |
| Eliminar documento | `DELETE /Documentos/Documentos/EliminarDocumentoAsync?codigoDocumento={codigoDocumento}` | Código de documento | Resultado de persistencia, todavía por acordar | **Pendiente de verificar contra implementación real** |

> **Estado de «Obtener metadatos» (apertura, 2026-10-22):** implementado en API y MAPI como `GET /Documentos/{codigoDocumento}` (metadatos: `codigoDocumento, codigoTipoDoc, titulo, resumen, nombreArchivo, fechaPublicacion, usuarioPublica, fechaCaduca, archivoDisponible`) y `GET /Documentos/{codigoDocumento}/Archivo` (binario para previsualizar PDF). Los documentos caducados se pueden abrir. Verificado solo localmente en ambiente no productivo (curl y Playwright, sin GAUDI; 401/403 no verificados); no es verificación de producción. Ver `docs/api-spec.yml`. Edición, eliminación, subida y descarga siguen sin implementar.

Los tipos anteriores se expresan de forma conceptual. Los nombres exactos, nullabilidad, casing JSON, representación de fechas, status HTTP, envoltura de respuesta, validación y semántica de campos deben verificarse con los controladores y consumidores reales.

El procedimiento de descarga proporcionado por el usuario devuelve únicamente `Documento`; MAPI debe obtener el nombre de archivo asociado mediante metadatos. La definición se toma como referencia de implementación, sin afirmar que se verificó en el servidor SQL.

## 5. Restricciones de alcance

- Se implementará como módulo de la solución existente `Bccr.Supen.Ves.Api`, no como aplicación independiente.
- Se mantendrá la arquitectura VES existente: API, Abstracciones y SI.SG en el gateway, y MAPI para negocio/acceso a SQL Server.
- Se conservarán SQL Server y los procedimientos almacenados documentados como referencia; la API/SI.SG no ejecutará SQL.
- No se implementan funcionalidades adicionales descritas en la propuesta evolucionada del README: comentarios, respuestas a comentarios, historial de comentarios, ciclo de vida ampliado, flujos de publicación externa, ni una nueva arquitectura React/Node/PostgreSQL/Prisma.
- No se modifican tablas ni procedimientos desde el código. _Decisión 2026-10-22: el listado muestra también los documentos caducados, por lo que ya no se requiere el filtro de vigencia en `ListarDocumentos`._
- Se preservan la generación de códigos y el predicado de actualización de `GuardarDocumento`, y el centinela `1` de `SubirArchivo`, según las decisiones del usuario.
- MAPI rechaza nombre de archivo nulo antes de invocar `SubirArchivo`.
- Los contratos candidatos permanecen marcados como pendientes de verificar contra la implementación real hasta que sus rutas, payloads y respuestas estén implementados y probados.

## 6. Criterios de aceptación verificables

1. La propuesta OpenSpec y este documento describen el mismo alcance original y excluyen las funcionalidades de la propuesta evolucionada.
2. Los siete contratos candidatos de la sección 4 permanecen explícitamente marcados como **pendientes de verificar contra implementación real** hasta que las rutas, DTOs y respuestas se implementen y prueben.
3. Cada operación CRUD/catálogo tiene un contrato de entrada, salida y errores consistente entre API y MAPI, verificado con pruebas contra las interfaces implementadas.
4. El catálogo devuelve solo registros con `Estado = 'A'`, con prueba para catálogo vacío.
5. El listado tiene pruebas para filtro por tipo, selección de todos los tipos, orden de publicación descendente, fechas futuras/expiradas/nulas incluidas todas en el listado (la caducidad no excluye).
6. Obtener metadatos y descargar binario tienen pruebas para documentos existentes, inexistentes y sin contenido; las ausencias devuelven 404.
7. Guardar metadatos tiene pruebas separadas de alta, el predicado de actualización legacy, entradas sin cambios y fallas; `-1` no se expone como éxito.
8. Subir archivo tiene pruebas de `.doc`, `.docx`, `.pdf`, `.ppt`, `.pptx`, `.xls`, `.xlsx`, tamaño máximo de 10 MB, nombre nulo, archivo existente/no existente y código centinela `1`.
9. Eliminar tiene pruebas de eliminación exitosa y registro inexistente (404).
10. Las pruebas comprueban que la API delega a MAPI y que el MAPI ejecuta llamadas parametrizadas en SQL Server; no hay acceso directo de la API a la base de datos.
11. Las pruebas unitarias focalizadas, las pruebas de integración acordadas y los builds de los proyectos afectados pasan antes de cerrar el cambio.
12. Los cambios de producción respetan las carpetas, interfaces, configuración, autenticación y versionamiento de los proyectos VES existentes.

## 7. Riesgos o ambigüedades que deben investigarse antes de implementar

- **Generación de identificadores:** por decisión del usuario, conservar `MAX(CodigoDocumento) + 1`; permanece el riesgo de alta concurrente y de tabla vacía, que debe comprobarse en un entorno de pruebas antes de producción.
- **Predicado de actualización:** por decisión del usuario, conservar la condición legacy que exige que varios campos difieran todos; probar expresamente su comportamiento con fechas nulas y valores iguales.
- **Fechas de caducidad:** decisión 2026-10-22: el listado y la apertura incluyen documentos caducados; no se despliega ningún filtro de vigencia. `NULL` sigue significando vigencia indefinida en el detalle.
- **Regla completa de listado:** conservar exclusión de `NombreArchivo` nulo, filtrar por tipo (o todos con `-1`) y ordenar por fecha de publicación descendente.
- **Carga de archivos:** por decisión del usuario, conservar el código centinela `1`; rechazar nombre nulo antes del SP para evitar que borre el registro.
- **Tipo y tamaño del archivo:** el usuario aprobó `.doc`, `.docx`, `.pdf`, `.ppt`, `.pptx`, `.xls`, `.xlsx` y 10 MB; verificar que límites de IIS, API y MAPI acepten el contenido multipart y su representación Base64 interna.
- **Definición de `BajarArchivo`:** el usuario proporcionó el procedimiento `[VES].[BajarArchivo]`; confirmar su presencia y firma en el SQL Server de destino. La base no se consultó desde este entorno.
- **Resultados vacíos/errores:** aplicar 404 para metadatos/archivo/eliminación sin registro y error explícito si guardar devuelve `-1`; verificar fallas de conversión y SQL.
- **Fidelidad de la referencia SQL:** el usuario autorizó usar el script documentado y su definición de `BajarArchivo` como referencia; el servidor no fue consultado y esa limitación debe constar en documentación y liberación.

Las decisiones de comportamiento indicadas en las restricciones y riesgos ya fueron aprobadas por el usuario; los puntos restantes son verificaciones de implementación, de entorno y de despliegue, no decisiones para reabrir silenciosamente.
