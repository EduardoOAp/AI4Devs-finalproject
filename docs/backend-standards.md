---
description: Estándares de backend para Bccr.Supen.Ves.Api y Bccr.Supen.Ves.Mapi dentro de la solución VES
globs: ["VES/Bccr.Supen.Ves.Api/**/*.cs", "VES/Bccr.Supen.Ves.Mapi/**/*.cs", "VES/Bccr.Supen.Ves.Api/**/*.csproj", "VES/Bccr.Supen.Ves.Mapi/**/*.csproj"]
alwaysApply: false
---

# Estándares de backend VES

## Alcance y arquitectura actual

Este documento aplica a los dos backends .NET del conjunto VES. No aplica a la
UI React, ni describe un backend Node.js:

```text
Navegador / Bccr.Supen.Ves.UI
             │ HTTP/HTTPS
             ▼
Bccr.Supen.Ves.Api
API Controller → SI.SG → IRequestsHandler → MAPI
                                           │ HTTP/HTTPS
                                           ▼
Bccr.Supen.Ves.Mapi
MAPI Controller → BW → DA → SQL Server
```

| Proyecto | Responsabilidad | Stack confirmado |
|---|---|---|
| `Bccr.Supen.Ves.Api` | API HTTP de orquestación consumida por la UI; `SI.SG` implementa el gateway HTTP al MAPI | .NET 8 / ASP.NET Core, `IRequestsHandler`, contratos y DTOs en `Bccr.Supen.Ves.Abstracciones` |
| `Bccr.Supen.Ves.Mapi` | Servicio con controladores, lógica de negocio y acceso a datos | .NET 8 / ASP.NET Core, EF Core 8, SQL Server y stored procedures |

Estas carpetas son dos servicios desplegables separados, no las capas de una
única aplicación. `Abstracciones` existe dentro de cada solución .NET para sus
interfaces y modelos; no es una capa compartida entre API y MAPI. Para el mapa
con UI, consultar [base-standards.md](./base-standards.md).

El documento anterior describía otra aplicación (LTI) con Node.js, TypeScript,
Express, Prisma, PostgreSQL, DDD, Jest y AWS Lambda. Esas tecnologías,
estructuras y pasos de despliegue **no corresponden** a los backends BCCR y no
deben imponerse. DDD, repositorios genéricos, migraciones, formatos de rutas
REST nuevos o cambios de contrato no son requisitos por sí mismos: seguir la
arquitectura y los contratos ya establecidos en el proyecto que se modifica.

Las buenas prácticas que sí se conservan de los estándares anteriores son:
responsabilidades claras, dependencias invertidas mediante contratos cuando
el proyecto ya los usa, SOLID/DRY con criterio, validación de entradas,
operaciones asíncronas correctas, errores explícitos, seguridad por defecto,
pruebas del comportamiento, cobertura de casos límite y revisión antes de
fusionar. Se expresan abajo con .NET y los patrones reales de VES.

## Stack y convenciones del repositorio

- Runtime y framework: .NET 8 / ASP.NET Core 8.
- MAPI: EF Core 8 (paquetes actualmente centralizados en
  `Bccr.Supen.Ves.Mapi/Codigo/Directory.Packages.props`) y SQL Server.
- MAPI: el esquema y los stored procedures los administra DBA; no agregar
  migraciones EF ni alterar objetos de base de datos desde el código.
- API y MAPI: conservar paquetes BCCR, convenciones de DI, middleware,
  autenticación, versionamiento y configuración ya presentes. No sustituir
  dependencias institucionales por implementaciones propias sin requerimiento.
- Código y documentación de los proyectos VES usan principalmente español.
  Mantener nombres, contratos, mensajes y convenciones existentes; no traducir
  ni renombrar código público por una regla genérica de idioma.
- Seguir el formato y el estilo C# del archivo vecino. Los nombres de clases,
  interfaces, métodos públicos y DTOs siguen PascalCase; no cambiar
  identificadores de dominio existentes (muchos están en español).
- Mantener nullable annotations y tipos definidos por el proyecto. Evitar
  `dynamic`, `object` o supresiones de nullabilidad que oculten errores cuando
  haya un tipo o una validación adecuada.
- Consultar `.csproj`, `Directory.Packages.props` y los documentos de cada
  servicio antes de asumir versiones o herramientas. No fijar versiones aquí
  si ya están centralizadas en el proyecto.

- En VES esta práctica se traduce a conservar `nullable` habilitado, usar tipos
concretos y contratos C# explícitos, y evitar `dynamic`, `object`, casts o
supresiones de nullabilidad cuando un tipo específico pueda expresar mejor el
contrato.

- Mantener el código legible como parte del estándar: nombres
descriptivos, métodos acotados, comentarios solo cuando expliquen decisiones o
casos no evidentes, y documentación técnica para condiciones especiales. No
agregar comentarios que repitan literalmente lo que el código ya expresa.

Referencias operativas y específicas:

- API: [AGENT.md](..\VES\Bccr.Supen.Ves.Api\AGENT.md),
  [SPEC.md](..\VES\Bccr.Supen.Ves.Api\SPEC.md),
  [endpoint guide](..\VES\Bccr.Supen.Ves.Api\Documentacion\ComoCrearUnNuevoEndpoint.md) y
  [Deploy guide](..\VES\Bccr.Supen.Ves.Api\Documentacion\Deploy.md).

- MAPI: [README](/VES/Bccr.Supen.Ves.Mapi/README.md),
  [AGENT.md](/VES/Bccr.Supen.Ves.Mapi/AGENT.md),
  [Implementation guide](..\VES\Bccr.Supen.Ves.Mapi\Documentacion\ComoImplementarUnMAPI.md),
  [environment verification guide](..\VES\Bccr.Supen.Ves.Mapi\Documentacion\VerificacionEntorno.md) y
  y [SCHEMA.md](/VES/Bccr.Supen.Ves.Mapi/Documentacion/SCHEMA.md).


## Convenciones de escritura C# y nomenclatura


### Matriz de convenciones: documento 1 → VES/.NET

> El documento 1 define convenciones concretas para variables, funciones, clases,
> interfaces, constantes, tipos y archivos. Todas se conservan aquí como
> intención normativa, adaptadas a las convenciones idiomáticas de C#/.NET y al
> código existente de VES.

| Elemento | Convención en documento 1 | Equivalencia adoptada en VES/.NET | Ejemplo VES |
|---|---|---|---|
| Variables locales | `camelCase` | `camelCase` | `codigoEntidad` |
| Parámetros | `camelCase` | `camelCase` | `codigoFondo` |
| Funciones | `camelCase` | Métodos C# en `PascalCase` | `ObtengaFondoAsync` |
| Métodos públicos | implícitamente `camelCase` en TS | `PascalCase` | `ActualiceRentabilidadAsync` |
| Métodos privados | implícitamente `camelCase` en TS | `PascalCase` por convención .NET/proyecto | `ValideSolicitud` |
| Clases | `PascalCase` | `PascalCase` | `FondoBW` |
| Interfaces | `PascalCase` | `I` + `PascalCase` | `IFondoDA` |
| Tipos | `PascalCase` | `PascalCase` | `FondoSolicitud` |
| Constantes | `UPPER_SNAKE_CASE` | `PascalCase` para `const`/`static readonly` salvo convención local distinta | `CantidadMaximaResultados` |
| Archivos | `camelCase` | `PascalCase` y normalmente coincidente con el tipo principal, según estilo C# del repositorio | `FondoBW.cs` |
| Propiedades públicas | no detallado | `PascalCase` | `CodigoEntidad` |
| Campos privados | no detallado | `_camelCase` | `_fondoDA` |
| Booleanos | no detallado | nombre afirmativo/descriptivo | `tieneAutorizacion` |
| Métodos asíncronos | `async`/Promise | sufijo `Async` cuando la convención del proyecto lo use | `ObtengaFondosAsync` |

La columna “Equivalencia adoptada” es la regla que deben seguir los cambios en
VES. La columna del documento 1 se conserva para mostrar el origen de la buena
práctica y evitar trasladar mecánicamente convenciones propias de TypeScript a
C#.

### Convenciones de archivos

> El documento exige nombres de archivo consistentes (`candidateService.ts`,
> `candidateController.ts`). En C#, el mismo principio se aplica usando la
> convención del ecosistema y del repositorio: el archivo debe reflejar el tipo
> principal que contiene.

**Evitar**

```text
fondo_bw.cs
fondobw.cs
fondoService.cs
archivo1.cs
```

**Recomendado**

```text
FondoBW.cs
IFondoBW.cs
FondoDA.cs
IFondoDA.cs
FondoController.cs
FondoSolicitud.cs
```

Cuando un archivo existente no siga esta convención, no renombrarlo como cambio
colateral. Aplicar la regla principalmente a archivos nuevos o a refactorizaciones
que incluyan explícitamente el cambio de nombres.

