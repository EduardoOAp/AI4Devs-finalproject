# Guía de desarrollo VES

Guía de entrada para configurar y trabajar en los tres componentes BCCR de
VES. Las prácticas
que siguen vigentes —mantener credenciales fuera del repositorio, probar los
cambios, usar pasos reproducibles y ejecutar cada servicio de forma aislada—
se conservan con las herramientas reales de VES.


## 0. Cómo usar esta guía

> **Comentario — Adaptado de `development_guide_1.md`:** La guía anterior
> estaba organizada como un recorrido operativo de instalación → configuración
> → base de datos → backend → frontend → pruebas. Se conserva esa intención
> paso a paso, pero se reemplazan completamente las tecnologías y comandos LTI
> por los componentes reales de VES.

Esta guía sirve para:

1. comprobar prerrequisitos;
2. obtener/acceder a los tres proyectos;
3. configurar cada componente de forma segura;
4. iniciar MAPI, API y UI;
5. validar el flujo completo;
6. ejecutar las pruebas correctas;
7. diagnosticar fallos frecuentes sin debilitar seguridad.

No utilizar esta guía como sustituto de los `README.md`, `AGENT.md`, `SPEC.md`
o documentos técnicos específicos de cada repositorio. Cuando exista una
diferencia, prevalece la documentación específica del componente.

### 0.1 Matriz de transformación desde `development_guide_1.md`

| Tema del documento 1 | LTI | Equivalencia VES |
|---|---|---|
| Repositorio | `AI4Devs-LTI-extended` | Repositorios BCCR `Bccr.Supen.Ves.UI`, `Bccr.Supen.Ves.Api`, `Bccr.Supen.Ves.Mapi` |
| Frontend | Node + CRA | React 19 + Vite + Node 24.18.0 |
| Backend | Node/Express | .NET 8 / ASP.NET Core |
| Base de datos | PostgreSQL en Docker | SQL Server de desarrollo accesible por red BCCR |
| ORM | Prisma | EF Core 8 + stored procedures |
| Migraciones | Prisma Migrate | No generar migraciones EF; coordinar cambios de esquema con DBA |
| Secretos | `.env` con credenciales | User Secrets/configuración segura/pipeline; nunca credenciales reales versionadas |
| Backend tests | npm/Jest | MSTest/xUnit según proyecto |
| Frontend tests | Jest/Cypress | Vitest + React Testing Library |
| E2E | Cypress | No introducir Cypress; usar suites actuales |
| Ejecución full stack | frontend/backend local | MAPI → API → UI |
| Configuración frontend | `REACT_APP_*` | `config/{ambiente}.json` + `NODE_CONFIG_ENV` + Vite |
| API frontend | URL `.env` | `Constantes.js` + configuración efectiva de VES |

### 0.2 Regla de seguridad al reutilizar documentación histórica

> **Comentario — Seguridad:** `development_guide_1.md` contiene ejemplos de
> credenciales y cadenas de conexión propios de LTI. **No se trasladan ni deben
> reutilizarse** en VES.

Nunca copiar desde documentación histórica:

- contraseñas;
- usuarios de base de datos;
- cadenas de conexión;
- tokens;
- certificados;
- URLs internas no confirmadas;
- secretos de `.env`.

Usar únicamente los mecanismos y accesos autorizados para VES.


## 1. Arquitectura y responsabilidades

```text
Navegador
   │
   ▼
Bccr.Supen.Ves.UI (React 19 / Vite)
   │ HTTP/HTTPS
   ▼
Bccr.Supen.Ves.Api (.NET 8)
   API Controller → SI.SG → IRequestsHandler
   │ HTTP/HTTPS
   ▼
Bccr.Supen.Ves.Mapi (.NET 8)
   MAPI Controller → BW → DA → EF Core / stored procedures
   │
   ▼
SQL Server
```

Son tres componentes separados, no tres capas de una sola solución:

