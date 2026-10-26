---
description: Estándares de frontend para la SPA Bccr.Supen.Ves.UI dentro de la arquitectura VES
globs: ["VES/Bccr.Supen.Ves.UI/Codigo/src/**/*.{js,jsx}", "VES/Bccr.Supen.Ves.UI/Codigo/package.json"]
alwaysApply: false
---

# Estándares de frontend VES

## Alcance y arquitectura real

Este documento aplica a la interfaz web `Bccr.Supen.Ves.UI`, no a las APIs ni
a las capas de datos. El flujo de la solución es:

```text
Navegador (UI React) → Bccr.Supen.Ves.Api → Bccr.Supen.Ves.Mapi → BW → DA → SQL Server
```

- La UI consume por HTTP/HTTPS únicamente `Bccr.Supen.Ves.Api`; no llama
  directamente a MAPI ni accede a SQL Server.
- La API de VES orquesta las solicitudes mediante su `SI.SG` y se comunica con
  MAPI. Las capas `MAPI → BW → DA` pertenecen solo al proyecto MAPI.
- La UI es una SPA escrita principalmente en JavaScript y JSX. No usa
  TypeScript, React Bootstrap, Cypress, Jest ni una capa de internacionalización
  i18n. No se deben imponer esas tecnologías ni convertir código para ajustarse
  a ellas.
- El mapa de los tres componentes está en
  [base-standards.md](./base-standards.md). La estructura y los comandos
  concretos de la UI se mantienen en
  [Bccr.Supen.Ves.UI/README.md](..\VES\Bccr.Supen.Ves.UI/README.md),
  [AGENT.md](..\VES\Bccr.Supen.Ves.UI/AGENT.md),
  [Deploy guide](..\VES\Bccr.Supen.Ves.UI\Documentacion\Deploy.md),
  [SPEC.md](..\VES\Bccr.Supen.Ves.UI/SPEC.md) y
  [Documentacion/DEVELOPMENT.md](..\VES\Bccr.Supen.Ves.UI\Documentacion\DEVELOPMENT.md).

Y para los diseños visuales seguir los lineamientos establecidos en 
  [Usability guide](..\VES\Bccr.Supen.Ves.UI\Documentacion\USABILIDAD.md).
  [Data load design system](..\VES\Bccr.Supen.Ves.UI\Prototipo-UI\data-load-design-system.html),
  [Design handoff guide](..\VES\Bccr.Supen.Ves.UI\Prototipo-UI\DESIGN-HANDOFF.md),
  [Design system guide](..\VES\Bccr.Supen.Ves.UI\Prototipo-UI\DESIGN-SYSTEM.md),
  [Design manifests guide](..\VES\Bccr.Supen.Ves.UI\Prototipo-UI\DESIGN-MANIFEST.json).

Las buenas prácticas de la versión anterior que siguen siendo útiles se
conservan aquí: componentes funcionales y hooks, estado local apropiado,
estados de carga y error, validación de formularios, accesibilidad, pruebas
centradas en el comportamiento, optimización medida y cambios graduales. Los
ejemplos y herramientas se ajustan a la aplicación VES actual.

## Tecnologías y estructura

Las versiones exactas se consultan en `Codigo/package.json` y `Codigo/.nvmrc`.
El stack actual incluye:

- React 19 y React Router DOM 6.
- JavaScript/JSX, Vite y Node.js 24.x.
- Material UI v7 (incluidos MUI X Data Grid y Date Pickers) y Bootstrap 5.
- Axios, Vitest, jsdom y React Testing Library.
- Day.js y ExcelJS.

Estructura principal de `Codigo/src/`:

| Ruta | Responsabilidad |
|---|---|
| `STI/` | Pantallas del módulo Sistema de Transferencia de Información |
| `SEC/` | Pantallas del módulo Servicio de Entidades de Custodia |
| `componentes/` | Componentes reutilizables de la aplicación |
| `estilos/` | Tema VES, soporte claro/oscuro, tokens e iconos |
| `infraestructura/` | Cliente HTTP, sesión, autenticación y utilidades transversales |
| `rutas/` | Protección y decisiones de navegación |
| `Constantes.js` | URLs y constantes de la API |
| `ConstantesGaudi.js` | Configuración de GAUDI/OpenID |
| `App.jsx`, `Dashboard.jsx` | Composición y navegación principal |

Las pantallas React se ubican por módulo o en `componentes/` si son
reutilizables. Se usa `.jsx` para archivos con JSX y `.js` para módulos de
lógica que no contienen JSX, de acuerdo con la configuración actual de Vite.
No reorganizar carpetas ni cambiar extensiones masivamente como parte de una
funcionalidad no relacionada.


### Matriz de transformación: documento 1 → VES

> Las reglas siguientes muestran la equivalencia que debe aplicar un desarrollador
> o agente al trabajar en VES.

