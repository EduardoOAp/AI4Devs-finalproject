# 05. Plan paso a paso — migración exclusivamente documental

> **Estado:** plan objetivo, no ejecución acreditada. **No construir comentarios en ninguna etapa.** Aplicar `config.yaml`, `openspec-tasks-mandatory-steps.md` y convenciones de cada componente VES. Todas las tareas implementables deben ser pequeñas (orientación: hasta 2 horas, según configuración OpenSpec), verificables y trazables.

## 0. Preparación: fuentes y alcance (antes de implementar)

1. Confirmar el cambio OpenSpec activo y leer `openspec/config.yaml`, estándares adjuntos y los `README.md`, `AGENT.md`, `SPEC.md` de UI/API/MAPI donde existan.
2. Identificar cada repositorio realmente afectado; crear **primero** rama feature por componente según proceso actual: UI/MAPI desde `dev` y API desde `main`, sujeto a verificación del repositorio. No asumir que una rama cubre automáticamente tres proyectos.
3. Documentar que el alcance se limita a `STI_Documentos` y `STI_TiposDocumento` y las operaciones documentales aprobadas. Retirar del cambio activo historias, tests, endpoints, entidades y DTOs de comentarios.
4. Congelar una matriz de trazabilidad: requerimiento → controller/API → SI.SG → MAPI/BW/DA → SP → UI → prueba. Mantener explícitas las decisiones abiertas.

**Evidencia de salida:** alcance firmado, inventario técnico, ramas y checklist OpenSpec; ninguna tarea de comentarios.

## 1. Descubrimiento y validación del legado

- Inventariar Web Forms, SOAP y componentes de documentos reales, flujos completos, roles/acciones, y errores frecuentes.
- Contrastar `data-model.md`, `Tablas_Documentos.md` (documentados como referencias), `MAPI/Documentacion/SCHEMA.md` y SP desplegados mediante mecanismos autorizados/DBA.
- Verificar contratos de `ListarTiposDocumentos`, `ListarDocumentos`, `ObtenerDocumento`, `GuardarDocumento`, `SubirArchivo`, `BajarArchivo`, `EliminarDocumento`.
- Describir reglas actuales de vencimiento, publicación `S/N`, visibilidad de registros sin archivo, alta/edición, descarga, eliminación.
- Analizar y escalar riesgos SQL: `MAX(ID)+1`, condiciones `UPDATE` y `NULL`, `SubirArchivo` con código `1`, columna de nombre de 30 chars, borrado físico y procedimiento de descarga no verificado.

**Salida/Gate:** especificación funcional de documentos y diferencias conocidas entre referencia y DB real; validación DBA donde corresponda. Sin ello, no habilitar escrituras.

## 2. Contratos antes que implementación

- Resolver rutas/versionado y nombres de DTOs compatibles con API y MAPI reales.
- Definir metadatos, filtros, paginación `total`, estado de errores y política de fechas.
- Acordar límites de archivos (propuesta: 25 MB; tipos PDF/DOC/DOCX/XLS/XLSX/PPTX), longitud de nombre e infraestructura que recibe binarios.
- Decidir alcance y mecanismo del mock de seguridad en pruebas aisladas; definir matriz autorizado/denegado/identidad ausente.
- Actualizar OpenAPI documental efectivamente aplicable y revisar que no contenga recursos de comentarios dentro de este cambio. `docs/api-spec.yml` requiere confirmación de contenido/canonicalidad, no suponer que ya documenta este dominio.

**Salida/Gate:** contrato aprobable con ejemplos, errores, matriz de permisos mock y reglas de compatibilidad; revisión UI/API/MAPI.

## 3. Preparar adaptación en MAPI y datos

- Crear/modificar DTOs del componente MAPI, DA parametrizado y BW siguiendo patrones actuales.
- Implementar primero lecturas: tipos/listado/detalle sin binario. Probar mapeo y reglas de filtro.
- Implementar escrituras solo tras cierre de problemas críticos de SP por DBA y con datos controlados.
- Determinar cómo se confirma el efecto real en alta, edición, subida, borrado y si se requiere manejo de transacciones.
- No introducir ORM paralelo ni migración EF ni cambio de esquema desde aplicación.

**Salida/Gate:** contratos MAPI y pruebas unitarias correspondientes; comprobación autorizada de SP y casos límite documentada.

## 4. Exponer y orquestar API

