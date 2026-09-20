# 4. Backend y contrato API

## Objetivo

Exponer una API HTTP pequeña que encapsule los servicios y procedimientos actuales del módulo documental.

## Endpoints iniciales

```text
GET    /api/document-types
GET    /api/documents
GET    /api/documents/{id}
POST   /api/documents
PUT    /api/documents/{id}
POST   /api/documents/{id}/file
GET    /api/documents/{id}/file
DELETE /api/documents/{id}
```

## Reglas del contrato

- JSON consistente para respuestas y errores
- paginación explícita en listados
- filtros por tipo y texto
- validación de fechas y campos requeridos
- `multipart/form-data` para carga de archivos
- descarga con `Content-Type` y nombre de archivo correctos
- códigos HTTP previsibles: `200`, `201`, `400`, `401`, `403`, `404`, `409`, `500`

## Modelo de respuesta de listado

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "total": 0
}
```

## Adaptación del legado

La API debe usar un adaptador interno para traducir:

- `DataSet` a DTOs tipados
- nombres legacy a nombres del contrato
- errores SQL o SOAP a errores HTTP
- permisos de `SeguridadVES` a autorización de casos de uso

El frontend no debe conocer procedimientos almacenados ni nombres de esquemas SQL.