| Carpeta | Responsabilidad |
|---|---|
| `Bccr.Supen.Ves.UI` | SPA de STI y SEC; consume únicamente la API |
| `Bccr.Supen.Ves.Api` | Orquesta las solicitudes HTTP y llama al MAPI a través de `SI.SG` |
| `Bccr.Supen.Ves.Mapi` | Lógica de negocio (`BW`) y acceso a datos (`DA`) |

La UI no debe llamar directamente a MAPI ni a SQL Server. Los contratos y DTOs
en `Bccr.Supen.Ves.Abstracciones` pertenecen a sus respectivas soluciones; no
son una biblioteca compartida entre las tres carpetas.

## 2. Requisitos y acceso

- Para UI: Node.js **24.18.0**, npm y Git. La versión está fijada en
  `Bccr.Supen.Ves.UI/Codigo/.nvmrc`; el repositorio también incluye Node
  portátil bajo `Codigo/.tools/`.
- Para API y MAPI: .NET 8 SDK. Visual Studio 2022, Rider o VS Code con la
  extensión de C# son opciones de desarrollo.
- Para restaurar paquetes .NET y acceder a los servicios y bases de datos de
  desarrollo puede ser necesaria la red/VPN BCCR y autenticación en los feeds
  NuGet privados.
- Para ejecutar el MAPI se requiere conectividad a los servidores SQL de
  desarrollo. No se necesita Docker ni una base PostgreSQL local para esta
  solución.
- Para depurar la API con IIS Express se requiere el soporte de IIS Express
  provisto por Visual Studio. La ejecución del MAPI también está documentada
  con `dotnet run`.

Confirma los requisitos específicos y las versiones de paquetes en el README,
los `.csproj` y los archivos de configuración del proyecto correspondiente.
No instales herramientas globales o dependencias adicionales sin necesidad.


### 2.1 Preflight antes de instalar o ejecutar

> **Comentario — Adaptado de `development_guide_1.md`:** Se conserva la idea de
> verificar prerrequisitos antes del setup, pero usando las versiones y
> dependencias reales de cada componente.

Antes de continuar, comprobar:

#### UI

```powershell
node --version
npm --version
git --version
```

La versión de Node debe corresponder con:

```text
Bccr.Supen.Ves.UI/Codigo/.nvmrc
```

Actualmente la guía del proyecto fija Node `24.18.0`; confirmar siempre el
archivo del repositorio antes de instalar otra versión.

#### API / MAPI

```powershell
dotnet --version
dotnet --list-sdks
git --version
```

Debe existir un SDK .NET 8 compatible con los `.csproj`.

#### Conectividad

Confirmar, según la tarea:

- acceso a red/VPN BCCR;
- acceso a feeds NuGet privados;
- permisos para SQL Server de desarrollo si se ejecutará MAPI;
- acceso al certificado cliente si el ambiente requiere mTLS;
- acceso al repositorio correspondiente.

### 2.2 No instalar por anticipado

No instalar por defecto:

- Docker;
- PostgreSQL;
- Prisma;
- Cypress;
- herramientas globales npm;
- SDKs adicionales;
- paquetes NuGet externos.

Instalar algo nuevo únicamente si el proyecto actual lo requiere y la
documentación específica lo confirma.


## 3. Obtener los proyectos

En este workspace, los tres directorios ya están disponibles bajo `VES/`.
Cuando se clonen por separado, usa las instrucciones de repositorio y acceso
aprobadas por BCCR que figuran en el README de cada proyecto:

- [UI: README y desarrollo](../Bccr.Supen.Ves.UI/README.md)
- [API: README](../Bccr.Supen.Ves.Api/README.md)
- [MAPI: README](../Bccr.Supen.Ves.Mapi/README.md)

No uses la URL ni las instrucciones de clonación de LTI que aparecían en la
versión previa de esta guía.


### 3.1 Verificación después de obtener el código

Una vez disponible cada repositorio:

