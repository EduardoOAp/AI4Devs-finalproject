# 3. Frontend React + TypeScript

## Tecnología propuesta

- React
- TypeScript con `strict: true`
- Vite para desarrollo y build
- React Router para navegación
- TanStack Query para consultas y mutaciones remotas
- React Hook Form para formularios
- Zod para validar respuestas y entradas
- Vitest y Testing Library para pruebas
- Playwright para flujos principales

## Estructura inicial

```text
src/
  app/
  components/
  features/documents/
    api/
    components/
    pages/
    schemas/
    types/
  features/auth/
  lib/
```

## Pantallas iniciales

- `DocumentListPage`
- `DocumentDetailsPage`
- `DocumentEditorPage`

## Estados obligatorios

Cada pantalla debe contemplar:

- carga
- resultado vacío
- error
- permisos insuficientes
- guardado en progreso
- éxito después de una mutación

## Regla de tipos

Los tipos de frontend deben representar contratos de la API. No deben copiar directamente clases VB.NET ni estructuras `DataSet`.
