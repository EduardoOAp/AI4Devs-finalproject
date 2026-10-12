# 06. Exclusión del manejo de comentarios de la migración documental

> **Decisión de alcance vigente:** la migración comprende **únicamente el módulo de documentos STI**. **No se implementará la funcionalidad de comentarios.** Este documento reemplaza la propuesta anterior de “Manejo y mantenimiento de comentarios” para impedir que sus ideas se interpreten como requisitos aprobados.

## 1. Propósito de esta constancia

Registrar explícitamente que los comentarios de usuarios sobre documentos fueron considerados en diseños anteriores pero quedan **fuera de esta entrega**. Su existencia en versiones históricas del diseño no autoriza crear entidades, API ni tareas. No es una especificación de la capacidad de comentarios.

## 2. Funcionalidad que no debe desarrollarse

| Elemento anterior | Decisión actual |
|---|---|
| `DocumentoComentario` y `DocumentoComentarioHistorialEstado` | No crear entidades, tablas ni cambios SQL |
| Comentarios raíz, respuestas e hilos | No crear UI, estructuras ni casos de uso |
| Estados `open`, `in_review`, `resolved`, `reopened`, `archived` | No introducir enums ni estados en contratos |
| Historial append-only de estados de comentario | No crear trazabilidad específica de comentarios |
| Rutas `/api/documents/{id}/comments...` | No exponer ni implementar endpoints |
| Editar/resolver/reabrir/archivar/eliminar comentarios | No implementar reglas ni permisos mock asociados |
| Pruebas, datos semilla o tareas sobre comentarios | No incluir en `tasks.md`, tests o piloto documental |
| Mock de permisos `comments.*` | No configurar escenarios ni perfiles para esta capacidad |

Este descarte **no** elimina ni cambia tablas, funcionalidades o comentarios existentes que pudieran residir en otros sistemas; simplemente establece que este proyecto no los modifica ni añade. Tampoco autoriza eliminar trazas históricas del repositorio sin revisar impacto.

## 3. Alcance que sí permanece

La migración documental cubre catálogo de tipos, consulta/listado, detalle, alta y modificación de metadatos, carga/descarga de archivos, eliminación con reglas definidas, permisos **simulados para pruebas** y convivencia con Web Forms. Véase `01-alcance-y-principios.md` para límites completos y `04-backend-api.md` para el contrato documental propuesto.

## 4. Diferencia entre comentarios de usuario y comentarios técnicos

La exclusión se refiere a **comentarios funcionales asociados a documentos**. No prohíbe comentarios aclaratorios en C#/JS, observaciones de revisión de PR, tickets, documentación de decisiones ni retroalimentación de CodeRabbit. Estos siguen los estándares VES de calidad y documentación. Ninguno constituye una entidad de comentarios sobre documentos.

## 5. Limpieza y control de coherencia

Al actualizar el cambio OpenSpec y documentos técnicos:

1. Retirar del alcance activo historias y criterios de aceptación sobre comentarios, respuestas, archivado e historial.
2. Eliminar definiciones **propuestas no implementadas** de endpoints/DTO/enums/esquemas de comentarios en el contrato de esta migración, previa revisión de consumidores y del propietario del archivo. No borrar definiciones ajenas o en uso solo por coincidencia de nombre.
3. Quitar tareas específicas de comentarios de los artefactos `tasks.md` del cambio documental; ajustar estimaciones y dependencias.
4. Evitar modelos, componentes y botones de comentarios en diseños y pruebas del nuevo módulo STI.
5. Revisar referencias cruzadas de los documentos 01–05 y confirmar que únicamente mencionan comentarios para declarar su exclusión.
6. Registrar la decisión en el control de cambios/PR correspondiente y mantener el histórico de decisiones, sin presentar el feature como pendiente dentro del alcance actual.

Búsquedas sugeridas para revisión humana o automatizada: `DocumentoComentario`, `CommentStatus`, `comments/`, `parentCommentId`, `historial de comentarios`, `reopen`, `replies`. Verificar contexto antes de borrar: las coincidencias de esta constancia son intencionales.

## 6. Criterios de aceptación

- Ninguna capacidad, tabla, procedimiento, API, frontend ni prueba **nueva** relacionada con comentarios forma parte del cambio.
- Los artefactos de OpenSpec y el contrato de la migración documental describen solo las operaciones documentales autorizadas.
- Los permisos mock cubren únicamente operaciones de documentos; no existe implementación de permisos de comentarios.
- El plan y los criterios de cierre del módulo no dependen de comentarios.
- Cualquier futura iniciativa de comentarios exige **requerimiento independiente, análisis de datos, arquitectura, contratos, aprobación y planificación específicos**; no debe activarse por reutilización de esta versión histórica.

## 7. Estado de la decisión

**Excluido del proyecto actual.** No es una fase pospuesta de esta misma entrega ni una tarea implícita para el equipo de frontend/backend.