- Controladores documentales compatibles con convenciones de rutas/versionado API.
- Implementar SI.SG/IRequestsHandler hacia MAPI y mapeo de excepciones según infraestructura institucional.
- Incorporar validación de parámetros, tamaño/tipo de archivo y DTOs; evitar SQL/HTTP interno en controllers.
- Usar proveedor mock en pruebas exclusivamente, sin modificar seguridad productiva o bypass mTLS.
- Pruebas de API y MAPI, contrato y llamadas manuales `curl` a URLs **verificadas**.

**Salida/Gate:** endpoints de consulta y mutación según capacidades implementadas, con reportes de pruebas; no atribuirlos a ambientes que no se probaron.

## 5. Construir UI React dentro de STI

- Usar React 19, JS/JSX, Vite, Material UI y cliente HTTP existentes; no imponer TypeScript por nombre histórico del documento.
- Entregar primero catálogo/listado paginado y filtrado, luego detalle, después creación/edición, y al final archivos/eliminación.
- Cada pantalla incluye carga, vacío, error, éxito, restricciones del mock y accesibilidad.
- No enlazar UI con MAPI/SOAP/SQL. Pruebas Vitest + React Testing Library y pruebas de interacción de los flujos.

**Salida/Gate:** vertical completo UI→API→MAPI validado con registros controlados, sin afectar otros módulos.

## 6. Asegurar archivos, permisos simulados y fallos

- Ejercitar archivo permitido/denegado, falta de archivo, vacío, tamaño 25 MB y 25 MB+1 byte, diferencias MIME/extensión, nombres largos, errores de gateway, red y timeout.
- Probar autorizado/403/401 **simulados**, sin credenciales productivas; no habilitar endpoints de mock públicamente.
- Confirmar que los cambios de metadatos persisten realmente y que la eliminación no produce pérdida accidental.
- Confirmar descargas con contenido, MIME y nombre seguros; medir comportamiento de binarios grandes.

**Salida/Gate:** matriz de pruebas y evidencias con fallos resueltos o decisiones de riesgo.

## 7. Convivencia y piloto reversible

- Probar rutas nuevas y Web Forms existentes con la misma semántica de datos, evitando escrituras concurrentes descontroladas.
- Revisar rendimiento de consulta, carga/descarga y comportamiento de caducidad/publicación.
- Habilitar progresivamente solo en entorno/usuarios aprobados, con mecanismo de retorno definido y verificado.
- No usar el mock como seguridad de producción. La adopción productiva de nueva funcionalidad requiere evaluación adicional de autenticación/autorización institucional.

**Salida/Gate:** acta de validación funcional, riesgos, criterios de reversión y decisión explícita de avance.

## 8. Actualización documental y cierre OpenSpec

- Revisar/actualizar los seis documentos de esta migración, OpenAPI, specs y `tasks.md`, guías y contratos reales afectados.
- Reflejar cambios derivados de implementación, pruebas ejecutadas y discrepancias. No afirmar desplegado a partir de un diseño.
- Asegurar que ningún artefacto de implementación activa contiene funciones de comentarios.
- PR por repositorio hacia rama de destino correcta; cumplir pruebas unitarias, verificación controlada de DB, `curl`, E2E Playwright MCP **si aplica y está disponible** y reportes de evidencia exigidos por OpenSpec. Si alguna verificación no es posible, marcar bloqueado y motivo; no darla por ejecutada ni delegar su ejecución simulando éxito.

**Salida/Gate:** documentación y reporte de pruebas coherentes, PR revisado y ruta de rollback.

## 9. Matriz de aceptación

| Capacidad | Evidencia mínima |
|---|---|
| Tipos y listado | Datos/campos correctos, filtros, orden, paginación, expiración |
| Detalle | Metadatos correctos, errores 404, sin binario en JSON |
| Alta y actualización | Validaciones, efecto confirmado, casos de nulos/concurrencia |
| Subida y descarga | Límites, MIME, nombre, binario correcto, errores HTTP |
| Eliminación | Regla autorizada, confirmación, efecto y recuperación/retorno definidos |
| Mock | 401/403/éxito simulados en backend; aislamiento |
| Integración | UI→API→MAPI→DA, sin acceso directo UI a SQL/SOAP |
| Regresión y convivencia | Web Forms sigue funcionando y rollback ensayado |
| Exclusión de comentarios | No hay rutas, DTO, tablas, pantallas ni tests nuevos de comentarios |

**Regla de avance:** la siguiente etapa no se declara completada sin evidencia y sin resolver bloqueadores críticos; otras etapas no dependientes pueden planificarse, pero nunca disfrazar incertidumbre como aprobación.