| Tema del documento 1 | Documento 1 | Equivalencia VES |
|---|---|---|
| React | React 18 | React 19 |
| Lenguaje | TypeScript preferente | JavaScript/JSX |
| Build | Create React App | Vite |
| UI | React Bootstrap/Bootstrap | Material UI v7 para código visual nuevo; conservar Bootstrap 5 donde ya exista |
| HTTP | Axios en servicios | `infraestructura/httpCliente.js` para código nuevo o zonas migradas; Axios existente no se migra masivamente |
| Routing | React Router | React Router DOM 6 según configuración existente |
| Unit/component tests | Jest + RTL | Vitest + React Testing Library |
| E2E | Cypress | No introducir Cypress; no forma parte del stack actual |
| i18n | i18next obligatorio | No introducir i18n; conservar español y textos del módulo |
| Estado | `useState`, `useEffect`, custom hooks | Se conserva, con los patrones actuales de VES |
| Componentes | Funcionales + hooks | Se conserva |
| Accesibilidad | HTML semántico, teclado, ARIA | Se conserva y se alinea con WCAG 2.1 AA documentado para VES |
| Variables/funciones | `camelCase` | `camelCase` |
| Componentes/tipos | `PascalCase` | Componentes `PascalCase`; no introducir tipos/interfaces TypeScript |
| Constantes | `UPPER_SNAKE_CASE` | Respetar convención real del módulo; algunos exports usan `LAS_RUTAS_EXTERNAS`, otros contratos existentes usan `camelCase` |
| Archivos | Componentes `PascalCase`, utilidades `camelCase` | Se conserva para archivos nuevos, usando `.jsx` con JSX y `.js` sin JSX |
| CSS | `kebab-case` | Mantener `kebab-case` en CSS propio; preferir tema/tokens/sx de MUI cuando aplique |
| Hooks | `use...` | Se conserva (`useTemaVes`, `use...`) |
| Idioma | Inglés | Español coherente con VES; términos de framework pueden mantenerse en inglés |

La columna **Equivalencia VES** tiene prioridad al generar o revisar código.


## Convenciones de código

- Mantener JavaScript/JSX y los patrones existentes. No introducir TypeScript
  como requisito para componentes nuevos.
- Seguir las convenciones españolas existentes para identificadores y
  comentarios: componentes en PascalCase; nombres de variables y funciones en
  camelCase, conservando las convenciones de cada módulo. Los exports
  constantes existentes pueden usar nombres como `LAS_RUTAS_EXTERNAS`.
- Usar componentes funcionales y hooks para código nuevo. Desestructurar props
  cuando mejore la claridad y proporcionar valores por defecto solo cuando el
  comportamiento requerido esté definido.
- Mantener los nombres públicos del API, las claves de configuración y los
  contratos existentes; no renombrarlos solo por estilo.
- Preferir componentes pequeños con responsabilidad clara. Extraer un hook o
  utilidad compartida cuando exista lógica de estado reutilizable, sin crear
  abstracciones anticipadamente.
- No agregar dependencias ni reemplazar librerías existentes sin necesidad
  justificada y aprobación del equipo.

Ejemplo de componente con las convenciones actuales:

```jsx
import Button from '@mui/material/Button';

export default function BotonConsultar({ estaCargando, alConsultar }) {
    return (
        <Button
            type="button"
            disabled={estaCargando}
            onClick={alConsultar}
        >
            {estaCargando ? 'Consultando...' : 'Consultar'}
        </Button>
    );
}
```


### Convenciones detalladas de escritura y nombres

> Se conservan las convenciones explícitas para componentes, variables, funciones, constantes,
> archivos, clases CSS y hooks. Se adaptan al JavaScript/JSX e idioma real de VES.

#### Componentes React

Usar `PascalCase` para componentes y nombres que describan claramente su
responsabilidad.

**Evitar**

```jsx
export default function boton() {
    // ...
}
```

```jsx
export default function Componente1() {
    // ...
}
```

**Recomendado**

```jsx
export default function BotonConsultar() {
    // ...
}
```

```jsx
export default function ResumenDeOperacion() {
    // ...
}
```

Un componente debe representar una responsabilidad visual o funcional clara.
No extraer componentes diminutos únicamente para reducir líneas; extraer cuando
mejore reutilización, legibilidad, aislamiento o prueba.

#### Variables y funciones

Usar `camelCase`.

```jsx
const codigoEntidad = 10;
const tiposDeArchivo = [];
const obtenerTiposDeArchivo = async () => {};
```

Evitar abreviaturas ambiguas o nombres sin intención:

```jsx
const x = 10;
const arr = [];
const proc = async () => {};
```

#### Handlers de eventos

Seguir la convención del módulo. En VES son válidos nombres semánticos en
español como:

```jsx
const alConsultar = () => {};
const alGuardar = () => {};
const alCambiarArchivo = (event) => {};
```

Si una zona existente usa `handle...`, conservarla en esa zona antes de
renombrar solo por estilo.

Evitar:

```jsx
const click = () => {};
const fn1 = () => {};
```

#### Booleanos

Usar nombres que expresen estado o condición:

```jsx
const [estaCargando, setEstaCargando] = useState(false);
const [estaGuardando, setEstaGuardando] = useState(false);
const puedeEditar = permisos.includes('editar');
const tieneErrores = errores.length > 0;
```

Evitar:

```jsx
const [flag, setFlag] = useState(false);
const ok = true;
```

#### Constantes

> VES contiene más de una convención pública en `Constantes.js`, por lo que 
> no debe renombrarse código existente mecánicamente.

Para constantes privadas/locales estables puede usarse una convención clara y
consistente con el archivo:

```jsx
const CANTIDAD_MAXIMA_RESULTADOS = 100;
const TIEMPO_ESPERA_MS = 30_000;
```

Para exports públicos ya establecidos, conservar su forma actual:

```jsx
export const LAS_RUTAS_EXTERNAS = { /* ... */ };
export const apiListarTiposArchivo = '...';
```

No duplicar URLs o valores de ambiente como constantes dentro de componentes.

#### Hooks

Todo custom hook debe comenzar con `use` y usar `camelCase`:

```jsx
function useTiposDeArchivo() {
    // ...
}
```

```jsx
const { esOscuro } = useTemaVes();
```

No llamar `use...` a una función que no use hooks ni usar hooks fuera de las
reglas de React.

#### Archivos

Para archivos nuevos:

```text
BotonConsultar.jsx
ResumenDeOperacion.jsx
useTiposDeArchivo.js
formatoFecha.js
httpCliente.js
```

Reglas:

- `PascalCase` para archivos cuyo export principal es un componente React.
- `camelCase` para hooks, utilidades, infraestructura y módulos sin componente.
- `.jsx` si contiene JSX.
- `.js` si no contiene JSX.
- No renombrar archivos existentes como efecto colateral de otra tarea.