1. verificar que se está en la rama esperada;
2. leer el `README.md` y `AGENT.md`;
3. revisar archivos de versión/configuración antes de ejecutar instalaciones;
4. no copiar `.env`, `appsettings` ni configuraciones de otro proyecto;
5. no sustituir configuración BCCR con archivos del proyecto LTI.

Comandos Git útiles para confirmar contexto:

```bash
git status
git branch --show-current
git remote -v
```

Estos comandos son de inspección; no modifican el repositorio.


## 4. Configuración segura

### UI

La configuración por ambiente está en
`Bccr.Supen.Ves.UI/Codigo/config/{ambiente}.json`, combinada con
`config/default.json`. `NODE_CONFIG_ENV` selecciona el ambiente durante
Vite build/dev; `vite.config.mjs` expone la configuración al bundle como
`__APP_CONFIG__`.

- Para desarrollo local, la UI apunta por defecto a
  `http://localhost:62525/` (API local).
- No fijar URLs de API en componentes ni modificar `Constantes.js` para
  cambiar ambientes.
- Todo valor incluido en el bundle del navegador es visible al usuario. Nunca
  colocar credenciales, claves privadas o secretos en `Codigo/config/` ni
  en el `.env` del frontend.
- El repositorio UI documenta un valor heredado `ENCRYPTION_KEY` como pendiente
  de gestión segura; no agregar ni replicar secretos en ese archivo.

### API

La URL del MAPI se configura mediante `ApiEndpoints` y la configuración por
ambiente. En desarrollo puede apuntar al MAPI compartido o a una instancia
local; consulta el README de API y `Codigo/API/appsettings.Development.json`.
No introduzcas secretos en archivos versionados ni cambies tokens que resuelve
el pipeline.

La API requiere el certificado cliente para comunicarse con MAPI en los
ambientes que aplican mTLS. Los decoradores que omiten el certificado son
exclusivos de `Development`; no se deben trasladar a Staging/QA/Producción.

### MAPI y SQL Server

Los valores `#{token}#` de configuración se sustituyen en el despliegue. El
`appsettings.Development.json` contiene configuración de referencia, no debe
contener credenciales personales.

Para desarrollo local, usa **User Secrets** según la guía de MAPI:

```powershell
cd VES\Bccr.Supen.Ves.Mapi\Codigo\MAPI
dotnet user-secrets set "ConnectionStrings:BDSupen" "<cadena de conexión local autorizada>"
```

Configura las otras cadenas de conexión que la operación requiera siguiendo
`Bccr.Supen.Ves.Mapi/README.md`. El identificador de User Secrets está asociado
al proyecto .NET; consulta el `.csproj` antes de asumirlo.

No incluyas contraseñas, tokens, certificados privados ni cadenas de conexión
reales en comandos guardados, Markdown, logs o commits. User Secrets se almacena
en el perfil local y no debe copiarse al repositorio. En integración y
producción las conexiones usan la identidad del proceso/IIS según configuración
de infraestructura.

El esquema SQL Server y los stored procedures los administra DBA. No generar
migraciones EF ni cambiar el esquema como parte de una tarea ordinaria de
aplicación; coordinar cambios con DBA.


### 4.1 Checklist de configuración por componente

> **Comentario — Adaptado de `development_guide_1.md`:** La guía anterior
> pedía crear `.env` para frontend/backend. En VES la misma necesidad de
> configuración por ambiente se resuelve con mecanismos distintos.

#### UI

Antes de iniciar:

- confirmar el ambiente seleccionado por `NODE_CONFIG_ENV`;
- revisar `config/default.json` y `config/{ambiente}.json`;
- comprobar la URL efectiva del API;
- verificar que no se haya introducido un secreto al bundle;
- no editar `Constantes.js` únicamente para “apuntar” a otro ambiente.

#### API

Antes de iniciar:

