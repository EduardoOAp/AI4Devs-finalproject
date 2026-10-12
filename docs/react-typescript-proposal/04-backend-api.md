# 04. Backend y contrato API — documentos STI

> **Estado:** contrato objetivo para discusión/implementación, no endpoints confirmados. Ajustar rutas y DTOs al versionamiento, serialización y patrones reales de `Bccr.Supen.Ves.Api` y `Bccr.Supen.Ves.Mapi`. **Sin comentarios**.

## 1. Arquitectura y responsabilidades

`UI → API Controller → SI.SG → IRequestsHandler → MAPI Controller → BW → DA → SQL Server`. .NET 8 / ASP.NET Core en API y MAPI; EF Core 8 y procedimientos almacenados bajo administración del DBA en MAPI. `Abstracciones` de API y MAPI son árboles independientes; crear/ajustar DTOs en sus respectivos contratos sin asumir proyecto compartido. No introducir Node.js, Express, Prisma ni migraciones automáticas.

## 2. Operaciones candidatas y trazabilidad

| Método / ruta lógica propuesta (no definitiva) | Propósito | Referencia SQL documentada | Permiso mock |
|---|---|---|---|
| `GET /api/document-types` | Tipos activos | `ListarTiposDocumentos` | `documents.read` |
| `GET /api/documents` | Filtrado/paginación | `ListarDocumentos` | `documents.read` |
| `GET /api/documents/{id}` | Detalle sin binario | `ObtenerDocumento` | `documents.read` |
| `POST /api/documents` | Alta de metadatos | `GuardarDocumento` | `documents.create` |
| `PUT /api/documents/{id}` | Actualización metadatos | `GuardarDocumento` | `documents.update` |
| `POST /api/documents/{id}/file` | Adjuntar o reemplazar | `SubirArchivo` | `documents.upload` |
| `GET /api/documents/{id}/file` | Descargar | `BajarArchivo` (verificar existencia real) | `documents.download` |
| `DELETE /api/documents/{id}` | Eliminar según reglas | `EliminarDocumento` | `documents.delete` |

**Atención:** los paths son una notación funcional propuesta por el documento original, no garantías de VES. Antes de codificar, revisar controllers, API versioning, DTOs, gateway SI.SG, `IRequestsHandler`, configuración y consumidores y actualizar OpenAPI si se declara fuente canónica para esta migración. No crear rutas `/comments`.

## 3. Esquemas lógicos de intercambio (ilustrativos)

### Documento

```json
{
  "id": 42,
  "documentTypeId": 2,
  "title": "Informe trimestral",
  "summary": "Informe publicado",
  "isPublic": true,
  "publicationDate": "2026-10-01T00:00:00",
  "expirationDate": null,
  "fileName": "informe.pdf"
}
```

Los nombres ingleses del ejemplo **no son convención obligatoria**: acordar mapeo final desde `CodigoDocumento`, `CodigoTipoDoc`, `Titulo`, `Resumen`, `Publico` (`S/N`), `FechaPublicacion`, `FechaCaduca`, `NombreArchivo` y mantener convención española de contratos existentes si corresponde. El binario `Documento` no viaja dentro del JSON de listado/detalle. `UsuarioPublica` debe resolverse en backend desde contexto simulado en pruebas (o desde arquitectura de seguridad real en un proyecto posterior), sin confiar en texto libre suministrado por navegador.

### Listado paginado propuesto

```json
{"items": [], "page": 1, "pageSize": 20, "total": 0}
```

Filtros lógicos `typeId`, `text`, `page`, `pageSize`; validar rangos, longitud y códigos. **El SP `ListarDocumentos` del SQL adjunto no garantiza paginación ni búsqueda textual nativa**: si se necesita, definir implementación y coherencia de `total`, orden y filtros con el DBA o con paginación controlada de resultados, justificando costes. Filtra tipo con centinela `-1`, excluye vencidos y registros sin `NombreArchivo` y ordena por fecha descendente. No modificar semántica sin decisión del negocio.

## 4. Validación de metadatos

| Campo SQL documentado | Restricción técnica de referencia | Validación a negociar |
|---|---|---|
| `Titulo` | `varchar(100)` NOT NULL | Requerido; máximo compatible con longitud SQL |
| `Resumen` | `varchar(255)` NOT NULL | Requerido; máximo compatible |
| `CodigoTipoDoc` | `int NULL`, FK | Opcional en SQL; si presente, tipo válido |
| `Publico` | `char(1)` NOT NULL, `S/N` | Valor explícito permitido; traducción booleana en DTO si se aprueba |
| `FechaPublicacion` | `datetime` NOT NULL | Fecha válida y zona/convención acordada |
| `FechaCaduca` | `datetime NULL` | Fecha opcional; política relativa a publicación por confirmar |
| `NombreArchivo` | `varchar(30)` NULL | Restricción **real de columna**; no asumir longitud de parámetro SP |
| `UsuarioPublica` | `varchar(50)` NOT NULL | Contexto backend de pruebas; no aceptar identidad no verificada |