### Idioma de identificadores, comentarios, errores y logs

> VES ya usa principalmente español en su código de dominio, contratos y
> documentación. Por tanto, **no se traslada literalmente la regla “todo en
> inglés”**; se conserva su objetivo real: usar un idioma consistente y
> comprensible dentro del proyecto.

Reglas para VES:

- Mantener el idioma del módulo y del contrato existente.
- Preferir español para nuevos identificadores de dominio cuando el módulo ya
  esté en español.
- No mezclar inglés y español dentro del mismo concepto sin una razón técnica.
- Conservar términos técnicos consolidados (`Controller`, `DTO`, `Async`,
  `Request`, `Response`, `DbContext`, etc.) cuando formen parte del framework o
  del código existente.
- Mensajes de error y logs deben ser descriptivos, consistentes y útiles para
  diagnóstico.
- No traducir nombres públicos existentes si ello rompe contratos,
  serialización, consumidores o consistencia histórica.

**Evitar**

```csharp
var candidate = await _fondoDA.GetFondosAsync(codigoEntidad);

_logger.LogError(
    "Failed actualizando el fondo {CodigoFondo}",
    codigoFondo);
```

**Recomendado**

```csharp
var fondos = await _fondoDA.ObtengaFondosAsync(codigoEntidad);

_logger.LogError(
    "No fue posible actualizar el fondo {CodigoFondo}",
    codigoFondo);
```

Si una integración externa exige nombres o mensajes en inglés, respetar el
contrato externo en esa frontera sin cambiar el vocabulario interno
innecesariamente.

### Convenciones de tipos y firmas

> En C# esta recomendación se traduce a nullability, tipos concretos, 
> DTOs/contratos explícitos y evitar `dynamic` o
> `object` como escape del sistema de tipos.

**Evitar**

```csharp
public object Procese(dynamic datos)
{
    return datos;
}
```

**Recomendado**

```csharp
public async Task<FondoResultado> ProceseAsync(
    FondoSolicitud solicitud,
    CancellationToken cancellationToken)
{
    // ...
}
```

Para entradas complejas, preferir contratos/DTOs existentes en
`Bccr.Supen.Ves.Abstracciones` antes que diccionarios, objetos anónimos o
estructuras sin tipo.

### Convenciones de métodos

Reglas:

- Nombre con verbo o intención de negocio.
- `PascalCase`.
- Sufijo `Async` para métodos asíncronos cuando corresponda.
- Evitar abreviaturas nuevas.
- Evitar nombres genéricos (`Process`, `Execute`, `Do`, `Handle`) si existe un
  nombre de dominio más preciso.
- Parámetros en `camelCase`.
- Retorno explícito y coherente con el contrato.
- Un método no debe mezclar responsabilidades de Controller, BW y DA.

**Evitar**

```csharp
public async Task<object> Process(int x, object data)
```

**Recomendado**

```csharp
public async Task<RentabilidadResultado> RegistreRentabilidadAsync(
    RentabilidadSolicitud solicitud,
    CancellationToken cancellationToken)
```

### Convenciones de constantes


**Documento 1**

```text
MAX_CANDIDATES_PER_PAGE
```

**Equivalente VES**

```csharp
private const int CantidadMaximaResultadosPorPagina = 100;
```

Para objetos o valores calculados en runtime:

```csharp
private static readonly TimeSpan TiempoEsperaPredeterminado =
    TimeSpan.FromSeconds(30);
```

No usar constantes para valores que deben cambiar por ambiente:

```csharp
// Evitar
private const string UrlMapi =
    "https://servidor-produccion/api";
```

Esos valores deben proceder de configuración.


### Regla principal: consistencia con el código VES

Antes de crear o renombrar un identificador:

1. Revisar archivos vecinos de la misma capa.
2. Mantener el idioma y vocabulario de dominio existentes.
3. No renombrar contratos públicos solo para cumplir una convención genérica.
4. Preferir nombres que expresen intención de negocio, no detalles técnicos.
5. Evitar abreviaturas nuevas salvo que ya formen parte del vocabulario del
   proyecto (`SG`, `BW`, `DA`, `DTO`, etc.).

### Clases, interfaces, records, enums y DTOs

| Elemento | Convención VES/.NET | Ejemplo |
|---|---|---|
| Clase | `PascalCase` | `FondoBW` |
| Interface | `I` + `PascalCase` | `IFondoDA` |
| Record / DTO | `PascalCase` | `FondoSolicitud` |
| Enum | `PascalCase` | `EstadoSolicitud` |
| Miembro de enum | `PascalCase` | `Pendiente` |
| Método público | `PascalCase` | `ObtengaLosFondosAsync` |
| Propiedad pública | `PascalCase` | `CodigoEntidad` |
| Namespace | `PascalCase` siguiendo proyecto | `Bccr.Supen.Ves.Abstracciones` |

**Evitar**

```csharp
public class fondo_bw
{
    public int codigo_entidad { get; set; }
}
```

**Recomendado**

```csharp
public sealed class FondoBW : IFondoBW
{
    public int CodigoEntidad { get; init; }
}
```

### Variables locales y parámetros

Usar `camelCase` y nombres descriptivos.

**Evitar**

```csharp
var x = await _fondoDA.ObtengaAsync(id);
var tmp = x.FirstOrDefault();
```

**Recomendado**

```csharp
var fondos = await _fondoDA.ObtengaAsync(codigoEntidad);
var primerFondo = fondos.FirstOrDefault();
```

Para valores booleanos, usar nombres que puedan leerse como una afirmación o
pregunta de negocio. Mantener la forma que ya use el módulo (`es`, `tiene`,
`debe`, `puede`, `seAplica`, etc.).

```csharp
bool seAplicaElFiltro;
bool tieneAutorizacion;
bool debeEnviarCertificado;
```

No usar prefijos húngaros ni codificar el tipo en el nombre:

```csharp
// Evitar
string strNombre;
int intCodigoEntidad;

// Recomendado
string nombre;
int codigoEntidad;
```

### Campos privados e inyección de dependencias

Usar `_camelCase` para campos privados, especialmente dependencias inyectadas.
Las dependencias que no cambian después de construir el objeto deben ser
`readonly`.

```csharp
public sealed class FondoBW : IFondoBW
{
    private readonly IFondoDA _fondoDA;
    private readonly ILogger<FondoBW> _logger;

    public FondoBW(
        IFondoDA fondoDA,
        ILogger<FondoBW> logger)
    {
        _fondoDA = fondoDA;
        _logger = logger;
    }
}
```

**Evitar**

```csharp
public sealed class FondoBW
{
    private FondoDA fondoDA = new FondoDA();
}
```

La segunda forma acopla la lógica de negocio a una implementación concreta y
dificulta pruebas y sustitución de dependencias.

### Constantes y valores inmutables

**Evitar números o cadenas mágicas**

```csharp
if (cantidad > 1000)
{
    // ...
}
```

**Recomendado**

```csharp
private const int CantidadMaximaResultados = 1000;

if (cantidad > CantidadMaximaResultados)
{
    // ...
}
```

Usar `const` únicamente para valores conocidos en compilación:

```csharp
private const int CantidadMaximaIntentos = 3;
```

Usar `static readonly` cuando el valor se determine en runtime o sea un objeto:

```csharp
private static readonly TimeSpan TiempoEsperaPredeterminado =
    TimeSpan.FromSeconds(30);
```

No convertir configuración de ambiente en constantes. URLs, timeouts
configurables, certificados, límites operativos y credenciales deben provenir
de la configuración institucional.

### Métodos y funciones

Los nombres deben expresar una acción o intención clara. Evitar nombres
genéricos como `Process`, `Handle`, `Execute` o `DoWork` cuando exista una
acción de dominio más precisa.

**Evitar**

```csharp
public async Task<List<Fondo>> Process(int id)
```

**Recomendado**

```csharp
public async Task<IReadOnlyCollection<Fondo>> ObtengaLosFondosAsync(
    int codigoEntidad,
    CancellationToken cancellationToken)
```

Para operaciones asíncronas que retornan `Task`/`Task<T>`, conservar el sufijo
`Async` cuando esa sea la convención existente del proyecto.

Un método debe mantener un nivel de abstracción coherente. Si simultáneamente
valida HTTP, aplica reglas de negocio y ejecuta SQL, probablemente está
mezclando responsabilidades que pertenecen a Controller, BW y DA.

### Tipos de retorno y colecciones

Preferir el contrato más específico que represente la intención:

```csharp
Task<Fondo?> ObtengaFondoAsync(...);
Task<IReadOnlyCollection<Fondo>> ObtengaFondosAsync(...);
```

Evitar exponer una colección mutable si el consumidor solo necesita leerla:

```csharp
// Evitar cuando no se necesita mutabilidad
Task<List<Fondo>> ObtengaFondosAsync(...);

// Preferible para contrato de solo lectura
Task<IReadOnlyCollection<Fondo>> ObtengaFondosAsync(...);
```