- revisar `ApiEndpoints`;
- confirmar la URL efectiva del MAPI;
- verificar el ambiente de ASP.NET Core;
- confirmar si se requiere certificado cliente;
- no usar decoradores/bypass de Development en otros ambientes.

#### MAPI

Antes de iniciar:

- confirmar conectividad SQL;
- configurar únicamente las cadenas necesarias mediante User Secrets o el
  mecanismo autorizado;
- revisar los tokens/configuración efectiva;
- no editar stored procedures ni esquema para lograr que el ambiente local
  “funcione”.

### 4.2 Configuración no es secreto

Distinguir:

```text
Configuración:
- host lógico
- timeout
- nombre de endpoint
- feature/config flags no sensibles

Secreto:
- password
- private key
- client secret
- token
- connection string con credenciales
```

Todo valor que llega al navegador es público. En backend, los secretos deben
permanecer fuera del repositorio y resolverse mediante los mecanismos
autorizados.


## 5. Ejecutar localmente

Los servicios se pueden iniciar independientemente. Para probar el flujo
completo con instancias locales, iniciar primero MAPI, después API y por último
la UI. También se puede apuntar API al MAPI compartido de desarrollo.


### 5.0 Secuencia recomendada para un entorno totalmente local

> **Comentario — Adaptado de `development_guide_1.md`:** Igual que la guía LTI
> separaba database/backend/frontend, VES debe iniciarse respetando las
> dependencias reales entre servicios.

Para ejecutar los tres componentes localmente:

```text
1. MAPI
   ↓
2. API
   ↓
3. UI
```

Motivo:

- MAPI necesita acceso a SQL Server;
- API necesita un MAPI accesible;
- UI necesita un API accesible.

No es obligatorio ejecutar los tres localmente. El API puede apuntar a un MAPI
compartido de desarrollo cuando el procedimiento del equipo lo permita.

### 5.0.1 Checkpoint antes de iniciar el siguiente componente

Antes de avanzar:

**MAPI**
- compiló;
- inició sin error de configuración;
- escucha en el perfil/puerto efectivo;
- tiene conectividad requerida a SQL Server.

**API**
- compiló;
- inició;
- `ApiEndpoints` apunta al MAPI correcto;
- el health check responde;
- mTLS/configuración corresponden al ambiente.

**UI**
- dependencias instaladas;
- Vite inicia;
- la configuración efectiva apunta al API esperado;
- el navegador puede alcanzar el API.

No asumir el puerto únicamente por esta guía. Confirmar siempre la salida del
proceso, `launchSettings.json` y la configuración efectiva.


### 5.1 MAPI

Desde el repositorio/carpeta MAPI:

```powershell
cd VES\Bccr.Supen.Ves.Mapi\Codigo
dotnet restore Bccr.Supen.Ves.Mapi.sln
dotnet build Bccr.Supen.Ves.Mapi.sln --no-restore
cd MAPI
dotnet run
```

Según el README del MAPI, `dotnet run` escucha normalmente en
`https://localhost:5001` y `http://localhost:5000`; confirma el puerto en la
salida local y `Properties/launchSettings.json`.

### 5.2 API de orquestación

Restaura y compila la solución:

```powershell
cd VES\Bccr.Supen.Ves.Api\Codigo
dotnet restore Bccr.Supen.Ves.Api.sln
dotnet build Bccr.Supen.Ves.Api.sln
```

En Visual Studio, ejecuta el perfil `IIS-development` documentado por el
proyecto. La URL local indicada es `https://localhost:44366`. Asegúrate de que
`ApiEndpoints` apunta a la instancia local de MAPI o al servicio compartido.
También se puede ejecutar el proyecto `API` con `dotnet run` usando el perfil
y configuración de desarrollo correctos.

Health check del API:

```text
GET https://localhost:44366/healthcheck?api-version=1
```

### 5.3 UI

Desde la carpeta `Codigo` de UI:

```powershell
cd VES\Bccr.Supen.Ves.UI\Codigo
npm install
npm start
```

