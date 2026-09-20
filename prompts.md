# Prompts de referencia para VES

Este archivo centraliza prompts reutilizables y contextualizados para continuar con análisis, documentación y propuestas de mejora del proyecto VES sin cambiar código productivo.

---

## 1) Prompt base para análisis de arquitectura actual

Prompt:

"Revisa el proyecto VES y documenta la arquitectura actual en ASP.NET Web Forms, con evidencia del repositorio. Enfócate en: stack, runtime, routing, páginas, master pages, App_Code, servicios SOAP, configuración en web.config, autenticación Forms, módulos y permisos. Identifica estructura real, riesgos y patrones observados. No cambies código ni implementes mejoras aún."

---

## 2) Prompt para analizar flujo de los .aspx.vb

Prompt:

"Explica el flujo real de las páginas ASP.NET Web Forms del proyecto VES, especialmente ListarDocumentos.aspx.vb, EditorDocumentos.aspx.vb y MasterPage.master.vb. Describe el ciclo de request, cómo se instancian servicios, cómo se consumen los DataSet, cómo se valida y navega, y qué responsabilidades tienen la página, el code-behind y la capa App_Code. Usa evidencia del repositorio y no modifiques código."

---

## 3) Prompt para documentación de frontend

Prompt:

"Documenta el frontend actual de VES como un sistema legado ASP.NET Web Forms. Describe stack, routing, páginas, master pages, controles server-side, CSS, JavaScript, estilos inline, componentes de AjaxControlToolkit y patrón de interacción. Separa claramente convenciones observadas y recomendaciones. No implementes ni cambies UI."

---

## 4) Prompt para documentación de backend

Prompt:

"Documenta el backend actual de VES con evidencia del repositorio. Explica el stack .NET Framework, servicios SOAP, conexiones SQL Server, uso de App_Code, DataSet, web.config, autenticación y permisos, logging global y flujo de llamadas entre páginas y servicios. Haz una separación clara entre convenciones observadas y recomendaciones. No cambies producción."

---

## 5) Prompt para diseño de base de datos y ORM

Prompt:

"Diseña un documento de base de datos para el proyecto VES con enfoque de compatibilidad futura con ORM. Basándote en el repositorio, identifica entidades, relaciones, procedimientos, patrones de persistencia, tablas de documentos y seguridad, tipo de datos y riesgos. También documenta cómo convertir este modelo legacy a un esquema más moderno compatible con Entity Framework o un ORM similar, sin tocar la infraestructura real."

---

## 6) Prompt para generar arquitectura futura

Prompt:

"Analiza la arquitectura actual del proyecto VES y propone una visión de arquitectura futura, manteniendo la compatibilidad actual. Documenta una evolución posible en capas: presentación, casos de uso, servicios de dominio, adaptación a integración y persistencia. No implementes código. Enfócate en migración segura, riesgos, capacidades nuevas y orden de modernización."

---

## 7) Prompt para revisar módulos de documentos

Prompt:

"Revisa el módulo de documentos de VES: ListarDocumentos, EditorDocumentos, BajarDocumento y sus servicios asociados. Explica el flujo funcional del documento desde la vista hasta la persistencia, la validación del formulario, el manejo del archivo binario, la conexión con servicios SOAP y una posible evolución hacia un modelo más desacoplado. No cambies código." 

---

## 8) Prompt para análisis de seguridad y permisos

Prompt:

"Revisa la capa de seguridad del proyecto VES con evidencia de configuración y servicios. Explica cómo se autentica, cómo se obtienen módulos y opciones, cómo se construye el menú dinámico, y qué implicaciones tiene para una modernización futura. No modifiques seguridad ni implementación actual."

---

## 9) Prompt para análisis de pruebas y calidad

Prompt:

"Revisa si el proyecto VES cuenta con pruebas automatizadas, pipelines o validaciones de calidad, y documenta la realidad del repositorio. Señala qué testing existe, qué falta, y cómo sería una estrategia mínima de validación para una próxima evolución sin tocar código productivo."

---

## 10) Prompt para propuesta de modernización gradual

Prompt:

"Propón una estrategia de modernización gradual para VES sin romper el sistema actual. Analiza Web Forms, autenticación, servicios SOAP, `DataSet`, `App_Code`, trazabilidad y base de datos. Divide la propuesta en fases, riesgos, beneficios y criterios de aceptación. Todo debe estar basado en evidencia del repositorio y sin implementar cambios."

---

## 11) Prompt para revisión de dependencias y fuera de alcance

Prompt:

"Revisa las dependencias del proyecto VES y clasifícalas por: esenciales, obsoletas, útiles pero peligrosas y fuera de alcance. Haz un análisis basado en el repositorio actual para evitar añadir tecnologías nuevas sin justificación ni romper la compatibilidad del sistema."

---

## 12) Prompt para preparar una propuesta de arquitectura actualizada

Prompt:

"Genera una propuesta de arquitectura actualizada para VES a partir del análisis del repositorio. Debe incluir: objetivos, principios, capas, flujo principal, límites entre UI y backend, integración de servicios, base de datos y estrategia de migración incremental. El documento debe ser claro para stakeholders y técnicos, sin tocar la aplicación actual."

---

## 13) Prompt para extraer un resumen ejecutivo

Prompt:

"Resumir en formato ejecutivo la situación actual de VES: stack, arquitectura, fortalezas, riesgos, puntos de mejora, dependencias, y estrategia recomendada para modernización segura. Usa evidencia del repositorio y no hagas cambios de código."

---

## 14) Prompt para análisis de dominio de documentos

Prompt:

"Analiza el dominio documental del proyecto VES. Identifica entidades, flujo de documentos, tipos, catálogo, periodo de vida, publicación, caducidad, archivo adjunto y operaciones CRUD. Describe cómo se modela actualmente y qué cambiaría en un diseño más moderno basado en un ORM."

---

## 15) Prompt para comparar arquitectura legacy vs target state

Prompt:

"Compara la arquitectura legacy actual del proyecto VES con un target state moderno, usando una matriz de capas, responsabilidades, riesgos y beneficios. Incluye Web Forms, code-behind, App_Code, SOAP, DataSet, autenticación y datos. No implementes ni modifiques producción."

---

## 16) Prompt para documentación de integración de servicios

Prompt:

"Documenta la integración de servicios SOAP del proyecto VES. Describe cómo se registran endpoints, cómo se consumen en `App_Code`, qué servicios intervienen en autenticación, documentos, archivos, seguridad y reportes, y qué patrón de integración se está usando."

---

## 17) Prompt para análisis de repositorio y alcance

Prompt:

"Revisa el repositorio VES y clasifica sus áreas por tipo: frontend, backend, servicios, seguridad, base de datos, configuración, assets, dependencias externas y fuera de alcance. Usa la estructura real del proyecto y explica qué partes son clave para una futura modernización."

---

## 18) Prompt para preparar una propuesta técnica para stakeholders

Prompt:

"Prepara una propuesta técnica para VES orientada a stakeholders, explicando la situación actual, el problema de mantenimiento, la estrategia de modernización y la propuesta de evolución arquitectónica. El documento debe ser claro, accionable y basado solo en evidencia del repositorio, sin cambios de implementación."

---

## 19) Prompt para revisar aspectos de calidad y escalabilidad

Prompt:

"Evalúa la arquitectura actual de VES en términos de mantenibilidad, escalabilidad, acoplamiento, seguridad y calidad de diseño. Identifica puntos débiles y oportunidades de mejora, con énfasis en documentos, autenticación y servicios. No realices cambios de código."

---

## 20) Prompt para preparar una hoja de ruta de modernización

Prompt:

"Diseña una hoja de ruta para modernizar el proyecto VES de manera incremental. Ordena tareas por impacto, riesgo y valor, incluye estratos de frontend, backend, base de datos, servicios y seguridad, y describe cómo evitar interrupciones durante la transición."

---

## Uso recomendado

Puedes reutilizar estos prompts como base para:
- análisis del código
- generación de documentación
- propuestas de arquitectura
- revisión de módulos y dominio
- preparación de sesiones con equipo técnico o stakeholders

La idea es mantener la documentación en un formato consistente y reutilizable sin cambiar la aplicación actual.