#### CSS propio

Cuando se use CSS tradicional, preferir `kebab-case`:

```css
.resumen-operacion {}
.boton-consultar {}
```

No crear nuevas clases CSS si el mismo objetivo se resuelve de forma coherente
con el tema MUI, `sx`, tokens VES o una clase ya existente.

#### Props

Desestructurar props cuando mejore la claridad:

```jsx
export default function BotonConsultar({
    estaCargando,
    alConsultar,
}) {
    // ...
}
```

Evitar pasar props genéricas sin significado:

```jsx
export default function BotonConsultar(props) {
    return <button onClick={props.x}>{props.y}</button>;
}
```

Usar valores por defecto solo cuando exista un comportamiento definido:

```jsx
export default function TablaResultados({
    filas = [],
    estaCargando = false,
}) {
    // ...
}
```

No usar defaults para esconder datos requeridos ausentes.

#### Objetos y desestructuración

Desestructurar cuando haga evidente qué datos se usan:

```jsx
const { name, value } = event.target;
```

Evitar desestructuración excesiva cuando dificulte rastrear el origen del dato.

#### Idioma de identificadores, mensajes y logs

> VES usa español en UI, negocio, identificadores y documentación; 
> se conserva la intención de **consistencia lingüística**, no la regla literal.

- Mantener español en conceptos del negocio y UI.
- Conservar términos propios del framework (`props`, `hook`, `render`, `DataGrid`)
  cuando sean naturales en el código.
- Evitar mezclar español e inglés en el mismo concepto sin necesidad.
- No traducir nombres públicos del API ni claves de configuración.
- Los mensajes visibles deben usar el vocabulario de la pantalla/módulo.
- Los logs de desarrollo deben ser breves, diagnósticos y sin datos sensibles.

**Evitar**

```jsx
const candidateList = await obtenerCandidates();
console.error('Falló loading candidatos');
```

**Recomendado**

```jsx
const listaDeCandidatos = await obtenerCandidatos();
console.error('No fue posible obtener los candidatos', error);
```

#### Imports

- Agrupar imports de forma consistente con archivos vecinos.
- Preferir imports desde las rutas reales del proyecto.
- No introducir alias nuevos solo para acortar rutas sin ajustar la
  configuración del proyecto.
- Eliminar imports sin uso.
- No importar toda una biblioteca cuando basta un módulo/componente.

Ejemplo:

```jsx
import Button from '@mui/material/Button';
import Alert from '@mui/material/Alert';
```

#### Comentarios

Los comentarios deben explicar decisiones, restricciones o comportamiento no
evidente.

**Evitar**

```jsx
// Cambia loading a true
setEstaCargando(true);
```

**Recomendado**

```jsx
// Evita un segundo envío mientras el API procesa la operación actual.
setEstaCargando(true);
```

No conservar grandes bloques de código comentado; Git ya mantiene historial.

### Checklist de nomenclatura y escritura

- [ ] Componentes en `PascalCase`.
- [ ] Variables y funciones en `camelCase`.
- [ ] Hooks comienzan con `use`.
- [ ] Booleanos expresan condición/estado.
- [ ] Archivos de componente en `PascalCase.jsx`.
- [ ] Utilidades/hooks en `camelCase.js`.
- [ ] CSS propio en `kebab-case`.
- [ ] Constantes siguen la convención del módulo; no se renombraron exports
      públicos solo por estilo.
- [ ] No hay nombres genéricos como `data1`, `tmp`, `fn`, `flag`, `x`.
- [ ] No hay URLs de ambiente duplicadas en componentes.
- [ ] No hay comentarios que repitan literalmente el código.
- [ ] El idioma es coherente con el módulo y los contratos existentes.


## Componentes, tema y estilos


### Patrones de componentes y composición

> Se conservan los > principios de componentes funcionales, props explícitas, 
> hooks y extracción de lógica reutilizable, pero sin introducir TypeScript 
> ni una arquitectura nueva.

#### Componentes funcionales

Para código nuevo usar funciones y hooks.

**Evitar introducir componentes de clase nuevos**

```jsx
class ConsultarFondos extends React.Component {
    // ...
}
```

**Recomendado**

```jsx
export default function ConsultarFondos() {
    const [estaCargando, setEstaCargando] = useState(false);

    // ...
}
```

No convertir componentes legacy de clase solo porque se toque otra parte de la
pantalla.

#### Composición sobre componentes monolíticos

Si una pantalla mezcla muchas responsabilidades visuales y la extracción aporta
valor, separar por responsabilidades.

```text
PantallaConsulta
├── FiltrosConsulta
├── ResumenConsulta
└── TablaResultados
```

La extracción debe responder a una necesidad real de legibilidad, reutilización
o prueba; no imponer una estructura idéntica a todas las pantallas.

#### Custom Hook para lógica reutilizable

Cuando varias pantallas comparten una misma lógica de estado/efectos:

```jsx
function useTiposDeArchivo() {
    const [tipos, setTipos] = useState([]);
    const [estaCargando, setEstaCargando] = useState(false);
    const [mensajeDeError, setMensajeDeError] = useState('');

    // ...
    return { tipos, estaCargando, mensajeDeError };
}
```

No extraer un hook si solo encapsula una línea o si oculta el flujo sin
reutilización ni mejora de prueba.

#### Presentación y acceso HTTP

El documento 1 propone una capa de servicios centralizada. En VES no es
obligatorio crear un directorio de servicios: para nuevas llamadas usar
`infraestructura/httpCliente.js` y conservar el ownership del estado cerca de
la pantalla, salvo que ya exista un patrón compartido.

**Evitar**

```jsx
axios.get('https://servidor-produccion/api/tipos');
```

**Recomendado**

