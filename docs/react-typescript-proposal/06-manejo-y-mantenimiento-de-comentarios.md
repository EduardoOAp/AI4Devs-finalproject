# 6. Manejo y mantenimiento de comentarios

## Objetivo

Adaptar al módulo documental de VES el patrón de trabajo trazable observado en `lidr-specboot`: comentarios organizados, estados claros, responsables, historial y reglas para mantenerlos sincronizados con el documento.

Este diseño se refiere a comentarios de usuarios sobre documentos. Los comentarios de código y de documentación técnica se mantienen mediante revisión de cambios y actualización de los documentos correspondientes.

## Situación actual de VES

En el código analizado no se observa una entidad, tabla, procedimiento ni servicio de comentarios para `STI_Documentos`.

Por tanto, no se debe intentar reutilizar `DataSet` o campos de `STI_Documentos` para guardar comentarios. El comentario debe ser una entidad independiente relacionada por `CodigoDocumento`.

## Entidad propuesta

Nombre conceptual: `DocumentoComentario`.

```text
DocumentoComentario
- Id
- CodigoDocumento (FK)
- IdComentarioPadre (FK opcional)
- Texto
- Estado
- AutorLogin
- FechaCreacion
- FechaActualizacion
- FechaResolucion (opcional)
- ResueltoPor (opcional)
- VersionDocumento (opcional)
```

### Estados

- `Abierto`: requiere atención.
- `En revisión`: alguien está trabajando en él.
- `Resuelto`: la observación fue atendida.
- `Reabierto`: volvió a requerir atención.
- `Archivado`: se conserva para historial, pero ya no aparece por defecto.

Los nombres son una propuesta y deben validarse con los usuarios antes de crear tablas o procedimientos.

## Hilos y respuestas

`IdComentarioPadre` permite que un comentario tenga respuestas y forme un hilo.

```text
Documento
  +-- Comentario raíz
       +-- Respuesta
       +-- Respuesta
            +-- Respuesta
```

La interfaz debe mostrar cada hilo ordenado por fecha, con el comentario raíz visible y sus respuestas agrupadas.

## Reglas de mantenimiento

1. Un comentario pertenece a un único documento.
2. El autor puede editar su comentario mientras esté abierto, según la política de seguridad.
3. Resolver un comentario no lo elimina.
4. Reabrir un comentario conserva el historial anterior.
5. Un comentario archivado no debe desaparecer de la auditoría.
6. El backend debe verificar que el usuario tenga acceso al documento antes de leer o modificar sus comentarios.
7. Las eliminaciones físicas deben evitarse; usar archivado o eliminación lógica.
8. El texto debe validarse en backend, incluyendo longitud máxima y contenido vacío.
9. Cada cambio de estado debe guardar quién lo realizó y cuándo.

## Flujo funcional

```text
Abrir documento
      |
      v
Consultar comentarios activos
      |
      v
Crear comentario o respuesta
      |
      v
Asignar estado abierto
      |
      +--> responder / editar
      |
      +--> marcar en revisión
      |
      +--> resolver
                |
                +--> reabrir si la respuesta no es suficiente
                +--> archivar cuando termine el ciclo
```

## API propuesta

```text
GET    /api/documents/{documentId}/comments
POST   /api/documents/{documentId}/comments
POST   /api/documents/{documentId}/comments/{commentId}/replies
PUT    /api/documents/{documentId}/comments/{commentId}
PATCH  /api/documents/{documentId}/comments/{commentId}/status
DELETE /api/documents/{documentId}/comments/{commentId}
GET    /api/documents/{documentId}/comments/history
```

La eliminación debe representar archivado o baja lógica, no borrar el historial sin autorización explícita.

## Contrato de comentario

```json
{
  "id": 15,
  "documentId": 42,
  "parentCommentId": null,
  "text": "Actualizar la fecha de vigencia.",
  "status": "open",
  "author": {
    "login": "usuario"
  },
  "createdAt": "2026-09-17T10:30:00Z",
  "updatedAt": "2026-09-17T10:30:00Z",
  "replies": []
}
```

El frontend React debe consumir este contrato tipado. No debe conocer tablas SQL, procedimientos almacenados ni `DataSet`.

## Componentes React propuestos

```text
features/comments/
  api/commentsApi.ts
  components/CommentThread.tsx
  components/CommentComposer.tsx
  components/CommentStatusControl.tsx
  components/CommentHistory.tsx
  pages/DocumentCommentsPanel.tsx
  schemas/commentSchemas.ts
  types/commentTypes.ts
```

El panel de comentarios puede aparecer en `DocumentDetailsPage` y posteriormente en el editor, cuando las reglas de negocio estén aprobadas.

## Permisos

La autorización debe apoyarse en la identidad y los permisos existentes de `SeguridadVES`, con reglas específicas para:

- consultar comentarios
- crear comentarios
- editar comentarios propios
- cambiar estado
- archivar comentarios
- consultar historial

La UI puede ocultar acciones no permitidas, pero la API debe volver a validar cada operación.

## Persistencia y transición

### Fase inicial

Crear un adaptador de comentarios separado de `Documentos.vb`. Si todavía no se usa ORM, puede invocar procedimientos almacenados nuevos y devolver DTOs tipados.

### Fase ORM

Mapear `DocumentoComentario` a una entidad ORM y conservar la API sin cambios. La migración de persistencia no debe obligar a modificar React.

### Integración con el legado

No se recomienda añadir lógica de comentarios directamente a las páginas `.aspx.vb`. La nueva funcionalidad debe vivir detrás de la API propuesta y relacionarse con el documento mediante su identificador.

## Relación con el patrón de lidr-specboot

Se reutilizan estas ideas:

- fuente única para el estado actual del trabajo
- cambios pequeños y revisables
- estados explícitos
- historial antes que borrado
- revisión separada de implementación
- documentación actualizada cuando cambia el comportamiento

No se copia literalmente su estructura de agentes o skills dentro de la aplicación, porque esas piezas sirven para el proceso de desarrollo, no para los usuarios finales de VES.

## Criterios de aceptación iniciales

- un usuario autorizado puede crear un comentario sobre un documento
- una respuesta queda agrupada en el hilo correcto
- el comentario puede pasar por estados definidos
- el historial conserva autor, fecha y cambio realizado
- un usuario sin permiso no puede leer ni modificar comentarios
- el documento puede seguir consultándose aunque no tenga comentarios
- la funcionalidad puede convivir con Web Forms durante un piloto
