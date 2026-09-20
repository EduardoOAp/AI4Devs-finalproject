# 2. Arquitectura objetivo

## Vista general

```text
React + TypeScript
        |
        | HTTPS / JSON
        v
API de documentos
        |
        v
Capa de aplicación
        |
        v
Adaptador de persistencia
        |
        +--> procedimientos existentes
        +--> modelo ORM futuro
        v
SQL Server / almacenamiento de archivos
```

## Componentes

### Frontend

Aplicación React responsable de navegación, formularios, tablas, validaciones de experiencia y estados de carga.

### API

Capa HTTP responsable de autenticación, autorización, validación, paginación, errores y serialización JSON.

### Aplicación

Casos de uso independientes de HTTP y de React:

- `ListDocuments`
- `GetDocument`
- `CreateDocument`
- `UpdateDocument`
- `UploadDocumentFile`
- `DownloadDocument`
- `DeleteDocument`

### Persistencia

Al inicio puede invocar los procedimientos existentes mediante un adaptador. Más adelante puede reemplazarse por repositorios y ORM sin cambiar el frontend.

## Regla de transición

Web Forms y la nueva aplicación pueden coexistir. El módulo nuevo debe consumir la API y no llamar directamente a SOAP ni a SQL Server.