```jsx
await httpCliente.get(apiListarTiposArchivo);
```


- Para componentes visuales nuevos, preferir Material UI v7, su tema existente
  y el design system VES en `estilos/temaVes.js`.
- Usar `useTemaVes()` y `veSTokens` cuando el componente necesite conocer el
  tema o acceder a tokens directamente. Si un componente puede heredar el tema
  MUI sin lógica condicional propia, preferir estilos del tema frente a
  duplicar colores claro/oscuro.
- La aplicación también conserva Bootstrap 5 en pantallas existentes. No
  migrar esas pantallas como parte de una tarea ajena. Evitar mezclar estilos
  competidores dentro de un mismo componente; combinar bibliotecas solo cuando
  la integración sea necesaria y siga los estilos ya adoptados en esa zona.
- No exigir que todo elemento visual sea un componente MUI: usar HTML semántico
  (`main`, `nav`, `form`, `label`, botones y encabezados) donde corresponda.
  Evitar elementos genéricos sin semántica cuando hay una alternativa adecuada.
- Mantener el soporte claro/oscuro, los estados de foco visibles y el
  comportamiento responsive de la pantalla al introducir o modificar estilos.
- Respetar `prefers-reduced-motion` en animaciones nuevas y evitar depender
  solamente del color para comunicar errores, selección o estado.

Ejemplo con los tokens del tema VES:

```jsx
import Box from '@mui/material/Box';
import Typography from '@mui/material/Typography';
import { useTemaVes } from '../estilos/ContextoDeTema';
import { veSTokens } from '../estilos/temaVes';

export default function ResumenDeOperacion({ titulo }) {
    const { esOscuro } = useTemaVes();

    return (
        <Box
            sx={{
                p: 2,
                color: esOscuro ? veSTokens.fgDark : veSTokens.neutral900,
                backgroundColor: esOscuro ? veSTokens.surfaceDark : veSTokens.neutral50,
            }}
        >
            <Typography variant="h6">{titulo}</Typography>
        </Box>
    );
}
```


### Patrones de UI/UX adaptados a VES

#### MUI para componentes visuales nuevos

**Recomendado**

```jsx
import Button from '@mui/material/Button';
import Stack from '@mui/material/Stack';

<Stack direction={{ xs: 'column', md: 'row' }} spacing={2}>
    <Button variant="contained">Consultar</Button>
    <Button variant="outlined">Limpiar</Button>
</Stack>
```

En una vista Bootstrap existente, mantener Bootstrap si migrarla no forma parte
del alcance.

#### Estados visuales

Todo control interactivo debe representar, según aplique:

- normal;
- hover;
- focus visible;
- disabled;
- loading;
- error;
- selected/active.

No comunicar un estado únicamente mediante color.

#### Responsive

- Diseñar primero según el comportamiento de la pantalla existente.
- Verificar al menos tamaños de móvil, tablet y escritorio relevantes.
- Evitar anchos fijos innecesarios.
- Preferir breakpoints/tokens del sistema visual existente.
- Tablas/DataGrid deben mantener un comportamiento utilizable en viewport
  reducido.

#### Navegación

Usar React Router y rutas existentes.

No usar:

```jsx
window.location.href = '/ruta-interna';
```

cuando la navegación pertenece a la SPA y puede resolverse con React Router.

Conservar navegación externa segura con las utilidades existentes.

#### Botonera común de acciones

Las pantallas del VES **no crean botones de acción propios** (Abrir, Nuevo,
Eliminar, Descargar…): usan la `Botonera` de `componentes/botones`, que consulta
las opciones del usuario en la base de datos de seguridad
(`usuario/opciones/ObtenerOpcionesUsuarioAsync`) y asocia cada `nombreFisico` con
la función de la pantalla (`laListaDeLasFuncionesDeLosBotones`). La pantalla
recibe `elUsuario`, `elCodigoDeModulo`, `elCodigoOpcionPadre` y `elTitulo` del
`Dashboard`.

- Se muestran todas las opciones que el usuario tiene en la pantalla. Las que la
  pantalla aún no implementa se pasan con `funcionOpcionNoImplementada` para
  avisar «Opción no disponible aún»; sin esa propiedad se omiten.
- Las acciones sobre un registro actúan sobre la fila seleccionada y avisan si no
  hay selección.
- No sustituir esta botonera por botones por fila ni por controles propios.

#### Previsualización de archivos

- Solo PDF, con límite de tamaño y `Content-Type` validado; el blob se fuerza a
  `application/pdf` y se libera su URL al desmontar.
- No usar `sandbox` en el `iframe` del visor de PDF (Chrome lo bloquea con
  `ERR_BLOCKED_BY_CLIENT`); usar `referrerPolicy="no-referrer"`.
- Verificar el render visual en un navegador real (Edge/Chrome): el navegador
  automatizado puede colgarse con el visor de PDF.


## Estado, efectos y formularios

- Usar `useState` para el estado local y elevarlo únicamente cuando deba ser
  compartido. `ContextoDeTema` y `sesionLocal` cubren preocupaciones
  transversales existentes; no introducir una biblioteca global de estado para
  un caso local.
- Usar `useEffect` para sincronización con sistemas externos, no como
  sustituto general de eventos o cálculo derivado. Declarar dependencias
  correctas y limpiar suscripciones, temporizadores y otros recursos.
- Algunas pantallas heredadas omiten dependencias con
  `eslint-disable-line react-hooks/exhaustive-deps`. No copiar ese patrón sin
  una razón comprobada ni cambiar esas dependencias mecánicamente: analizar
  primero el flujo de datos y validar el comportamiento.
- Usar formularios controlados o `FormData` según el patrón del componente.
  Validar entradas con mensajes comprensibles; conservar valores si ocurre un
  error y evitar envíos duplicados deshabilitando la acción mientras se guarda.