Vite sirve la UI en `http://localhost:3000`. En su configuración
`localdev`, la UI apunta a `http://localhost:62525/`. Ejecuta el API en esa
dirección o configura la UI según el procedimiento de desarrollo documentado
por el equipo.

Para crear un build local sin cambiar la versión:

```powershell
npm run build:local
```

Los scripts `build:pruebas` y `build:produccion` incrementan la versión de
`package.json`; no los ejecutes como validación rutinaria local.


### 5.4 Verificación manual mínima del flujo

Después de iniciar los componentes necesarios:

1. comprobar que MAPI inició correctamente;
2. comprobar el health check del API;
3. abrir la UI;
4. ejecutar únicamente un flujo funcional seguro y conocido;
5. revisar errores de red/consola;
6. confirmar que la UI llama al API y no directamente a MAPI;
7. confirmar que no se muestran datos sensibles en errores.

La validación manual complementa, pero no sustituye, las pruebas automatizadas.

### 5.5 Puertos y perfiles: evitar supuestos

Esta guía documenta varios perfiles/URLs porque los componentes pueden ejecutarse
con mecanismos diferentes (por ejemplo IIS Express, `dotnet run` o configuración
`localdev`).

Por ello:

- no cambiar código para hacer coincidir un puerto “esperado”;
- confirmar el puerto real en la salida de ejecución;
- revisar `launchSettings.json`, `ApiEndpoints` y configuración Vite;
- alinear la configuración siguiendo el procedimiento del equipo.


## 6. Pruebas y verificaciones

Ejecuta las verificaciones de la carpeta que modificaste.


### 6.0 Matriz rápida de validación

> **Comentario — Adaptado de `development_guide_1.md`:** Se conserva la
> separación de pruebas por componente, sustituyendo npm/Jest/Cypress del
> proyecto LTI por las suites reales de VES.

| Cambio realizado | Validación mínima |
|---|---|
| Solo UI | `npm run lint` + `npm test` + `npm run build:local` |
| Solo API | `dotnet build` + `dotnet test` de la solución API |
| Solo MAPI | `dotnet build` + `dotnet test` de la solución MAPI |
| API ↔ MAPI | Pruebas de ambos componentes + verificación del contrato |
| UI ↔ API | Pruebas UI/API pertinentes + verificación de ruta/configuración |
| UI ↔ API ↔ MAPI | Validaciones de los tres + flujo integrado controlado |
| Cambio SQL/SP | Validación de aplicación + coordinación/validación DBA |

No ejecutar herramientas de otro proyecto para “cumplir” una checklist:
Cypress, Prisma o Jest no forman parte del flujo actual de VES.


### UI

Desde `Bccr.Supen.Ves.UI/Codigo/`:

```powershell
npm run lint
npm test
npm run build:local
```

Las pruebas usan Vitest y React Testing Library. No se usa Jest ni Cypress en
el stack actual.

### API

Desde `Bccr.Supen.Ves.Api/Codigo/`:

```powershell
dotnet build Bccr.Supen.Ves.Api.sln
dotnet test Bccr.Supen.Ves.Api.sln
```

La suite contiene pruebas unitarias MSTest y pruebas de integración xUnit;
consulta los `.csproj` para ejecutar un proyecto específico.

### MAPI

Desde `Bccr.Supen.Ves.Mapi/Codigo/`:

```powershell
dotnet build Bccr.Supen.Ves.Mapi.sln --no-restore
dotnet test Bccr.Supen.Ves.Mapi.sln
dotnet test Bccr.Supen.Ves.Mapi.sln --collect:"XPlat Code Coverage"
```

Las pruebas por capa se encuentran en `DA.Pruebas`, `BW.Pruebas` y
`MAPI.UnitTests`, con MSTest y Moq/NSubstitute según los patrones existentes.
Las pruebas unitarias no deben depender de bases de datos reales, feeds
privados, certificados de producción ni servicios externos.