No cambiar firmas públicas existentes únicamente por esta recomendación; se
aplica principalmente a código nuevo.

### `var` y tipos explícitos

Usar `var` cuando el tipo sea evidente por el lado derecho y mejore la lectura:

```csharp
var configuracion = _proveedorApiEndpoints
    .ObtengaLaConfiguracion("ObtengaLosFondosAsync");
```

Preferir tipo explícito cuando aporte contexto relevante:

```csharp
IReadOnlyCollection<Fondo> fondos =
    await _fondoDA.ObtengaFondosAsync(codigoEntidad);
```

Evitar usar `var` para ocultar tipos importantes del contrato o cuando dificulte
entender el código.

### Nullability y tipos seguros

Mantener nullable reference types y expresar nulos de forma explícita.

**Evitar**

```csharp
public object Obtenga(int id)
{
    // ...
}
```

**Recomendado**

```csharp
public async Task<Fondo?> ObtengaFondoAsync(
    int codigoFondo,
    CancellationToken cancellationToken)
{
    // ...
}
```

No usar `null!`, casts inseguros o `dynamic` para silenciar advertencias sin
resolver la causa.

### Parámetros

Evitar listas largas de parámetros primitivos cuando representen un único
concepto de entrada y el proyecto ya use DTOs para ese límite.

**Evitar**

```csharp
public Task ActualiceAsync(
    int entidad,
    int fondo,
    int regimen,
    DateTime fecha,
    decimal valor,
    string tipo)
```

**Recomendado cuando el contrato lo justifica**

```csharp
public Task ActualiceAsync(
    ActualizacionRentabilidad solicitud,
    CancellationToken cancellationToken)
```

No crear un DTO nuevo solo para envolver uno o dos parámetros cuando no mejora
el contrato.

### Expresiones condicionales y retornos tempranos

Reducir anidamiento con validaciones y retornos tempranos cuando mejore la
claridad.

**Evitar**

```csharp
if (solicitud is not null)
{
    if (solicitud.CodigoEntidad > 0)
    {
        return await _servicio.ProceseAsync(solicitud);
    }
}

throw new ArgumentException("Solicitud inválida.");
```

**Recomendado**

```csharp
ArgumentNullException.ThrowIfNull(solicitud);

if (solicitud.CodigoEntidad <= 0)
{
    throw new ArgumentException(
        "El código de entidad debe ser mayor que cero.",
        nameof(solicitud));
}

return await _servicio.ProceseAsync(solicitud);
```

Usar las validaciones y excepciones institucionales ya existentes si el módulo
dispone de ellas.

### Comentarios y documentación

Los comentarios deben explicar **por qué**, restricciones, decisiones o
comportamientos no evidentes; no repetir el código.

**Evitar**

```csharp
// Incrementa contador
contador++;
```

**Recomendado**

```csharp
// El MAPI legacy requiere numeración basada en 1 para esta secuencia.
contador++;
```

Mantener XML documentation en contratos públicos únicamente donde el proyecto
ya la utilice o donde aporte valor real. No generar documentación repetitiva de
forma automática.

### Formato y legibilidad

- Una responsabilidad principal por tipo.
- Evitar métodos extensos con múltiples niveles de anidación.
- Evitar regiones enormes que oculten una clase con demasiadas
  responsabilidades.
- Mantener `using` y orden/formato según `.editorconfig` o el estilo del
  proyecto.
- No introducir reformateos masivos en un cambio funcional.
- Evitar código comentado; Git conserva el historial.
- Evitar warnings nuevos del compilador o del analizador estático.



### Checklist de nomenclatura

Antes de crear código nuevo comprobar:

- [ ] Variables locales y parámetros en `camelCase`.
- [ ] Clases, DTOs, enums, propiedades y métodos en `PascalCase`.
- [ ] Interfaces con prefijo `I` y `PascalCase`.
- [ ] Campos privados en `_camelCase`.
- [ ] Dependencias inyectadas privadas y `readonly` cuando corresponda.
- [ ] Constantes y `static readonly` con nombres descriptivos; sin números
      mágicos.
- [ ] Archivos `.cs` nombrados según su tipo principal y convención del módulo.
- [ ] Métodos asíncronos con `Async` según convención existente.
- [ ] Booleanos con nombres afirmativos/descriptivos.
- [ ] Sin prefijos húngaros (`strNombre`, `intCodigo`, etc.).
- [ ] Sin abreviaturas nuevas innecesarias.
- [ ] Sin `dynamic`, `object` o `null!` usados para ocultar problemas de tipos.
- [ ] Mensajes y logs en el idioma coherente con VES y con contexto suficiente.
- [ ] No se renombraron contratos públicos únicamente por estilo.


## Límites de responsabilidad y arquitectura

### API de orquestación (`Bccr.Supen.Ves.Api`)

La API valida y recibe solicitudes HTTP, aplica los atributos, filtros y
middleware configurados, y delega las operaciones a `SI.SG`. El SG resuelve el
`ApiEndpoint` mediante `IProveedorApiEndpoints` y usa `IRequestsHandler` para
llamar al MAPI. La API no contiene el acceso EF/SQL del MAPI ni debe duplicar
su lógica de negocio o acceso a datos.

Al agregar un endpoint, seguir el flujo de la guía del proyecto:

1. Reutilizar o agregar el DTO en `Bccr.Supen.Ves.Abstracciones`.
2. Definir el contrato SG y, cuando corresponda en el patrón del proyecto, el
   contrato de controller.
3. Implementar la llamada en `SI.SG` por medio de `IRequestsHandler`.
4. Implementar el controller en `API/Controllers/Version1/` delegando al SG.
5. Agregar la configuración del endpoint y pruebas relevantes.

Ejemplo ilustrativo del patrón de gateway existente:

```csharp
public sealed class FondoSG : IFondoSG
{
    private readonly IProveedorApiEndpoints _proveedorApiEndpoints;
    private readonly IRequestsHandler _requestHandler;

    public FondoSG(
        IProveedorApiEndpoints proveedorApiEndpoints,
        IRequestsHandler requestHandler)
    {
        _proveedorApiEndpoints = proveedorApiEndpoints;
        _requestHandler = requestHandler;
    }

    public async Task<IEnumerable<Fondo>> ObtengaLosFondosDeUnaEntidadAsync(
        int elCodigoEntidad,
        bool seAplicaElFiltro)
    {
        var configuracion = _proveedorApiEndpoints
            .ObtengaLaConfiguracion("ObtengaLosFondosDeUnaEntidadAsync");
        var apiEndpoint = new ApiEndpoint
        {
            Ruta = string.Format(
                configuracion.Ruta,
                elCodigoEntidad,
                seAplicaElFiltro),
            EnviaCertificadoDeAutorizacion = true,
            EnviaContextoBccr = true,
            Monitoreable = true
        };

        var respuesta =
            await _requestHandler.GetAsync<IEnumerable<Fondo>>(apiEndpoint);
        return respuesta.Respuesta;
    }
}
```

El ejemplo es orientativo: conservar firmas, opciones del endpoint y
comportamiento del SG ya existente en el módulo que se modifica. La ruta y su
nombre configurado deben coincidir. No construir URLs de ambiente en código ni
crear clientes HTTP alternativos cuando ya corresponde `IRequestsHandler`.

### Servicio de datos y negocio (`Bccr.Supen.Ves.Mapi`)

La arquitectura interna es **MAPI Controller → BW → DA → SQL Server**.
`Bccr.Supen.Ves.Abstracciones` contiene las interfaces de MAPI, BW y DA y los
modelos/DTOs correspondientes. Los controllers delegan reglas de negocio a BW;
BW usa contratos DA; DA encapsula los accesos a datos. Mantener la dirección de
dependencias existente y usar inyección por constructor.

- Controller: HTTP, atributos, binding y delegación; evitar lógica de datos.
- BW: reglas de negocio y coordinación de operaciones.
- DA: acceso a las bases configuradas, EF Core y stored procedures existentes.
- Reutilizar las abstracciones y convenciones actuales; no agregar capas
  `domain/application/infrastructure` propias de plantillas ajenas.
- No crear una entidad rica, agregado DDD, fábrica o repositorio nuevo salvo
  que resuelva una necesidad del dominio y encaje con el diseño que existe.

### Responsabilidad única, SOLID y DRY

- Dar a cada tipo una responsabilidad concreta, especialmente a controllers,
  clases BW y DA.
- Preferir composición e interfaces de proyecto sobre dependencias directas
  hacia implementaciones concretas. Inyectar dependencias por constructor.
- Mantener interfaces pequeñas y enfocadas cuando se creen contratos nuevos;
  no dividir interfaces existentes sin una razón funcional.