- Toda operación asíncrona visible debe representar sus estados de carga,
  éxito y error que apliquen. No convertir errores en respuestas exitosas ni
  ocultar fallos.


### Ejemplos de estado, efectos y formularios

> Se mantienen los principios de estado local, estados de carga/error, formularios controlados y
> custom hooks. Se añaden reglas de React modernas para evitar efectos
> innecesarios y actualizaciones obsoletas.

#### Estado local

Usar `useState` para estado que pertenece al componente:

```jsx
const [estaCargando, setEstaCargando] = useState(false);
const [mensajeDeError, setMensajeDeError] = useState('');
```

No elevar el estado a `App` o a un contexto si únicamente lo consume una
pantalla.

#### Estado derivado

**Evitar guardar estado que puede calcularse**

```jsx
const [cantidad, setCantidad] = useState(0);

useEffect(() => {
    setCantidad(filas.length);
}, [filas]);
```

**Recomendado**

```jsx
const cantidad = filas.length;
```

#### `useEffect`

Usar `useEffect` para sincronizar con algo externo: red, DOM externo,
suscripciones, timers o APIs del navegador.

**Evitar**

```jsx
useEffect(() => {
    setNombreCompleto(`${nombre} ${apellido}`);
}, [nombre, apellido]);
```

**Recomendado**

```jsx
const nombreCompleto = `${nombre} ${apellido}`;
```

#### Limpieza de efectos

```jsx
useEffect(() => {
    const controlador = new AbortController();

    cargarDatos({ signal: controlador.signal });

    return () => {
        controlador.abort();
    };
}, []);
```

Usar el mecanismo de cancelación compatible con `httpCliente`/Axios y el código
real de la zona; el ejemplo representa el patrón.

#### Formularios controlados

```jsx
const [formulario, setFormulario] = useState({
    codigoEntidad: '',
    fechaInicio: '',
});

const alCambiar = (event) => {
    const { name, value } = event.target;

    setFormulario((anterior) => ({
        ...anterior,
        [name]: value,
    }));
};
```

#### Evitar doble envío

```jsx
<Button
    type="submit"
    disabled={estaGuardando}
>
    {estaGuardando ? 'Guardando...' : 'Guardar'}
</Button>
```

#### Error de guardado

Ante un error:

- conservar los datos que el usuario introdujo;
- mostrar un mensaje comprensible;
- restaurar el estado de interacción;
- no mostrar éxito;
- conservar detalles técnicos solo para diagnóstico, no en UI.


## Comunicación con la API

- Usar URLs y rutas definidas en `Constantes.js`; no duplicar URLs de ambiente
  en componentes ni fijar una URL local o de producción.
- Todas las llamadas de negocio de la UI deben dirigirse a
  `Bccr.Supen.Ves.Api`, respetando sus rutas versionadas y convenciones. Nunca
  conectar la UI directamente a MAPI, SQL Server o stored procedures.
- El proyecto migra gradualmente sus llamadas Axios hacia
  `infraestructura/httpCliente.js`. Usar ese cliente para nuevas llamadas o
  al modificar una zona ya migrada; no reemplazar de una sola vez las llamadas
  existentes ni cambiar sus contratos sin una tarea específica.
- `httpCliente` configura timeout e interceptor de sesión; también exporta
  `ObtengaMensajeDeErrorHttp`. Reutilizar estas capacidades donde corresponda
  y conservar los datos útiles del error para diagnóstico.
- Mantener solicitudes, loading y error cerca de la pantalla que posee su
  estado, a menos que la funcionalidad ya tenga un patrón compartido claro.
  No es requisito crear un directorio o una capa de servicios nueva.

Ejemplo de solicitud de UI usando el cliente compartido y una URL de
configuración existente:

```jsx
import { useState } from 'react';
import { Alert, Button } from '@mui/material';
import httpCliente, {
    ObtengaMensajeDeErrorHttp,
} from '../infraestructura/httpCliente';
import { apiListarTiposArchivo } from '../Constantes';

export default function ConsultarTiposDeArchivo() {
    const [estaCargando, setEstaCargando] = useState(false);
    const [mensajeDeError, setMensajeDeError] = useState('');

    const alConsultar = async () => {
        setEstaCargando(true);
        setMensajeDeError('');

        try {
            await httpCliente.get(apiListarTiposArchivo);
        } catch (error) {
            setMensajeDeError(ObtengaMensajeDeErrorHttp(error));
        } finally {
            setEstaCargando(false);
        }
    };

    return (
        <>
            <Button disabled={estaCargando} onClick={alConsultar}>
                {estaCargando ? 'Consultando...' : 'Consultar'}
            </Button>
            {mensajeDeError && <Alert severity="error">{mensajeDeError}</Alert>}
        </>
    );
}
```


### Patrones de comunicación HTTP

> Se conserva la centralización de configuración HTTP y manejo consistente de errores, 
> pero se adapta al cliente compartido existente de VES.

#### No duplicar configuración

**Evitar**

```jsx
const respuesta = await axios.get(
    'http://localhost:5000/api/tipos-archivo'
);
```

**Recomendado**

```jsx
const respuesta = await httpCliente.get(apiListarTiposArchivo);
```

#### Error de red

```jsx
try {
    const respuesta = await httpCliente.get(apiListarTiposArchivo);
    setTipos(respuesta.data);
} catch (error) {
    setMensajeDeError(ObtengaMensajeDeErrorHttp(error));
}
```

No mostrar `error.stack`, respuesta completa, tokens ni detalles internos al
usuario.

#### Solicitudes concurrentes

Cuando dos solicitudes sean independientes y el endpoint/UX lo justifique,
pueden ejecutarse en paralelo:

```jsx
const [catalogos, parametros] = await Promise.all([
    httpCliente.get(apiCatalogos),
    httpCliente.get(apiParametros),
]);
```

