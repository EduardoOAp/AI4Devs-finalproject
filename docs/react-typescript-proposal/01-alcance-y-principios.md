# 1. Alcance y principios

## Problema actual

El módulo de documentos depende de páginas Web Forms, eventos de servidor, `DataSet`, proxies SOAP y servicios que acceden directamente a SQL Server.

## Primera capacidad a modernizar

El primer vertical debe ser **consulta y gestión básica de documentos**:

1. listar documentos
2. filtrar por tipo
3. consultar el detalle
4. crear o editar metadatos
5. cargar un archivo
6. descargar un archivo
7. eliminar un documento según permisos

## Principios

- migración incremental, no reemplazo masivo
- API como límite entre frontend y backend
- TypeScript estricto
- contratos y modelos tipados
- autorización en backend, no solo en la interfaz
- conservar inicialmente las operaciones y reglas existentes
- cada paso debe poder validarse de forma independiente

## Decisiones pendientes

- mecanismo final de autenticación para la SPA
- ubicación del binario: base de datos o almacenamiento de archivos
- framework backend para la API
- estrategia definitiva de migración de procedimientos almacenados