### Cambios que atraviesan más de un componente

Si se cambia el contrato API↔MAPI, verifica la ruta configurada, método HTTP,
versión, parámetros, serialización/DTO, autenticación y mTLS en ambos extremos.
Si el cambio también afecta la UI, comprueba que use la URL/configuración de
ambiente correcta y que gestione carga y error.


### 6.1 Qué significa que una validación pase

Un comando exitoso no basta si:

- aparecen warnings nuevos relevantes;
- una prueba fue omitida para hacer pasar la suite;
- se modificó un contrato sin actualizar consumidores;
- un test unitario requiere infraestructura externa inesperadamente;
- el build utilizado incrementó versiones o produjo cambios colaterales.

Después de ejecutar las pruebas:

```bash
git status
git diff
```

o el equivalente en la herramienta Git utilizada, para comprobar que la
validación no alteró archivos inesperadamente.

### 6.2 Pruebas locales sin infraestructura productiva

Las pruebas unitarias no deben necesitar:

- SQL Server de producción;
- certificados productivos;
- credenciales reales;
- red externa no controlada;
- feeds adicionales no autorizados.

Las suites de integración pueden tener otros requisitos; verificar su README y
configuración antes de interpretarlas como pruebas unitarias fallidas.

### 6.3 Cobertura

El documento LTI utilizaba un comando `npm run test:coverage` para backend.
Eso no se traslada literalmente.

En VES:

- API/MAPI: usar las opciones y Quality Gate del proyecto;
- MAPI documenta `--collect:"XPlat Code Coverage"`;
- UI: respetar la configuración Vitest/pipeline existente;
- no inventar un porcentaje universal de cobertura si el repositorio no lo
  establece.


## 7. Prácticas de trabajo conservadas

- Mantener los cambios acotados a la tarea y respetar las capas responsables.
- Escribir o actualizar pruebas para el comportamiento nuevo y regresiones
  relevantes, siguiendo el framework del proyecto.
- Validar entradas en el servidor y preservar errores útiles; no ocultar fallos
  con respuestas de éxito vacías.
- Usar SQL parametrizado; nunca concatenar datos de entrada para formar SQL.
- Mantener secretos fuera del repositorio y del bundle de navegador.
- No duplicar reglas de negocio entre API y MAPI ni añadir capas o dependencias
  sin una necesidad concreta.
- Antes de desplegar, seguir el pipeline y el procedimiento del repositorio y
  ambiente correspondiente. Los builds locales no sustituyen autorización de
  despliegue.

Para convenciones y arquitectura detalladas, consultar
[base-standards.md](./base-standards.md),
[frontend-standards.md](./frontend-standards.md) y
[backend-standards.md](./backend-standards.md).


### 7.1 Flujo de trabajo recomendado

> **Comentario — Adaptado de `development_guide_1.md`:** La guía anterior era
> estrictamente secuencial. Se conserva ese valor operativo con un flujo que
> funciona para cualquiera de los tres componentes.

```text
1. Identificar componente
        ↓
2. Leer estándar/README
        ↓
3. Verificar ambiente
        ↓
4. Hacer cambio mínimo
        ↓
5. Compilar/lint
        ↓
6. Ejecutar pruebas
        ↓
7. Validar integración si aplica
        ↓
8. Revisar git diff/status
        ↓
9. Documentar cambios relevantes
```

Para cambios complejos, seguir además la estrategia de planificación/modelos
definida en `base-standards.md`.

### 7.2 No “arreglar” el ambiente debilitando controles

Si una ejecución falla por:

- autenticación;
- certificado;
- CORS;
- permisos;
- conexión SQL;
- feed privado;
- configuración de ambiente;

investigar primero la configuración y el acceso correcto.

No resolver un problema local mediante:

- `[AllowAnonymous]` no requerido;
- desactivar validación TLS;
- omitir mTLS fuera de Development;
- ampliar CORS indiscriminadamente;
- hardcodear credenciales;
- cambiar permisos globales;
- apuntar a una base no autorizada.