- Extraer lógica repetida cuando representa la misma regla de negocio y puede
  tener una única fuente de verdad. No generalizar código que solo parece
  similar ni agregar abstracciones antes de que hagan falta.
- Usar herencia solo cuando la sustitución respete realmente el contrato de la
  clase base; preferir composición para compartir comportamiento.
- No mover una regla de negocio entre API y MAPI: verificar qué servicio es su
  dueño y preservar el contrato entre ambos.

> Aplicar Open/Closed con criterio: preferir extensiones mediante
contratos, composición, estrategias o implementaciones ya previstas por el
proyecto antes que modificar múltiples consumidores para una variación de
comportamiento. No introducir fábricas, eventos de dominio o jerarquías de
herencia únicamente para cumplir una plantilla arquitectónica.

> Evitar abstracciones genéricas para operaciones DA solo por eliminar
líneas repetidas. Extraer componentes comunes únicamente cuando representen la
misma regla, el mismo contrato y el mismo ciclo de cambio.


### Ejemplos de SOLID y DRY aplicados a VES

#### SRP — Single Responsibility Principle

**Evitar: Controller con validación de negocio y acceso a datos**

```csharp
[HttpGet]
public async Task<IActionResult> ObtengaFondos(int codigoEntidad)
{
    if (codigoEntidad <= 0)
    {
        return BadRequest();
    }

    await using var contexto = new VesDbContext();
    var fondos = await contexto.Fondos
        .Where(x => x.CodigoEntidad == codigoEntidad)
        .ToListAsync();

    return Ok(fondos);
}
```

**Recomendado: cada capa conserva su responsabilidad**

```csharp
[HttpGet]
public async Task<IActionResult> ObtengaFondos(
    int codigoEntidad,
    CancellationToken cancellationToken)
{
    var fondos = await _fondoBW.ObtengaFondosAsync(
        codigoEntidad,
        cancellationToken);

    return Ok(fondos);
}
```

```csharp
public sealed class FondoBW : IFondoBW
{
    private readonly IFondoDA _fondoDA;

    public FondoBW(IFondoDA fondoDA)
    {
        _fondoDA = fondoDA;
    }

    public Task<IReadOnlyCollection<Fondo>> ObtengaFondosAsync(
        int codigoEntidad,
        CancellationToken cancellationToken)
    {
        if (codigoEntidad <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(codigoEntidad));
        }

        return _fondoDA.ObtengaFondosAsync(
            codigoEntidad,
            cancellationToken);
    }
}
```

Controller atiende HTTP; BW aplica reglas; DA encapsula SQL/EF.

#### OCP — Open/Closed Principle

No interpretar OCP como obligación de crear herencia. En VES normalmente es
preferible extender por contratos y composición.

**Evitar**

```csharp
public sealed class Notificador
{
    public Task EnvieAsync(string tipo, string mensaje)
    {
        if (tipo == "correo")
        {
            // ...
        }
        else if (tipo == "sms")
        {
            // ...
        }

        return Task.CompletedTask;
    }
}
```

**Recomendado cuando realmente existen variantes de comportamiento**

```csharp
public interface INotificador
{
    Task EnvieAsync(
        string mensaje,
        CancellationToken cancellationToken);
}
```

```csharp
public sealed class NotificadorCorreo : INotificador
{
    public Task EnvieAsync(
        string mensaje,
        CancellationToken cancellationToken)
    {
        // Implementación correspondiente.
        return Task.CompletedTask;
    }
}
```

Agregar esta abstracción únicamente si el dominio realmente necesita variantes;
no crear estrategias especulativas.

#### LSP — Liskov Substitution Principle

Una implementación de una interfaz debe respetar su contrato y no sorprender
al consumidor.

**Evitar**

```csharp
public sealed class FondoDA : IFondoDA
{
    public Task<IReadOnlyCollection<Fondo>> ObtengaFondosAsync(
        int codigoEntidad,
        CancellationToken cancellationToken)
    {
        throw new NotSupportedException();
    }
}
```

Si una implementación no puede cumplir el contrato, la abstracción probablemente
no representa correctamente esa implementación.

**Recomendación:** preferir composición y contratos pequeños; usar herencia
solamente cuando la sustitución sea válida.

#### ISP — Interface Segregation Principle

**Evitar una interfaz que obligue a implementar operaciones no relacionadas**

```csharp
public interface IFondoOperaciones
{
    Task<IReadOnlyCollection<Fondo>> ObtengaAsync();
    Task ActualiceAsync(Fondo fondo);
    Task EnvieCorreoAsync(Fondo fondo);
    Task GenereReporteAsync(Fondo fondo);
}
```

**Recomendado**

```csharp
public interface IFondoDA
{
    Task<IReadOnlyCollection<Fondo>> ObtengaAsync(
        CancellationToken cancellationToken);

    Task ActualiceAsync(
        Fondo fondo,
        CancellationToken cancellationToken);
}
```

Los servicios de correo o reportería pertenecen a contratos separados si
realmente existen como responsabilidades distintas.

#### DIP — Dependency Inversion Principle

**Evitar**

```csharp
public sealed class FondoBW
{
    private readonly FondoDA _fondoDA = new();
}
```

**Recomendado**

```csharp
public sealed class FondoBW : IFondoBW
{
    private readonly IFondoDA _fondoDA;

    public FondoBW(IFondoDA fondoDA)
    {
        _fondoDA = fondoDA;
    }
}
```

El BW depende del contrato `IFondoDA`, no de la implementación concreta.

#### DRY — Don't Repeat Yourself

**Evitar duplicar una misma regla de negocio**

```csharp
public void Cree(Fondo fondo)
{
    if (fondo.CodigoEntidad <= 0)
    {
        throw new ArgumentException("Entidad inválida.");
    }

    // ...
}

public void Actualice(Fondo fondo)
{
    if (fondo.CodigoEntidad <= 0)
    {
        throw new ArgumentException("Entidad inválida.");
    }

    // ...
}
```

**Recomendado cuando se trata de la misma regla**

```csharp
private static void ValideEntidad(Fondo fondo)
{
    if (fondo.CodigoEntidad <= 0)
    {
        throw new ArgumentException("Entidad inválida.");
    }
}
```

DRY no significa crear helpers genéricos para cualquier similitud textual. Dos
bloques parecidos pueden representar reglas distintas y tener ciclos de cambio
diferentes.


## Acceso a datos MAPI

- Reutilizar stored procedures y esquemas existentes; consultar
  `Documentacion/SCHEMA.md` antes de elegir la base, esquema o procedimiento.
- Usar SQL parametrizado. Para nuevos procedimientos/resultados EF, el patrón
  recomendado del MAPI es `FromSqlInterpolated` con parámetros interpolados;
  nunca concatenar entradas para formar SQL:

```csharp
var resultados = await _contexto.Set<Modelos.MiModelo>()
    .FromSqlInterpolated(
        $"EXEC [esquema].[SP_Nombre] @Parametro = {valor}")
    .ToListAsync();
```

- Los modelos de `DA/Modelos/` pueden mapear resultados de stored procedures,
  no necesariamente tablas. Mapear los nombres de columnas reales y seguir
  cómo el repositorio y el `DbContext` registran los modelos no clave.
- No hacer consultas LINQ sobre modelos de resultados de SP si el modelo está
  definido exclusivamente para su ejecución con `FromSqlInterpolated`.
- Los helpers institucionales de repositorio que ya existan pueden aparecer en
  código legado. Para trabajo nuevo, preferir la parametrización segura
  anterior. Si el cambio necesariamente sigue un patrón legado, revisar su
  seguridad, documentar la limitación si aplica y evitar perpetuar concatenación
  de SQL.
- No agregar migraciones EF, actualizar esquema, crear/modificar tablas o
  procedimientos desde el código. Coordinar dichos cambios con DBA y mantener
  el esquema fuera del ciclo de despliegue de la aplicación.
- No ejecutar cambios de datos en pruebas unitarias. Usar los dobles,
  contexto InMemory u otras estrategias que ya estén configuradas; recordar
  que EF InMemory no valida el comportamiento exacto de SQL Server ni de un SP.
- Proyectar/leer solo los datos necesarios cuando el procedimiento y su
  contrato lo permitan. No cambiar un SP compartido o su firma para una
  optimización unilateral.

> Evitar patrones equivalentes a N+1. Si una operación requiere
información relacionada, evaluar si puede obtenerse en una sola consulta,
stored procedure o lectura coordinada, siempre respetando los contratos y
objetos administrados por DBA.

> Antes de proponer índices, cambios de plan de ejecución o
modificaciones de stored procedures, medir el problema y coordinarlo con DBA.
El backend puede identificar la necesidad, pero no debe aplicar cambios
unilaterales sobre el esquema.

> Cuando una operación sea transaccional, probar tanto la confirmación
como el rollback ante fallos. Mantener la frontera transaccional en la capa que
ya es responsable de coordinar la operación; no distribuir una transacción
entre API y MAPI.


