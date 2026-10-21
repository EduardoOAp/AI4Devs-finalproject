> Detalla en esta sección los prompts principales utilizados durante la creación del proyecto, que justifiquen el uso de asistentes de código en todas las fases del ciclo de vida del desarrollo. Esperamos un máximo de 3 por sección, principalmente los de creación inicial o  los de corrección o adición de funcionalidades que consideres más relevantes.
Puedes añadir adicionalmente la conversación completa como link o archivo adjunto si así lo consideras


## Índice

1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 1. Descripción general del producto

**Prompt 1:**
"Resumir en formato ejecutivo la situación actual de VES: stack, arquitectura, fortalezas, riesgos, puntos de mejora, dependencias, y estrategia recomendada para modernización segura. Usa evidencia del repositorio y no hagas cambios de código."

**Prompt 2:**
"Propón una estrategia de modernización gradual para VES sin romper el sistema actual. Analiza Web Forms, autenticación, servicios SOAP, `DataSet`, `App_Code`, trazabilidad y base de datos. Divide la propuesta en fases, riesgos, beneficios y criterios de aceptación. Todo debe estar basado en evidencia del repositorio y sin implementar cambios."

**Prompt 3:**

---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**

**Prompt 1:**

"Revisa el proyecto VES y documenta la arquitectura actual en ASP.NET Web Forms, con evidencia del repositorio. Enfócate en: stack, runtime, routing, páginas, master pages, App_Code, servicios SOAP, configuración en web.config, autenticación Forms, módulos y permisos. Identifica estructura real, riesgos y patrones observados. No cambies código ni implementes mejoras aún."

**Prompt 2:**

"Analiza la arquitectura actual del proyecto VES y propone una visión de arquitectura futura, manteniendo la compatibilidad actual. Documenta una evolución posible en capas: presentación, casos de uso, servicios de dominio, adaptación a integración y persistencia. No implementes código. Enfócate en migración segura, riesgos."

**Prompt 3:**

### **2.2. Descripción de componentes principales:**

**Prompt 1:**

"Explica el flujo real de las páginas ASP.NET Web Forms del proyecto VES, especialmente ListarDocumentos.aspx.vb, EditorDocumentos.aspx.vb y MasterPage.master.vb. Describe el ciclo de request, cómo se instancian servicios, cómo se consumen los DataSet, cómo se valida y navega, y qué responsabilidades tienen la página, el code-behind y la capa App_Code. Usa evidencia del repositorio y no modifiques código."

**Prompt 2:**
"Documenta el frontend actual de VES como un sistema legado ASP.NET Web Forms. Describe stack, routing, páginas, master pages, controles server-side, CSS, JavaScript, estilos inline, componentes de AjaxControlToolkit y patrón de interacción. Separa claramente convenciones observadas y recomendaciones. No implementes ni cambies UI."

**Prompt 3:**
"Documenta el backend actual de VES con evidencia del repositorio. Explica el stack .NET Framework, servicios SOAP, conexiones SQL Server, uso de App_Code, DataSet, web.config, autenticación y permisos, logging global y flujo de llamadas entre páginas y servicios. Haz una separación clara entre convenciones observadas y recomendaciones. No cambies producción."

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

**Prompt 1:**
"Revisa el módulo de documentos de VES: ListarDocumentos, EditorDocumentos, BajarDocumento y sus servicios asociados. Explica el flujo funcional del documento desde la vista hasta la persistencia, la validación del formulario, el manejo del archivo binario, la conexión con servicios SOAP y una posible evolución hacia un modelo más desacoplado. No cambies código." 

**Prompt 2:**
"Analiza el dominio documental del proyecto VES. Identifica entidades, flujo de documentos, tipos, catálogo, periodo de vida, publicación, caducidad, archivo adjunto y operaciones CRUD. Describe cómo se modela actualmente."

**Prompt 3:**
"Compara la arquitectura legacy actual del proyecto VES con un target state moderno, usando una matriz de capas, responsabilidades, riesgos y beneficios. Incluye Web Forms, code-behind, App_Code, SOAP, DataSet, autenticación y datos. No implementes ni modifiques producción."

### **2.4. Infraestructura y despliegue**

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

### **2.5. Seguridad**

**Prompt 1:**
"Revisa la capa de seguridad del proyecto VES con evidencia de configuración y servicios. Explica cómo se autentica, cómo se obtienen módulos y opciones, cómo se construye el menú dinámico, y qué implicaciones tiene para una modernización futura. No modifiques seguridad ni implementación actual."

**Prompt 2:**
"Documenta la integración de servicios SOAP del proyecto VES. Describe cómo se registran endpoints, cómo se consumen en `App_Code`, qué servicios intervienen en autenticación, documentos, archivos, seguridad y reportes, y qué patrón de integración se está usando."

**Prompt 3:**

### **2.6. Tests**

**Prompt 1:**
"Revisa si el proyecto VES cuenta con pruebas automatizadas, pipelines o validaciones de calidad, y documenta la realidad del repositorio. Señala qué testing existe, qué falta, y cómo sería una estrategia mínima de validación para una próxima evolución sin tocar código productivo."

**Prompt 2:**

**Prompt 3:**

---

### 3. Modelo de Datos

**Prompt 1:**
"Diseña un documento de base de datos para el proyecto VES. Basándote en el repositorio, identifica entidades, relaciones, procedimientos, patrones de persistencia, tablas de documentos y seguridad, tipo de datos y riesgos. También documenta cómo convertir este modelo legacy a un esquema más moderno compatible con Entity Framework o un ORM similar, sin tocar la infraestructura real."

**Prompt 2:**
"Revisa el repositorio VES y clasifica sus áreas por tipo: frontend, backend, servicios, seguridad, base de datos, configuración, assets, dependencias externas y fuera de alcance. Usa la estructura real del proyecto y explica qué partes son clave para una futura modernización."

**Prompt 3:**

---

### 4. Especificación de la API

**Prompt 1:**
"Revisa las dependencias del proyecto VES y clasifícalas por: esenciales, obsoletas, útiles pero peligrosas y fuera de alcance. Haz un análisis basado en el repositorio actual para evitar añadir tecnologías nuevas sin justificación ni romper la compatibilidad del sistema."

**Prompt 2:**
"Evalúa la arquitectura actual de VES en términos de mantenibilidad, escalabilidad, acoplamiento, seguridad y calidad de diseño. Identifica puntos débiles y oportunidades de mejora, con énfasis en documentos, autenticación y servicios. No realices cambios de código."

**Prompt 3:**

---

### 5. Historias de Usuario

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 6. Tickets de Trabajo

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 7. Pull Requests

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**