No usar `Promise.all` si una solicitud depende del resultado de la otra o si el
flujo requiere tratamiento independiente de errores.

#### Datos obsoletos

Si una búsqueda puede cambiar rápidamente de criterios, cancelar la solicitud
anterior o ignorar su resultado cuando ya no corresponda al criterio actual.

#### Caché

El documento 1 recomienda cachear respuestas cuando sea apropiado. En VES no se
debe introducir una biblioteca de caché global solo por esa recomendación.
Definir explícitamente:

- qué dato puede quedar obsoleto;
- cuánto tiempo es válido;
- cómo se invalida;
- si contiene información sensible o dependiente de permisos.


## Accesibilidad y usabilidad

- Usar etiquetas semánticas y asociar cada control de formulario con una
  etiqueta accesible.
- Proporcionar nombre accesible a botones de icono, texto alternativo útil a
  imágenes informativas y `alt=""` a imágenes decorativas.
- Mantener navegación por teclado, foco visible, orden de foco lógico y
  contraste suficiente. Seguir el objetivo de accesibilidad WCAG 2.1 AA
  documentado para VES.
- Anunciar errores y resultados dinámicos con semántica apropiada (por ejemplo
  `role="alert"` en mensajes urgentes). No comunicar estado solo por color.
- Conservar el idioma español de la aplicación y del negocio. No incorporar
  i18n ni traducir etiquetas como parte de una tarea no relacionada; mantener
  consistencia con los textos y términos del módulo.


### Ejemplos de accesibilidad

> Se conservan HTML semántico, teclado, texto alternativo y nombres accesibles. En VES se aplica
> además el objetivo WCAG 2.1 AA documentado por el proyecto.

#### Botón de icono

**Evitar**

```jsx
<IconButton onClick={alEliminar}>
    <DeleteIcon />
</IconButton>
```

**Recomendado**

```jsx
<IconButton
    aria-label="Eliminar archivo"
    onClick={alEliminar}
>
    <DeleteIcon />
</IconButton>
```

#### Clickable `div`

**Evitar**

```jsx
<div onClick={alConsultar}>Consultar</div>
```

**Recomendado**

```jsx
<Button onClick={alConsultar}>Consultar</Button>
```

Usar el elemento semántico correcto evita tener que recrear manualmente
comportamiento de teclado, foco y roles.

#### Formularios

Cada control debe tener un nombre accesible mediante `label`, propiedad MUI
equivalente o `aria-label` cuando no exista etiqueta visible adecuada.

#### Mensajes dinámicos

Para errores urgentes:

```jsx
<Alert severity="error" role="alert">
    {mensajeDeError}
</Alert>
```

No abusar de `role="alert"` para mensajes informativos no urgentes.

#### Imágenes

```jsx
<img src={grafico} alt="Distribución mensual de operaciones" />
<img src={decoracion} alt="" />
```

El `alt` debe describir propósito, no repetir “imagen de”.


## Pruebas y calidad

- El runner es Vitest con `globals: true`, configuración en
  `vite.config.mjs` y entorno `jsdom`. Las pruebas usan React Testing Library.
- Ubicar las pruebas junto a la funcionalidad con sufijos `.test.js` o
  `.test.jsx`, siguiendo el patrón de la carpeta (`STI`, `SEC`,
  `componentes`, `infraestructura`, `estilos`, `rutas`).
- Preferir consultas accesibles de Testing Library (`getByRole`,
  `getByLabelText`, `findBy...`) y comprobar el comportamiento observable.
  Usar selectores `data-testid` solo cuando no haya una consulta semántica
  adecuada.
- Cubrir comportamientos relevantes: casos exitosos, validación, errores,
  estados de carga y casos límite según aplique. No añadir pruebas de
  integración con Cypress o Jest: no forman parte del stack actual.
- `describe`, `test`, `expect` y `beforeEach` están disponibles globalmente.
  Los mocks y espías se importan de `vitest` (`vi.mock`, `vi.fn`,
  `vi.clearAllMocks`); no usar `jest.*`.
- Antes de finalizar cambios de código de UI, ejecutar desde
  `Bccr.Supen.Ves.UI/Codigo/`:

```bash
npm run lint
npm test
npm run build:local
```

No ejecutar `build:pruebas` ni `build:produccion` solo como validación local:
esos scripts incrementan la versión en `package.json`.


### Patrones de escritura de pruebas

> Se conserva la filosofía de probar flujos y comportamiento, nombres descriptivos, casos de
> éxito/error/límite y aislamiento. Jest/Cypress se transforman al stack real:
> Vitest + React Testing Library.

#### Nombre de pruebas

Preferir nombres orientados al comportamiento:

```jsx
test('muestra los tipos de archivo cuando la consulta es exitosa', async () => {
    // ...
});

test('muestra un mensaje de error cuando falla la consulta', async () => {
    // ...
});
```

Evitar:

```jsx
test('test1', () => {});
test('componente funciona', () => {});
```

#### Arrange / Act / Assert

El patrón AAA puede usarse para organizar pruebas no triviales:

```jsx
test('deshabilita Consultar mientras la solicitud está en curso', async () => {
    // Arrange
    // configurar mock y renderizar

    // Act
    // ejecutar acción de usuario

    // Assert
    // verificar comportamiento observable
});
```

No es obligatorio escribir los comentarios si el test ya es claro.

#### Consultas semánticas

**Preferir**

```jsx
screen.getByRole('button', { name: /consultar/i });
screen.getByLabelText(/código de entidad/i);
await screen.findByRole('alert');
```

**Evitar como primera opción**

```jsx
screen.getByTestId('btn-consultar');
```

`data-testid` es válido cuando no existe una consulta semántica estable.

#### Interacción del usuario

