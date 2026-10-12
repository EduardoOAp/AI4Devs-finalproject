# 03. Frontend — migración de documentos STI en React

> **Aclaración de tecnología:** el nombre histórico de este archivo contiene “react-typescript”, pero la UI VES verificada por los estándares adjuntos usa **React 19 + JavaScript/JSX + Vite**. **No se exige TypeScript ni conversión masiva**. Adoptarlo solo si se aprueba expresamente una aplicación nueva, fuera de la migración incremental. No incluye comentarios.

## 1. Referencias y dependencias existentes

Aplicar `frontend-standards.md` y las guías del repositorio `Bccr.Supen.Ves.UI`. Stack documentado: React Router DOM 6, Material UI v7, Bootstrap 5 en zonas existentes, Axios/`infraestructura/httpCliente.js`, Vitest, React Testing Library, Day.js y ExcelJS. Respetar tema, componentes, configuración `Codigo/config/{ambiente}.json`, `Constantes.js` y convenciones del módulo STI. No introducir por defecto TanStack Query, Zod, React Hook Form, Jest, Cypress o Playwright como paquetes nuevos: justificar cambios y contrastar dependencias efectivas; Playwright MCP puede usarse para E2E del flujo de trabajo si está disponible.

## 2. Organización propuesta sin reestructuración global

```text
Bccr.Supen.Ves.UI/Codigo/src/
  STI/
    documentos/                  # ubicación final según convención real
      paginas/
        ListaDocumentos.jsx
        DetalleDocumento.jsx
        EditarDocumento.jsx
      componentes/
        FiltrosDocumentos.jsx
        TablaDocumentos.jsx
        FormularioDocumento.jsx
        GestorArchivoDocumento.jsx
      servicios/
        documentosApi.js
      utilidades/
        validacionesDocumento.js
  infraestructura/
    httpCliente.js               # reutilizar cliente vigente
  Constantes.js                  # respetar mecanismo de endpoints existente
```

Estructura ilustrativa: ubicar según la organización efectivamente observada en `STI/`, sin crear directorios genéricos `features/auth/` o `types/` por una plantilla ajena. Si hace falta UI de perfiles simulados, implementarla solamente como *fixture/selector de pruebas* separado de la navegación real.

## 3. Pantallas y comportamiento

### Listado

- Tabla con título, tipo, publicación, caducidad y visibilidad cuando el contrato lo permita; acciones según permisos mock.
- Filtro por tipo, búsqueda textual si backend la soporta, orden conocido y paginación real (no paginar solamente la página cargada).
- Reutilizar los componentes MUI existentes para accesibilidad, navegación por teclado, paginación y estados vacíos.
- Distinguir *no existen registros* de *filtro sin coincidencias* y de *error de servidor*.
- Documentar que `ListarDocumentos` legado excluye documentos sin archivo y caducados; una vista administrativa diferente requiere decisión contractual.

### Detalle

- Consultar por ID con manejo explícito de `404` y errores temporales.
- Ver metadatos, descargar archivo si existe, acciones condicionadas por mock.
- No insertar binario en estado global; pedir descarga bajo demanda.

### Alta/edición

- Campos según `STI_Documentos`: título (100 caracteres), resumen (255), tipo opcional si se conserva nulabilidad, público `S/N`, fecha de publicación, fecha de caducidad opcional; `UsuarioPublica` derivado del contexto mock **en backend**, no confiable si lo envía el navegador.
- El ID se genera o confirma en backend. Evitar campos propios del esquema SQL en el formulario como `Documento image`.
- Errores por campo, botón deshabilitado durante envío, prevención de dobles guardados y confirmación solamente cuando el backend confirma persistencia.
- Evitar suponer que un `200` garantiza que `GuardarDocumento` hizo `UPDATE`: API debe verificar efecto.

### Carga/descarga y eliminación