## 8. Resolución de problemas

| Síntoma | Comprobaciones |
|---|---|
| `dotnet restore` no encuentra paquetes | Conectividad/VPN BCCR, autenticación NuGet y fuentes privadas configuradas. No añadir feeds no confiables. |
| API no encuentra un endpoint MAPI | Revisar `ApiEndpoints`, nombre exacto del endpoint, URL del MAPI y el archivo de configuración efectivo para `Development`. |
| Error de certificado al llamar al MAPI | Confirmar ambiente y configuración mTLS; no desactivar certificados fuera de Development. |
| Error de conexión SQL local | Confirmar red/permisos, cadenas de conexión requeridas y User Secrets; nunca copiar credenciales al repositorio. |
| Error de límite de carga `404.13` | Revisar que límites de IIS y de ASP.NET Core/API estén alineados; consultar README y guía de deploy de API. |
| UI no conecta al API | Confirmar que el API está activo en el host/puerto configurado para `localdev` y revisar consola/red del navegador. |
| Pruebas API requieren servicios externos | Ejecutar el proyecto de pruebas apropiado; pruebas unitarias deben sustituir dependencias externas. Verificar si se está intentando correr una suite de integración. |

No solventar fallos de ambiente relajando autenticación, permisos, CORS o
validaciones sin aprobación y análisis del impacto.

### 8.1 Diagnóstico por capas

Cuando el flujo completo falla, aislar el problema de abajo hacia arriba:

```text
SQL Server
   ↑
MAPI
   ↑
API
   ↑
UI
```

Ejemplos:

- si MAPI no puede acceder a SQL Server, no diagnosticar primero React;
- si API no puede alcanzar MAPI, comprobar `ApiEndpoints` antes de modificar UI;
- si el health check de API responde pero UI falla, revisar configuración,
  navegador y solicitud HTTP de la UI.

### 8.2 Información útil al reportar un fallo

Registrar sin secretos:

- componente;
- comando ejecutado;
- ambiente/perfil;
- mensaje de error;
- endpoint/ruta lógica afectada;
- etapa donde ocurre;
- si build/tests pasan;
- cambio reciente relacionado.

No incluir:

- contraseñas;
- tokens;
- connection strings reales;
- contenido de certificados privados;
- datos sensibles.

## 9. Instrucciones de `development_guide_1.md` que NO aplican a VES

> **Comentario — Comparación con `development_guide_1.md`:** Estas instrucciones
> pertenecían al sistema LTI y no deben utilizarse para configurar VES.

### No usar la clonación LTI

No usar:

```text
git@github.com:LIDR-academy/AI4Devs-LTI-extended.git
```

para obtener VES.

Seguir las instrucciones BCCR de cada repositorio.

### No usar credenciales LTI

No copiar usuarios, contraseñas ni `DATABASE_URL` mostrados en la guía antigua.
Aunque aparezcan como ejemplos históricos, deben tratarse como datos no
reutilizables.

### No crear `.env` al estilo LTI

No crear automáticamente:

```text
backend/.env
frontend/.env
REACT_APP_API_URL
DATABASE_URL
```

VES utiliza mecanismos distintos por componente.

### No iniciar PostgreSQL con Docker

No ejecutar como requisito VES:

```bash
docker-compose up -d
docker-compose ps
```

La base de datos de VES es SQL Server y el acceso de desarrollo se realiza según
infraestructura BCCR.

### No usar Prisma

No ejecutar:

```bash
npm run prisma:generate
npx prisma migrate deploy
npx prisma db seed
```

Los backends VES son .NET/EF Core y el esquema/stored procedures se administran
con DBA.

### No ejecutar backend mediante npm

No usar para API/MAPI:

```bash
npm run dev
npm test
npm run test:watch
npm run test:coverage
```

Usar los comandos .NET documentados por cada solución.