### Ejemplos de acceso a datos seguro y eficiente

> Se conservan las
> recomendaciones de seleccionar solo los datos necesarios, evitar N+1,
> parametrizar consultas y probar transacciones, adaptadas a EF Core 8,
> SQL Server y stored procedures administrados por DBA.

#### SQL parametrizado

**Evitar**

```csharp
var sql =
    $"EXEC [INADM].[ObtengaFondo] @Entidad = '{codigoEntidad}'";

var resultado = await _contexto.Set<FondoResultado>()
    .FromSqlRaw(sql)
    .ToListAsync();
```

**Recomendado**

```csharp
var resultado = await _contexto.Set<FondoResultado>()
    .FromSqlInterpolated(
        $"EXEC [INADM].[ObtengaFondo] @Entidad = {codigoEntidad}")
    .ToListAsync(cancellationToken);
```

La interpolación de `FromSqlInterpolated` se convierte en parámetros; no
confundirla con construir manualmente una cadena SQL.

#### Proyección de datos

Cuando se consulte una entidad EF real y el contrato lo permita, seleccionar
solo los campos necesarios.

```csharp
var fondos = await _contexto.Fondos
    .AsNoTracking()
    .Where(fondo => fondo.CodigoEntidad == codigoEntidad)
    .Select(fondo => new FondoResumen
    {
        Codigo = fondo.Codigo,
        Nombre = fondo.Nombre
    })
    .ToListAsync(cancellationToken);
```

No aplicar este patrón mecánicamente a modelos que existen únicamente para
mapear resultados de stored procedures.

#### Evitar N+1

**Evitar**

```csharp
var entidades = await _contexto.Entidades
    .AsNoTracking()
    .ToListAsync(cancellationToken);

foreach (var entidad in entidades)
{
    entidad.Fondos = await _contexto.Fondos
        .Where(fondo => fondo.CodigoEntidad == entidad.Codigo)
        .ToListAsync(cancellationToken);
}
```

**Preferir**, si el modelo EF y el contrato lo soportan, una consulta coordinada
mediante proyección, navegación, `Include` o un stored procedure apropiado. La
solución exacta debe seguir el modelo real de MAPI y puede requerir coordinación
con DBA.

#### `AsNoTracking`

Para lecturas puras de entidades EF, considerar `AsNoTracking()` cuando el
objeto no vaya a actualizarse dentro de la misma unidad de trabajo.

```csharp
var fondo = await _contexto.Fondos
    .AsNoTracking()
    .SingleOrDefaultAsync(
        x => x.Codigo == codigoFondo,
        cancellationToken);
```

No usarlo si el flujo depende del tracking para una actualización posterior.

#### Transacciones

Usar una transacción explícita solo cuando la operación coordine cambios que
deben confirmarse o revertirse como una unidad y el patrón de acceso del proyecto
lo requiera.

```csharp
await using var transaccion =
    await _contexto.Database.BeginTransactionAsync(cancellationToken);

try
{
    // Cambios relacionados.
    await _contexto.SaveChangesAsync(cancellationToken);

    await transaccion.CommitAsync(cancellationToken);
}
catch
{
    await transaccion.RollbackAsync(cancellationToken);
    throw;
}
```

No abrir una transacción manual alrededor de cada `SaveChangesAsync`; EF Core ya
maneja transacción para una única operación de guardado cuando el proveedor lo
soporta.


## API, contratos y validación

- La API expone rutas de dominio versionadas (`api-version=1.0`) y consume
  endpoints MAPI configurados. MAPI también expone contratos versionados según
  sus controllers y configuración existentes. No imponer rutas REST nuevas
  sobre acciones legacy ni cambiar casing, nombres de parámetros o cuerpos de
  respuesta sin revisar consumidores.
- Usar verbos HTTP y códigos de estado coherentes con el comportamiento actual.
  Conservar serialización, nombres de campos y forma de respuesta establecidos
  por el API y los paquetes BCCR.
- Mantener la validación en los límites de entrada (binding, validadores,
  atributos o validaciones del proyecto) y volver a comprobar en BW cualquier
  invariante de negocio. La validación de UI no sustituye la validación del
  servidor.
- No aceptar valores de identidad, permisos, rutas de archivo, nombres de SQL
  ni límites de recursos sin validar. Para valores de ruta/parámetros usar las
  APIs y el binding existentes, no interpolación manual de SQL ni composición
  insegura de URLs.
- No agregar validadores o formatos de error paralelos cuando el middleware o
  paquete institucional ya gestiona la respuesta. Propagar o convertir errores
  de acuerdo con el manejo de excepciones configurado.
- Al integrar cambios entre API y MAPI, comprobar la configuración del
  endpoint, método HTTP, versión, parámetros, DTO/JSON, autenticación y
  certificado requeridos en ambos extremos.

> Mantener respuestas HTTP y errores consistentes con los contratos
existentes. No introducir envoltorios JSON, códigos de error o formatos de
respuesta nuevos en un endpoint aislado si el middleware o los paquetes BCCR
ya establecen el formato común.

> Validar y normalizar entradas antes de ejecutar lógica de negocio.
La normalización no debe cambiar silenciosamente el significado de los datos;
cuando un valor no sea válido, preferir una respuesta de validación explícita.


### Ejemplos de validación y contratos

> Validar antes de
> procesar sigue siendo una regla central, pero en VES la validación se ubica
> donde corresponda al diseño existente: binding/atributos o validadores en el
> límite HTTP y reglas/invariantes en BW.

**Validación de entrada HTTP**

```csharp
[HttpGet]
public async Task<IActionResult> ObtengaFondos(
    [FromQuery] int codigoEntidad,
    CancellationToken cancellationToken)
{
    if (codigoEntidad <= 0)
    {
        return BadRequest("El código de entidad debe ser mayor que cero.");
    }

    var resultado = await _fondoSG.ObtengaFondosAsync(
        codigoEntidad,
        cancellationToken);

    return Ok(resultado);
}
```

Si el proyecto dispone de filtros, validadores o middleware institucional para
esa validación, reutilizarlos en lugar de duplicarla manualmente.

**Regla de negocio en BW**

```csharp
public async Task<IReadOnlyCollection<Fondo>> ObtengaFondosAsync(
    int codigoEntidad,
    CancellationToken cancellationToken)
{
    if (!await _entidadDA.ExisteAsync(codigoEntidad, cancellationToken))
    {
        throw new EntidadNoExisteException(codigoEntidad);
    }

    return await _fondoDA.ObtengaFondosAsync(
        codigoEntidad,
        cancellationToken);
}
```

No confiar únicamente en que la UI ya validó el valor.


## Errores, observabilidad y asincronía

- Informar errores explícitamente; no retornar respuestas de éxito vacías para
  ocultar fallos ni capturar excepciones de forma amplia sin registrarlas,
  contextualizarlas o relanzarlas correctamente.
- No exponer stack traces, SQL, cadenas de conexión, tokens, certificados,
  claims sensibles ni secretos en respuestas HTTP o logs.
- Usar manejo de excepciones, correlation ID y logging ya configurados por
  los paquetes institucionales. Registrar contexto operacional útil (módulo,
  operación/correlation ID y resultado) sin datos sensibles.
- Elegir niveles y mensajes de log que permitan diagnosticar sin registrar el
  mismo error repetidamente en cada capa. Evitar catch/rethrow que pierda el
  stack trace (`throw ex`) o que duplique logs.
- En código asíncrono, propagar `async`/`await` y cancellation tokens si la
  firma existente los admite. No bloquear con `.Result` o `.Wait()` ni envolver
  trabajo I/O-bound en `Task.Run`.
- Ejecutar tareas independientes en paralelo solo si no hay dependencia de
  orden y los servicios/DbContext usados soportan esa concurrencia. No hacer
  operaciones paralelas concurrentes con la misma instancia de `DbContext`.
- Para endpoints de carga/descarga de archivos, conservar los límites
  coordinados entre ASP.NET Core, IIS, MAPI y su configuración. No subir un
  límite solo en una capa.

> Preferir retornos tempranos cuando reduzcan anidación y hagan más
claro el camino de error o validación. No usar excepciones como mecanismo de
flujo normal cuando el contrato pueda expresar el caso esperado de forma
explícita.

> Mantener logging estructurado cuando la infraestructura existente lo
admita, incluyendo identificadores de operación, correlation ID y datos de
contexto no sensibles. Usar niveles de log coherentes (`Debug`, `Information`,
`Warning`, `Error`) y evitar registrar el mismo fallo en cada capa.

> No envolver una excepción únicamente para cambiar el texto si no se
aporta contexto útil, clasificación o semántica adicional. Cuando se relance
la misma excepción conservar el stack original.


### Ejemplos de manejo de errores