Toda entrada debe validarse en API y las invariantes de negocio en BW. No inventar `status`/enums de otro proyecto. Revisar compatibilidad de `DateTime` SQL con el contrato de fecha y serialización ASP.NET real.

## 5. Contrato de archivos propuesto y riesgos

Límite inicial documentado en el diseño previo: **25 MB** por archivo. Tipos propuestos: PDF (`.pdf`, `application/pdf`), Word (`.doc`, `.docx`), Excel (`.xls`, `.xlsx`) y PowerPoint (`.pptx`) con los MIME oficiales correspondientes. Validar extensión, MIME declarada y, cuando sea viable, firma/contenido; el MIME del navegador no es suficiente. Verificar con el negocio si se admite `.ppt`, dado que el diseño previo especifica solo `.pptx`. Los límites deben concordar con web server, API, gateway y MAPI y con almacenamiento `image` legado. Nunca leer archivos ilimitados completos en memoria sin control.

Subida: `multipart/form-data` con campo de archivo acordado; archivo ausente/vacío o formulario mal formado → `400`; mayor a 25 MB → `413`; tipo no permitido → `415`; sin identidad mock → `401`; sin permiso mock → `403`; documento inexistente → `404`; conflicto de estado/concurrencia → `409` si corresponde. Errores siguen middleware institucional, **no un envelope JSON ad hoc**.

Descarga: devolver contenido binario correcto, tipo MIME y `Content-Disposition` seguro con nombre; `404` si documento o archivo no existe. No exponer rutas físicas ni datos internos. `SubirArchivo` tiene código especial `1` y puede borrar fila si nombre nulo según SQL de referencia: no activar sin pruebas controladas. `BajarArchivo` está marcado como procedimiento de referencia no verificado.

## 6. Mock de seguridad: contrato y aislamiento

Un proveedor sustituible de **pruebas** entrega identidad simulada y permisos `read/create/update/upload/download/delete`. El backend valida por operación y por documento cuando se definan reglas. Los `401` y `403` representan *escenarios simulados*, no integración real. No implementar `SeguridadVES`, IdP, login ni mecanismos productivos de autorización para esta entrega. No desactivar JWT/mTLS institucional ni propagar bypass de certificados a staging/producción: confinar el mock a harness/entornos aislados. No confiar en `role` o `userId` enviados libremente por el cliente como autorización.

## 7. Tratamiento de errores y efectos reales

- `200/201`: solo si se confirma lectura/cambio real; `204` si la convención existente aplica a eliminación.
- `400`: validación/binding incorrecto; `401/403`: mock; `404`: no encontrado; `409`: conflicto identificable; `413/415`: archivo; `500`: error inesperado sin filtrar SQL, stack traces ni detalles sensibles.
- Respetar serialización y middleware de errores vigente en los componentes, versionamiento y formato de controllers existentes.
- La API traduce errores de la operación MAPI/DA sin suponer que el retorno de `GuardarDocumento` confirma filas actualizadas. El SP usa condición conjuntiva problemática y nulos; definir comprobación de filas afectadas/lectura posterior y coordinar corrección con DBA.

## 8. Persistencia y transacciones

En DA reutilizar procedimientos y SQL parametrizado (`FromSqlInterpolated` según el patrón de MAPI cuando aplique). No concatenar SQL, no cambiar scripts de schema ni ejecutar EF migrations. Solicitar a DBA análisis de `MAX(ID)+1` (concurrencia y vacío), lógica de `UPDATE`, compatibilidad de columnas, eliminación física y contrato de `SubirArchivo`/`BajarArchivo`. Registrar si un procedimiento incluye su propia transacción y cómo se manejan fallos parciales; no suponer atomicidad entre requests separados.

## 9. Matriz de pruebas obligatoria

Pruebas unitarias de controlador, SI.SG y BW para entradas válidas, errores, permisos mock, cambios parciales y casos límite; integración de API↔MAPI en ambiente autorizado; validación documental de lectura y escritura contra el SP real con datos controlados, sin alterar producción; `curl` a endpoints efectivos; E2E de UI si aplica. Incluir explícitamente tabla vacía, alta concurrente, edición de solo un campo, `FechaCaduca=null`, nombre de 30/31 caracteres, subida a documento `1`, descarga sin archivo y eliminación cancelada. No declarar tests ejecutados si solo se especificaron.

## 10. Criterios de aceptación y documentación relacionada

Al aprobarse: contrato OpenAPI actual de documentos coincide con controller real, SI.SG, MAPI, validaciones y UI; endpoints versionados como exige VES; sin comentarios; sin migraciones EF; riesgo DBA cerrado o registrado; pruebas y reportes en OpenSpec. El documento `05-plan-paso-a-paso.md` contiene el orden de implementación y `06-manejo-y-mantenimiento-de-comentarios.md` solo explica exclusión.