### No introducir Cypress

No ejecutar ni instalar como parte del setup VES:

```bash
npm run cypress:open
npm run cypress:run
```

La UI utiliza Vitest + React Testing Library.

### No asumir versiones LTI de Node/npm

La versión válida para UI es la indicada por `.nvmrc` del repositorio VES, no
el mínimo Node 16/npm 8 del documento anterior.

### No asumir los puertos LTI

Los puertos `3000`/`3001` del sistema LTI no definen la topología VES.
Confirmar perfiles y configuración real de cada componente.

## 10. Quick start de desarrollo

### Solo UI

```powershell
cd VES\Bccr.Supen.Ves.UI\Codigo
npm install
npm start
```

Validar:

```powershell
npm run lint
npm test
npm run build:local
```

### Solo API

```powershell
cd VES\Bccr.Supen.Ves.Api\Codigo
dotnet restore Bccr.Supen.Ves.Api.sln
dotnet build Bccr.Supen.Ves.Api.sln
dotnet test Bccr.Supen.Ves.Api.sln
```

Confirmar el perfil/configuración antes de ejecutar el host.

### Solo MAPI

```powershell
cd VES\Bccr.Supen.Ves.Mapi\Codigo
dotnet restore Bccr.Supen.Ves.Mapi.sln
dotnet build Bccr.Supen.Ves.Mapi.sln --no-restore
dotnet test Bccr.Supen.Ves.Mapi.sln

cd MAPI
dotnet run
```

MAPI requiere conectividad/configuración SQL adecuada.

### Flujo completo local

```text
1. Configurar MAPI
2. Iniciar MAPI
3. Configurar API → MAPI
4. Iniciar API
5. Verificar health check
6. Configurar UI → API
7. Iniciar UI
8. Probar flujo funcional
9. Ejecutar pruebas de cada componente modificado
```

## 11. Checklist final del ambiente de desarrollo

### Prerrequisitos

- [ ] Node coincide con `.nvmrc` para UI.
- [ ] .NET 8 SDK disponible para API/MAPI.
- [ ] Git disponible.
- [ ] VPN/red BCCR disponible cuando aplica.
- [ ] Feeds NuGet privados accesibles cuando aplica.
- [ ] Acceso SQL autorizado para MAPI cuando aplica.

### Configuración

- [ ] UI usa ambiente/configuración correcta.
- [ ] API apunta al MAPI correcto.
- [ ] MAPI tiene únicamente las conexiones autorizadas necesarias.
- [ ] No hay secretos nuevos en archivos versionados.
- [ ] mTLS/certificados corresponden al ambiente.

### Ejecución

- [ ] MAPI inicia correctamente si es necesario.
- [ ] API inicia y health check responde.
- [ ] UI inicia con Vite.
- [ ] UI consume API, no MAPI.
- [ ] Puertos efectivos fueron confirmados, no asumidos.

### Pruebas

- [ ] UI: lint + Vitest + build local, si se modificó UI.
- [ ] API: build + tests, si se modificó API.
- [ ] MAPI: build + tests, si se modificó MAPI.
- [ ] Contratos cruzados se verificaron en ambos lados.
- [ ] Cambios SQL se coordinaron con DBA.
- [ ] No se introdujeron Jest/Cypress/Prisma/Docker por herencia del proyecto LTI.

### Seguridad

- [ ] Sin passwords/tokens/cadenas reales en commits.
- [ ] Sin bypass de autenticación fuera de Development.
- [ ] Sin relajación de CORS/TLS/mTLS para resolver un fallo local.
- [ ] Sin datos sensibles en logs o capturas de error.

### Antes de cerrar

- [ ] `git status` no muestra cambios inesperados.
- [ ] Se revisó `git diff`.
- [ ] Documentación afectada permanece alineada.
- [ ] Se siguieron `base-standards.md`, `frontend-standards.md` y/o
      `backend-standards.md` según el componente.