> Se conserva la
> recomendación de errores descriptivos, propagación natural y manejo
> centralizado; se adapta al middleware y excepciones existentes en ASP.NET Core
> y paquetes BCCR.

**Evitar perder el stack trace**

```csharp
try
{
    await _servicio.ProceseAsync();
}
catch (Exception ex)
{
    throw ex; // Evitar
}
```

**Recomendado cuando solo se necesita relanzar**

```csharp
try
{
    await _servicio.ProceseAsync();
}
catch
{
    throw;
}
```

Mejor aún: no agregar `try/catch` si no se va a agregar contexto, compensación,
clasificación o una acción concreta.

**Agregar contexto sin exponer secretos**

```csharp
try
{
    await _fondoDA.ActualiceAsync(fondo, cancellationToken);
}
catch (SqlException ex)
{
    _logger.LogError(
        ex,
        "Error actualizando fondo {CodigoFondo} para entidad {CodigoEntidad}",
        fondo.Codigo,
        fondo.CodigoEntidad);

    throw;
}
```

No registrar cadenas de conexión, tokens, certificados, datos sensibles ni SQL
completo con valores confidenciales.

### Logging estructurado

**Evitar**

```csharp
_logger.LogInformation(
    "Actualizando fondo " + fondo.Codigo + " de " + fondo.CodigoEntidad);
```

**Recomendado**

```csharp
_logger.LogInformation(
    "Actualizando fondo {CodigoFondo} de entidad {CodigoEntidad}",
    fondo.Codigo,
    fondo.CodigoEntidad);
```

El logging estructurado conserva campos consultables y evita construir cadenas
cuando el nivel no está habilitado.

### Asincronía y cancelación

> El principio
> `async/await` se conserva. En .NET se añade propagación de
> `CancellationToken` cuando la firma y dependencias lo soporten.

**Evitar bloqueo síncrono**

```csharp
var fondos = _fondoDA.ObtengaFondosAsync(codigoEntidad).Result;
```

```csharp
_fondoDA.ObtengaFondosAsync(codigoEntidad).Wait();
```

**Recomendado**

```csharp
var fondos = await _fondoDA.ObtengaFondosAsync(
    codigoEntidad,
    cancellationToken);
```

**Evitar `Task.Run` para I/O**

```csharp
var fondos = await Task.Run(
    () => _fondoDA.ObtengaFondosAsync(codigoEntidad));
```

El acceso EF/HTTP ya es I/O asíncrono; envolverlo en `Task.Run` agrega costo sin
hacerlo más escalable.

### Paralelismo seguro

Operaciones independientes pueden ejecutarse en paralelo **solo** cuando sus
dependencias lo permitan.

```csharp
var tareaCatalogos = _catalogoSG.ObtengaAsync(cancellationToken);
var tareaParametros = _parametroSG.ObtengaAsync(cancellationToken);

await Task.WhenAll(tareaCatalogos, tareaParametros);

var catalogos = await tareaCatalogos;
var parametros = await tareaParametros;
```

No ejecutar consultas EF concurrentes sobre la misma instancia de `DbContext`.


## Seguridad y configuración

- Validar entradas y autorizar cada operación del lado servidor. No asumir que
  el frontend validó o autorizó al usuario.
- Mantener `[Authorize]`, perfiles y excepciones anónimas de acuerdo con el
  requerimiento y el patrón del controller. Un endpoint de health check
  anónimo debe ser una excepción deliberada, de alcance mínimo y documentada.
- La API usa autenticación JWT según ambiente; el handler existente puede
  decodificar claims y confiar en la validación realizada antes por la UI o
  infraestructura. Decodificar un JWT **no valida** firma, emisor ni audiencia.
  No alterar esta frontera de confianza sin revisar la arquitectura de
  autenticación y los consumidores.
- La API se comunica con MAPI con mTLS en los ambientes que lo requieren,
  incluidos Staging y Producción. Mantener el envío y configuración del
  certificado cliente en esos ambientes; los decoradores que omiten
  certificados están limitados a Development. No extender un bypass a otros
  ambientes.
- Leer configuración de `IConfiguration`/options y de los mecanismos por
  ambiente existentes. No fijar URLs, credenciales, rutas de servidor,
  certificados o claves en código.
- `appsettings` puede contener tokens que el pipeline sustituye; respetar el
  esquema de configuración y usar User Secrets/variables seguras para
  credenciales locales. Nunca commitear secretos reales ni incluirlos en
  ejemplos.
- No ampliar CORS, `AllowedHosts`, acceso anónimo ni permisos del proceso por
  comodidad. Ajustar según el consumidor y el ambiente, con revisión de
  seguridad.
- Parametrizar SQL y validar identificadores/direcciones que no puedan
  parametrizarse. No desactivar controles de SonarQube para evitar una
  advertencia sin justificar y revisar el riesgo.

> Validar al arranque la configuración obligatoria cuando el patrón del
proyecto lo permita, de modo que una ausencia crítica falle de forma temprana
y explícita en lugar de aparecer durante la atención de una solicitud.

> Evitar estado global mutable para conexiones, clientes o datos de
solicitud. Usar el contenedor de inyección de dependencias y los ciclos de vida
(`Singleton`, `Scoped`, `Transient`) ya definidos por ASP.NET Core y por el
proyecto, verificando especialmente el ciclo de vida de `DbContext`.

> Sanitizar o rechazar entradas que puedan terminar en SQL, rutas,
cabeceras, archivos, logs o URLs. Parametrizar siempre que la plataforma lo
permita y aplicar allow-lists cuando se trate de identificadores que no puedan
parametrizarse.


### Ejemplos de seguridad y configuración

> Las recomendaciones de
> no commitear secretos, usar configuración externa, validar entradas y evitar
> estado global se conservan, usando `IConfiguration`, Options, DI y mecanismos
> institucionales.

#### Configuración tipada

Cuando el proyecto ya use Options, preferir configuración tipada a cadenas
dispersas:

```csharp
public sealed class MapiOptions
{
    public const string SectionName = "Mapi";

    public required string BaseUrl { get; init; }
    public int TimeoutSegundos { get; init; }
}
```