Usar las utilidades de interacción ya instaladas/configuradas en la suite. Si
`user-event` está disponible, preferir interacción cercana al comportamiento
real sobre disparar eventos de bajo nivel manualmente.

#### Mock de HTTP con Vitest

Usar `vi.mock`, `vi.fn`, `vi.spyOn` según el patrón existente.

```jsx
import { vi } from 'vitest';

vi.mock('../infraestructura/httpCliente', () => ({
    default: {
        get: vi.fn(),
    },
    ObtengaMensajeDeErrorHttp: vi.fn(),
}));
```

No usar:

```jsx
jest.mock(...)
jest.fn()
```

#### Qué probar

Según aplique:

1. render inicial;
2. loading;
3. resultado exitoso;
4. error;
5. validación;
6. estado vacío;
7. botones disabled/enabled;
8. navegación;
9. accesibilidad básica;
10. actualización por interacción del usuario;
11. caso límite relevante.

No todos los componentes requieren todos los casos.

#### Qué no probar

- detalles internos de hooks o estado si no son observables;
- nombres de funciones privadas;
- estructura exacta del árbol DOM si no forma parte del contrato;
- clases MUI generadas dinámicamente;
- implementación interna de React.

#### Ejemplo completo

```jsx
test('muestra error y conserva la acción disponible cuando falla la consulta', async () => {
    httpCliente.get.mockRejectedValueOnce(new Error('fallo'));
    ObtengaMensajeDeErrorHttp.mockReturnValueOnce(
        'No fue posible realizar la consulta.'
    );

    render(<ConsultarTiposDeArchivo />);

    await user.click(
        screen.getByRole('button', { name: /consultar/i })
    );

    expect(
        await screen.findByRole('alert')
    ).toHaveTextContent('No fue posible realizar la consulta.');

    expect(
        screen.getByRole('button', { name: /consultar/i })
    ).toBeEnabled();
});
```

Ajustar imports y helpers a la configuración real de la suite.

#### Anti-patrones de pruebas

- usar `jest.*`;
- depender de una API o red real en unit/component tests;
- usar timeouts/sleeps arbitrarios;
- probar detalles privados;
- usar `data-testid` cuando existe un rol/label accesible;
- compartir estado mutable entre tests;
- depender del orden de ejecución;
- ocultar un fallo dentro de `try/catch`;
- perseguir cobertura numérica sin validar comportamiento.

#### Cobertura

No imponer un porcentaje proveniente de otro proyecto si el pipeline VES no lo
define. Priorizar escenarios significativos y respetar los umbrales/Quality
Gate efectivos del repositorio.

## Configuración, seguridad y límites del navegador

- La configuración por ambiente vive en `Codigo/config/{ambiente}.json` y se
  fusiona con `config/default.json`. `NODE_CONFIG_ENV` selecciona la
  configuración durante Vite build/dev; `vite.config.mjs` la expone como
  `__APP_CONFIG__`. Consultar `..\VES\Bccr.Supen.Ves.UI\Documentacion\Deploy.md` antes de modificarla.
- No usar `Constantes.js` para fijar un ambiente ni cambiar su mecanismo de
  configuración. Los valores que llegan al bundle del navegador son públicos:
  nunca colocar secretos, claves privadas o credenciales en la configuración
  del cliente.
- No agregar secretos a `Codigo/.env`; el valor heredado `ENCRYPTION_KEY` está
  documentado como pendiente de gestión segura y no debe replicarse.
- Para enlaces externos, conservar validación y apertura segura
  (`noopener,noreferrer`) según el patrón de `EnlacesExternos.js` y
  `Dashboard.jsx`. No formar destinos confiando sin validación en entradas
  recibidas.
- Evitar persistir tokens o datos sensibles en `localStorage`. Respetar los
  flujos de autenticación GAUDI/OpenID y de sesión implementados en
  `Autenticacion.jsx`, `LandingGaudi.jsx` y `infraestructura/`.


### Ejemplos de configuración y seguridad del cliente

>  Se conserva la separación de configuración por ambiente y la prohibición de secretos en el
> cliente. La implementación se adapta al mecanismo `config/*.json` + Vite de
> VES, no a `.env` o Create React App.

#### Configuración

**Evitar**

```jsx
const API_URL = 'https://api-produccion/';
```

dentro de un componente.

**Recomendado**

Consumir las constantes/configuración ya expuestas por el mecanismo VES.

#### Los secretos no existen en frontend

Todo valor incluido en el bundle puede ser inspeccionado por el usuario.
Nunca colocar allí:

- contraseñas;
- client secrets;
- private keys;
- connection strings;
- tokens permanentes.

#### HTML dinámico

Evitar `dangerouslySetInnerHTML` salvo necesidad real y contenido confiable o
sanitizado mediante el mecanismo aprobado por el proyecto.

#### URLs externas

No construir destinos externos directamente desde entradas no confiables.
Reutilizar validación y apertura segura existente (`noopener,noreferrer`).

#### Persistencia del navegador

No persistir tokens, claims sensibles o información confidencial adicional en
`localStorage` por conveniencia. Seguir el flujo de sesión/autenticación ya
implementado.


## Rendimiento y mantenimiento

- Medir antes de optimizar. Evitar re-renderizados evitables, pero no agregar
  `memo`, `useMemo` o `useCallback` sin necesidad demostrada.
- Mantener el splitting y la configuración de chunks existente en
  `vite.config.mjs`. Considerar carga diferida para módulos/pantallas grandes
  cuando aporte una mejora medible y sea compatible con navegación y
  despliegue.
- Usar `loading="lazy"` para imágenes no críticas cuando sea apropiado;
  conservar el manejo específico de cargas y descargas de archivos grandes.
- Mantener claves estables para listas y cancelar o ignorar resultados
  asíncronos obsoletos cuando una pantalla pueda desmontarse o cambiar sus
  criterios de consulta.