- Selector de archivos con extensiones inicialmente propuestas `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.pptx` y límite propuesto de 25 MB, sujeto a contrato aprobado.
- Construir `FormData` sin forzar manualmente el boundary de `Content-Type`; mostrar progreso si cliente y API lo soportan.
- Mensajes diferenciados para `400`, `413`, `415`, `404`, `409` y fallo de red. Validación frontend no reemplaza validación servidor.
- Descarga como `Blob`/respuesta binaria, preservar nombre proporcionado de forma segura y liberar URLs de objeto temporales si se crean.
- Solicitar confirmación antes de eliminar; no eliminar optimistamente un registro sin respuesta definitiva, especialmente si el backend invoca `EliminarDocumento` físico.

## 4. Gestión de estado y contratos

La UI obtiene únicamente DTOs JSON de `Bccr.Supen.Ves.Api`; no representa `DataSet`, modelos VB.NET ni tablas SQL como dependencia directa. Mantener estados `loading`, `empty`, `error`, `success`, `saving` y `mockForbidden` de forma consistente. Cancelar/ignorar respuestas obsoletas al cambiar filtros o desmontar vistas; evitar solicitudes duplicadas. Mantener la fuente de verdad del documento en servidor, refrescando listado/detalle después de mutaciones.

Ejemplo conceptual del adaptador (verificar firmas reales de `httpCliente` antes de copiar):

```js
// Interfaz lógica, no implementación acoplada a una firma de cliente no verificada
const documentosApi = {
  listar: (filtros) => cliente.get(rutaDocumentos, { params: filtros }),
  obtener: (id) => cliente.get(`${rutaDocumentos}/${encodeURIComponent(id)}`),
  crear: (datos) => cliente.post(rutaDocumentos, datos),
  actualizar: (id, datos) => cliente.put(`${rutaDocumentos}/${encodeURIComponent(id)}`, datos),
};
```

Resolver `cliente` y `rutaDocumentos` con los mecanismos que realmente expone la UI y el endpoint versionado acordado; el ejemplo no constituye ruta ni API vigente.

## 5. Permisos simulados y accesibilidad

Ocultar o deshabilitar acciones sin permiso como indicación visual, mostrar avisos claros y contemplar una identidad ausente. **La decisión real de la prueba ocurre del lado API/BW**, no en `localStorage` ni mediante ocultamiento de botones. No conectar módulos de inicio de sesión existentes a mock fuera de un ambiente de pruebas controlado. Cumplir controles semánticos, foco, etiquetas de formulario y estándares WCAG 2.1 AA referidos en frontend VES.

## 6. Matriz mínima de pruebas

| Escenario | Resultado esperado |
|---|---|
| Listado sin registros / filtros sin coincidencia | Mensajes claros y sin fallos de render |
| Filtros, paginación, cambios rápidos | Consulta y resultados correctos sin carreras |
| Detalle inexistente | Vista `404` manejable |
| Campos obligatorios y longitudes | Validación visual + rechazo desde API |
| Crear/editar correctamente | Estado actualizado tras confirmación real |
| Mock: autorizado, prohibido, identidad ausente | `success`, `403`, `401` simulados según caso |
| Archivo ausente/vacío/tipo inválido/exceso de tamaño | Errores diferenciados y sin subida aceptada |
| Descarga binaria | Archivo con contenido, MIME y nombre esperados |
| Eliminación y cancelación | No mutar al cancelar; acción confirmada si procede |
| Regresión visual y accesibilidad básica | Navegación teclado, etiquetas, feedback |

Usar Vitest y React Testing Library, integración con API simulada según infraestructura disponible y E2E conforme al flujo OpenSpec si aplica; no afirmar que esas pruebas corrieron solo por documentarlas.

## 7. Definición de terminado del frontend

Integración UI→API documentada y probada, sin llamadas SOAP/MAPI/SQL, sin rutas de comentarios, sin cambios globales de framework, con tests y errores de archivo/permiso representados, y navegación STI compatible con pantallas existentes.