```csharp
builder.Services
    .AddOptions<MapiOptions>()
    .Bind(builder.Configuration.GetSection(MapiOptions.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

No introducir Options si el proyecto usa otro mecanismo institucional de
configuración; el principio es centralizar y validar la configuración, no
cambiar la infraestructura.

#### Secretos

**Nunca**

```json
{
  "ConnectionStrings": {
    "Ves": "Server=servidor;User Id=usuario;Password=clave-real;"
  }
}
```

en un archivo versionado con credenciales reales.

Usar los mecanismos de sustitución del pipeline, User Secrets o variables
seguras ya definidos por el proyecto.

#### Ciclos de vida DI

No registrar un `DbContext` o un componente dependiente del contexto como
`Singleton`.

```csharp
// Evitar
services.AddSingleton<VesDbContext>();
```

Seguir el registro EF Core y los ciclos de vida existentes. Antes de cambiar
`Singleton`, `Scoped` o `Transient`, revisar las dependencias transitivas y el
uso concurrente.


## Pruebas

### Frameworks existentes por proyecto

| Proyecto de pruebas | Uso/framework actual |
|---|---|
| API `Bccr.Supen.Ves.Api.UnitTests` | MSTest, NSubstitute y Moq |
| API `SI.SG.Pruebas` | MSTest; probar gateway sustituyendo `IRequestsHandler` |
| API `API.IntegrationTests` | xUnit y `CustomApplicationFactory` |
| MAPI `DA.Pruebas`, `BW.Pruebas`, `MAPI.UnitTests` | MSTest, con Moq/NSubstitute según el patrón de la capa |

No introducir xUnit/NUnit a las pruebas unitarias MSTest ni convertir suites
existentes. La excepción actual son las pruebas de integración API, que usan
xUnit. Ver los `.csproj` de cada proyecto y mantener su estilo.

### Cobertura y casos

- Probar el comportamiento observable en la capa apropiada: controller,
  gateway, regla BW o mapping DA.
- Cubrir caminos exitosos y relevantes de validación, error, límites,
  ausencia de datos y parámetros, según la función. No es necesario duplicar
  cada caso en todas las capas.
- En API, sustituir el SG al probar un controller y `IRequestsHandler` al
  probar el SG. En MAPI, sustituir la dependencia inferior para probar BW o
  controller. Probar la capa DA con el mecanismo aislado ya configurado.
- Usar `CustomApplicationFactory` para integración HTTP de API donde aporte
  valor. Las pruebas unitarias no deben requerir red BCCR, credenciales,
  certificado de producción ni base de datos real.
- No fijar un porcentaje universal de cobertura en esta guía: respetar el
  Quality Gate, configuración de cobertura y umbrales efectivos del pipeline
  correspondiente; aumentar cobertura con pruebas significativas.
- No mockear todo indiscriminadamente ni probar detalles privados. Aserciones
  claras; AAA es una guía útil, no una obligación de introducir estructura
  repetitiva.
- Mantener los tests al cambiar contratos y probar la serialización/config
  integrada cuando una modificación pueda romper el flujo API→MAPI.
- No marcar pruebas exitosas si la operación bajo prueba falló; evitar
  `catch` que absorba una excepción sin una aserción que demuestre el
  comportamiento esperado.

> Organizar pruebas con un patrón claro de preparación, ejecución y
verificación (AAA) cuando mejore la lectura. Los nombres deben describir el
comportamiento y la condición probada, siguiendo la convención existente del
proyecto en lugar de imponer nombres de Jest o TypeScript.

> Usar datos de prueba realistas y reutilizables. Cuando exista
repetición significativa, preferir builders, factories, helpers o métodos de
creación de datos de prueba compatibles con MSTest/xUnit antes que duplicar
valores arbitrarios en muchos tests.

> Cubrir, según aplique, camino exitoso, errores esperados e inesperados,
límites, ausencia de datos, validaciones, transformación/mapeo y puntos de
integración. Incluir escenarios de timeout, cancelación y fallo de dependencias
cuando formen parte del comportamiento observable.

> En pruebas de controller sustituir la dependencia de negocio
correspondiente; en pruebas de BW o SG sustituir su dependencia inferior; en
pruebas DA aislar el acceso según la infraestructura disponible. Verificar
parámetros enviados, valor retornado y efectos laterales relevantes, no detalles
privados de implementación.

> Mantener las pruebas rápidas y deterministas. Evitar I/O real en
pruebas unitarias, configuraciones excesivas, dependencias entre tests y mocks
tan detallados que repliquen internamente la implementación bajo prueba.

> Cuando haya lógica transaccional, verificar escenarios de éxito,
error y rollback. Cuando haya asincronía o concurrencia, comprobar rechazo de
tareas, cancelación y timeouts cuando sean riesgos reales del flujo.


### Patrón de escritura de pruebas

> El documento anterior
> define pruebas con nombres orientados a comportamiento, patrón AAA, mocks por
> capa, datos realistas y cobertura de happy path/error/edge cases. Se conservan
> esas prácticas utilizando MSTest/xUnit y Moq/NSubstitute según cada proyecto.

#### Convención de nombres

Usar nombres que indiquen **método + escenario + resultado esperado**, siguiendo
el idioma y estilo del proyecto.

Ejemplos:

```text
ObtengaFondosAsync_EntidadValida_RetornaFondos
ObtengaFondosAsync_EntidadInexistente_LanzaExcepcion
ActualiceAsync_DADevuelveError_PropagaExcepcion
```

No imponer `snake_case` de Jest ni renombrar suites existentes.

#### Patrón AAA

```csharp
[TestMethod]
public async Task ObtengaFondosAsync_EntidadValida_RetornaFondos()
{
    // Arrange
    var fondoEsperado = new Fondo
    {
        Codigo = 1,
        Nombre = "ROP"
    };

    var fondoDA = Substitute.For<IFondoDA>();
    fondoDA
        .ObtengaFondosAsync(
            10,
            Arg.Any<CancellationToken>())
        .Returns(new[] { fondoEsperado });

    var sut = new FondoBW(fondoDA);

    // Act
    var resultado = await sut.ObtengaFondosAsync(
        10,
        CancellationToken.None);

    // Assert
    Assert.AreEqual(1, resultado.Count);

    await fondoDA
        .Received(1)
        .ObtengaFondosAsync(
            10,
            Arg.Any<CancellationToken>());
}
```

AAA mejora la lectura, pero no requiere comentarios `Arrange/Act/Assert` si el
test ya es trivial y el estilo existente no los usa.

### Mocking por capa

| Unidad bajo prueba | Sustituir normalmente | No debería requerir |
|---|---|---|
| API Controller | SG | MAPI/SQL real |
| SG | `IRequestsHandler`, proveedor de endpoints | Red BCCR |
| MAPI Controller | BW | DA/SQL real |
| BW | DA y otras dependencias de negocio | SQL real |
| DA | Contexto/infraestructura aislada según suite | Producción |

No mockear el objeto bajo prueba ni crear mocks de tipos simples sin necesidad.

### Ejemplo de prueba de Controller con Moq

```csharp
[TestMethod]
public async Task ObtengaFondos_EntidadValida_RetornaOk()
{
    // Arrange
    var fondoSG = new Mock<IFondoSG>();
    fondoSG
        .Setup(x => x.ObtengaFondosAsync(
            10,
            It.IsAny<CancellationToken>()))
        .ReturnsAsync(new[]
        {
            new Fondo { Codigo = 1, Nombre = "ROP" }
        });

    var controller = new FondoController(fondoSG.Object);

    // Act
    var resultado = await controller.ObtengaFondos(
        10,
        CancellationToken.None);

    // Assert
    Assert.IsInstanceOfType(resultado, typeof(OkObjectResult));

    fondoSG.Verify(
        x => x.ObtengaFondosAsync(
            10,
            It.IsAny<CancellationToken>()),
        Times.Once);
}
```

Ajustar constructor, nombres y tipos al controller real. No agregar Moq a una
suite que ya esté estandarizada únicamente con NSubstitute sin una razón.

### Ejemplo de prueba de SG

El equivalente del documento anterior a “mockear repositorio en service tests”
es sustituir `IRequestsHandler` cuando se prueba un SG.

```csharp
[TestMethod]
public async Task ObtengaFondosAsync_ConfiguracionValida_InvocaRequestHandler()
{
    // Arrange
    var proveedor = Substitute.For<IProveedorApiEndpoints>();
    proveedor
        .ObtengaLaConfiguracion("ObtengaFondosAsync")
        .Returns(new ConfiguracionApiEndpoint
        {
            Ruta = "api/fondos/{0}"
        });

    var requestHandler = Substitute.For<IRequestsHandler>();
    requestHandler
        .GetAsync<IEnumerable<Fondo>>(Arg.Any<ApiEndpoint>())
        .Returns(new RespuestaApi<IEnumerable<Fondo>>
        {
            Respuesta = new[] { new Fondo() }
        });

    var sut = new FondoSG(proveedor, requestHandler);

    // Act
    var resultado = await sut.ObtengaFondosAsync(10);

    // Assert
    Assert.IsNotNull(resultado);

    await requestHandler
        .Received(1)
        .GetAsync<IEnumerable<Fondo>>(
            Arg.Is<ApiEndpoint>(endpoint =>
                endpoint.Ruta.Contains("10")));
}
```

Los tipos exactos de respuesta/configuración deben ser los que realmente tenga
el proyecto; el ejemplo muestra el patrón, no autoriza inventar contratos.

### Happy path, errores y edge cases

Para cada comportamiento relevante, considerar:

1. Entrada válida y resultado esperado.
2. Entrada inválida.
3. Dependencia devuelve colección vacía o `null` si el contrato lo permite.
4. Dependencia lanza excepción esperada.
5. Error inesperado y propagación.
6. Valores límite.
7. Cancelación/timeout si forman parte del contrato.
8. Mapping/serialización cuando exista riesgo de incompatibilidad API→MAPI.

No todos los métodos requieren los ocho escenarios. Seleccionar los que tengan
valor según el riesgo y el comportamiento observable.

### Pruebas de excepciones

**MSTest**

```csharp
[TestMethod]
public async Task ObtengaFondosAsync_EntidadInvalida_LanzaExcepcion()
{
    var fondoDA = Substitute.For<IFondoDA>();
    var sut = new FondoBW(fondoDA);

    await Assert.ThrowsExceptionAsync<ArgumentOutOfRangeException>(
        () => sut.ObtengaFondosAsync(
            0,
            CancellationToken.None));
}
```

No usar:

```csharp
try
{
    await sut.ObtengaFondosAsync(0, CancellationToken.None);
}
catch
{
    Assert.IsTrue(true);
}
```

Ese patrón puede marcar como exitosa una prueba que no demuestra qué excepción
ocurrió ni si el método falló por la causa esperada.

### Datos de prueba reutilizables

Cuando muchos tests necesitan objetos complejos, usar builder/factory de test
si la suite ya lo admite.

```csharp
internal static class FondoTestData
{
    public static Fondo CreeFondoValido(
        int codigo = 1,
        int codigoEntidad = 10)
    {
        return new Fondo
        {
            Codigo = codigo,
            CodigoEntidad = codigoEntidad,
            Nombre = "Fondo de prueba"
        };
    }
}
```

Esto evita valores mágicos repetidos y hace explícita la intención del dato.

### Pruebas transaccionales

Cuando DA contenga una operación transaccional relevante, cubrir:

- commit cuando todas las operaciones terminan correctamente;
- rollback cuando una operación falla;
- propagación de la excepción;
- ausencia de efectos parciales.

No usar una base de producción ni infraestructura BCCR real para una prueba
unitaria.

### Pruebas de asincronía, cancelación y timeout

Cuando el método soporte cancelación:

```csharp
[TestMethod]
public async Task ObtengaFondosAsync_SolicitudCancelada_PropagaCancelacion()
{
    using var cts = new CancellationTokenSource();
    cts.Cancel();

    var fondoDA = Substitute.For<IFondoDA>();
    fondoDA
        .ObtengaFondosAsync(
            Arg.Any<int>(),
            Arg.Any<CancellationToken>())
        .Returns<Task<IReadOnlyCollection<Fondo>>>(
            _ => throw new OperationCanceledException());

    var sut = new FondoBW(fondoDA);

    await Assert.ThrowsExceptionAsync<OperationCanceledException>(
        () => sut.ObtengaFondosAsync(10, cts.Token));
}
```

No agregar tests de concurrencia artificiales si la funcionalidad no tiene
comportamiento concurrente.

### Anti-patrones de pruebas

- Probar métodos privados directamente.
- Duplicar la misma prueba en Controller, BW y DA sin que cambie el
  comportamiento observado.
- Usar sleeps (`Thread.Sleep`) para sincronizar pruebas.
- Depender del orden de ejecución de tests.
- Compartir estado mutable entre tests.
- Usar datos de producción.
- Capturar excepciones y marcar el test como exitoso sin verificar el tipo o
  comportamiento.
- Configurar mocks tan detallados que el test replique la implementación.
- Verificar solo cobertura numérica en lugar de comportamiento útil.
- Ignorar una prueba fallida o deshabilitarla sin documentar la causa.


Comandos habituales desde el directorio `Codigo/` del proyecto correspondiente:

```powershell
dotnet build <solucion>.sln
dotnet test <solucion>.sln
dotnet test <solucion>.sln --collect:"XPlat Code Coverage"
```

Los nombres exactos de solución, proyectos y opciones de restore están en el
README de cada servicio. El restore puede requerir feeds privados y red BCCR.

## Rendimiento y confiabilidad

- Medir consultas y operaciones antes de optimizar. Respetar límites de
  request, timeouts y contratos configurados.
- Evitar consultas repetidas y cargas de datos innecesarias; coordinar
  cualquier optimización de stored procedure con DBA.
- Usar operaciones asíncronas de EF/HTTP para I/O. No introducir `Task.Run`
  para aparentar asincronía.
- Usar timeouts existentes para llamadas HTTP y dependencias externas.
  Reintentos solo para fallos transitorios identificados, con límite,
  idempotencia y política compatible con los paquetes del proyecto.
- Preservar correlation ID y trazabilidad al cruzar API→MAPI.
- No cachear información de autorización, identidad o datos que deban reflejar
  cambios inmediatos sin una decisión explícita de expiración e invalidación.

> Preferir consultas que traigan solo las columnas y filas necesarias,
evitando materializar colecciones completas cuando el contrato permita filtrar,
paginar o proyectar en base de datos.

> Revisar consultas repetitivas entre capas y llamadas HTTP redundantes.
No introducir paralelismo automáticamente: usarlo solo para operaciones
independientes, con límites de concurrencia y sin compartir un `DbContext`
entre operaciones simultáneas.

> Los reintentos deben aplicarse solo a errores transitorios conocidos.
Toda política de retry debe tener límite, timeout, trazabilidad y considerar
idempotencia para evitar duplicar operaciones.


### Ejemplos de rendimiento y confiabilidad

> Se conservan las
> recomendaciones de evitar cargas innecesarias, medir antes de optimizar,
> paralelizar solo operaciones independientes y propagar errores sin
> sobreenvolverlos.

#### Evitar materializaciones innecesarias

**Evitar**

```csharp
var todos = await _contexto.Fondos
    .ToListAsync(cancellationToken);