- No refactorizar vistas STI/SEC, cambiar lógica de negocio, actualizar
  dependencias ni alterar contratos del API como efectos secundarios de una
  mejora visual.


### Patrones de rendimiento

> Se conservan lazy loading, code splitting, optimización de assets, reducción de renders y
> eficiencia HTTP, pero solo cuando exista beneficio medible.

#### `useMemo` y `useCallback`

**Evitar optimización preventiva**

```jsx
const titulo = useMemo(() => 'Consulta', []);
```

```jsx
const alCerrar = useCallback(() => {
    setAbierto(false);
}, []);
```

si no existe una razón medible o una dependencia que requiera identidad estable.

**Usar cuando corresponda**, por ejemplo, un cálculo costoso derivado de datos
grandes:

```jsx
const filasOrdenadas = useMemo(
    () => ordenarFilas(filas, criterio),
    [filas, criterio]
);
```

#### Claves de listas

**Evitar**

```jsx
filas.map((fila, indice) => (
    <Fila key={indice} fila={fila} />
));
```

cuando exista un identificador estable.

**Recomendado**

```jsx
filas.map((fila) => (
    <Fila key={fila.id} fila={fila} />
));
```

#### Lazy loading

Considerar `React.lazy`/splitting para pantallas grandes que no se necesiten en
el arranque, sin alterar la estrategia existente de chunks ni navegación.

#### Imágenes

Para imágenes no críticas:

```jsx
<img
    src={miniatura}
    alt="..."
    loading="lazy"
/>
```

No aplicar lazy loading a recursos necesarios para el contenido inicial si
perjudica UX.

#### API

- no repetir la misma solicitud innecesariamente;
- evitar pedir datos que la pantalla no necesita si el API permite filtrado;
- cancelar/ignorar respuestas obsoletas;
- paralelizar solo solicitudes independientes;
- no introducir caching global sin estrategia de expiración/invalidez.


## Flujo de trabajo

- Analizar primero las convenciones de la pantalla y sus componentes vecinos.
- Hacer cambios pequeños y relacionados; no asumir que una recomendación
  genérica prevalece sobre los requisitos del negocio o las convenciones
  vigentes de VES.
- Seguir el flujo de ramas y revisión descrito en `Bccr.Supen.Ves.UI/AGENT.md`.
- Mantener README, SPEC y guías de desarrollo alineados con el comportamiento
  real cuando cambien arquitectura, herramientas o procesos.

## Checklist rápido para código frontend nuevo

### Arquitectura y API

- [ ] La UI consume únicamente `Bccr.Supen.Ves.Api`.
- [ ] No hay llamadas directas a MAPI, SQL Server o stored procedures.
- [ ] URLs/rutas se obtienen de `Constantes.js` o configuración existente.
- [ ] Nuevas llamadas usan `httpCliente` cuando corresponde.
- [ ] No se creó una nueva capa/abstracción sin necesidad.

### Nombres y estructura

- [ ] Componentes en `PascalCase`.
- [ ] Variables y funciones en `camelCase`.
- [ ] Hooks comienzan con `use`.
- [ ] Booleanos tienen nombres descriptivos.
- [ ] Archivos `.jsx` contienen JSX y `.js` lógica sin JSX.
- [ ] CSS propio usa `kebab-case`.
- [ ] No se renombraron contratos públicos por estilo.
- [ ] Idioma coherente con el módulo.

### React

- [ ] Componente funcional para código nuevo.
- [ ] Props claras/desestructuradas cuando aporta legibilidad.
- [ ] Estado mantenido lo más local posible.
- [ ] No hay estado derivado innecesario.
- [ ] `useEffect` se usa para sincronización real, no para cálculos simples.
- [ ] Efectos limpian recursos cuando corresponde.
- [ ] Listas usan `key` estable.

### UI/UX

- [ ] Nuevos componentes visuales usan MUI/tema VES cuando corresponde.
- [ ] Se conserva claro/oscuro.
- [ ] Responsive verificado.
- [ ] Loading/error/empty/success representados cuando aplican.
- [ ] No hay estado comunicado solo por color.
- [ ] Acciones async evitan doble envío.

### Accesibilidad

- [ ] HTML semántico.
- [ ] Controles con nombre accesible.
- [ ] Navegación por teclado.
- [ ] Foco visible.
- [ ] Imágenes con `alt` correcto.
- [ ] Errores dinámicos anunciados apropiadamente.
- [ ] Contraste suficiente.

### Seguridad

- [ ] Sin secretos en código/config del bundle.
- [ ] Sin tokens sensibles nuevos en `localStorage`.
- [ ] Enlaces externos validados y abiertos de forma segura.
- [ ] No se usa `dangerouslySetInnerHTML` sin justificación/sanitización.
- [ ] Errores visibles no exponen detalles internos.

### Pruebas

- [ ] Vitest + React Testing Library.
- [ ] Consultas semánticas antes de `data-testid`.
- [ ] Pruebas centradas en comportamiento.
- [ ] Éxito/error/loading/validación cubiertos según riesgo.
- [ ] Sin Jest/Cypress nuevos.
- [ ] Sin llamadas reales a red en unit/component tests.

### Rendimiento

- [ ] No hay `useMemo`/`useCallback` preventivos.
- [ ] No hay solicitudes HTTP duplicadas innecesariamente.
- [ ] Respuestas obsoletas se cancelan/ignoran cuando aplica.
- [ ] Imágenes/assets se optimizan cuando aporta valor.
- [ ] No se cambió splitting/chunks sin medición.

### Antes de Pull Request

- [ ] `npm run lint`
- [ ] `npm test`
- [ ] `npm run build:local`
- [ ] No se ejecutaron builds que incrementan versión solo para validar local.
- [ ] No hay cambios colaterales de dependencias, contratos o módulos no
      relacionados.