var fondo = todos.FirstOrDefault(x => x.Codigo == codigoFondo);
```

**Recomendado**

```csharp
var fondo = await _contexto.Fondos
    .AsNoTracking()
    .SingleOrDefaultAsync(
        x => x.Codigo == codigoFondo,
        cancellationToken);
```

#### Paginación

Para endpoints que puedan retornar volúmenes grandes, usar los mecanismos de
paginación ya definidos por el contrato antes de cargar toda la colección en
memoria. No cambiar unilateralmente la firma de un endpoint existente para
introducir paginación.

#### Reintentos

No envolver todas las llamadas en reintentos genéricos.

**Evitar**

```csharp
for (var intento = 0; intento < 10; intento++)
{
    try
    {
        return await _servicio.EnvieAsync();
    }
    catch
    {
        // Reintento indiscriminado.
    }
}
```

Una política de retry debe limitarse a fallos transitorios identificados,
respetar timeout e idempotencia y usar la infraestructura del proyecto cuando
exista.


## Flujo de trabajo

- Leer primero el `README.md`, `AGENT.md`, SPEC y guías de la capa afectada.
- Hacer el cambio mínimo que preserve el contrato y las convenciones existentes.
- Si la funcionalidad atraviesa API y MAPI, verificar ambos lados del contrato;
  si afecta SQL o el esquema, coordinar con DBA.
- Ejecutar build y pruebas del proyecto que cambió; para cambios de interfaz
  entre servicios, incluir pruebas que cubran la integración pertinente.
- Revisar diagnósticos del build, pruebas y Quality Gate. No introducir
  correcciones no relacionadas ni actualizar paquetes como efecto colateral.
- Seguir las ramas, revisiones y procedimientos de despliegue descritos por
  cada repositorio. Esta guía no prescribe AWS Lambda, Serverless Framework,
  npm scripts ni migraciones Prisma para los backends .NET.

> Trabajar cambios funcionales en ramas pequeñas y enfocadas cuando el
flujo Git del repositorio lo permita. Usar commits descriptivos y someter los
cambios a revisión antes de fusionarlos.

> Antes de commit o Pull Request, ejecutar como mínimo el build y las
pruebas pertinentes del proyecto modificado, y revisar advertencias nuevas,
análisis estático y Quality Gate. No dar por válido un cambio únicamente porque
compile.

> Actualizar o crear pruebas cuando cambie comportamiento existente.
TDD puede utilizarse cuando aporte valor, pero no es una exigencia universal
para VES.

> No incluir cambios de formato, actualizaciones de paquetes,
refactorizaciones amplias o correcciones ajenas al objetivo de la rama salvo
que sean necesarias para completar de forma segura el cambio solicitado.

### Recomendaciones del documento anterior que NO se trasladan literalmente

> Las siguientes no deben convertirse en reglas de VES.

- No exigir DDD, agregados, Value Objects, factories o Domain Events por defecto.
  VES debe conservar **API Controller → SI.SG → IRequestsHandler → MAPI** y
  **MAPI Controller → BW → DA → SQL Server**.
- No imponer nombres, mensajes o commits en inglés cuando el repositorio VES y
  sus contratos institucionales usan español.

## Checklist rápido para código nuevo

> Lista de control para revisión humana o asistida por agentes.

Antes de considerar terminado un cambio backend:

- [ ] El cambio respeta API→SG→MAPI o MAPI→BW→DA según corresponda.
- [ ] No se introdujeron capas/patrones DDD que VES no necesita.
- [ ] Nombres de clases, interfaces, métodos, variables y campos siguen C# y el
      estilo del módulo.
- [ ] No hay `dynamic`, `object`, `null!` o casts innecesarios para evitar
      tipado.
- [ ] No hay números/cadenas mágicas cuando representan una regla estable.
- [ ] Dependencias se reciben por DI y no se crean con `new` dentro de BW/SG
      cuando existe un contrato.
- [ ] Controller no contiene SQL ni reglas de negocio que pertenezcan a BW.
- [ ] BW no contiene acceso SQL directo.
- [ ] DA usa SQL parametrizado y respeta objetos administrados por DBA.
- [ ] No se agregaron migraciones EF.
- [ ] Entradas se validan en el servidor.
- [ ] No se exponen secretos o datos sensibles en logs/respuestas.
- [ ] Logging es estructurado y no duplica el mismo error en todas las capas.
- [ ] No hay `.Result`, `.Wait()` ni `Task.Run` para I/O.
- [ ] `CancellationToken` se propaga donde el contrato lo soporte.
- [ ] No existe concurrencia sobre el mismo `DbContext`.
- [ ] Consultas traen solo los datos necesarios cuando el contrato lo permite.
- [ ] Se revisó riesgo de N+1.
- [ ] Las pruebas cubren comportamiento útil: éxito, errores y límites
      relevantes.
- [ ] Los mocks están en la frontera correcta de la unidad bajo prueba.
- [ ] No se depende de red BCCR, certificados o SQL de producción en unit tests.
- [ ] Build y tests pertinentes pasan.
- [ ] Se revisaron warnings y Quality Gate.
- [ ] No se incluyeron cambios colaterales no relacionados.

